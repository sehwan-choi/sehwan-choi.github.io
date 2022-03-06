---
layout: post
title:  "Spring MultiPartFile을 이용한 File upload"
subtitle:   "Spring MultiPartFile을 이용한 File upload"
date:   2022-03-07 02:22:22 +0900
categories: spring
tags: spring Type MultiPartFile FileUpload
comments: true
---


<br>

- 목차
    - [스프링과 파일 업로드](#스프링과-파일-업로드)
        - [MultipartFile 주요 메서드](#multipartfile-주요-메서드)
    - [예제로 구현하는 파일 업로드, 다운로드](#예제로-구현하는-파일-업로드-다운로드)
        - [요구-사항](#요구-사항)
        - [예제 코드](#예제-코드)
        - [View 코드](#view-코드)
    - [결과 확인](#결과-확인)


# 스프링과 파일 업로드

<br>

스프링은 MultipartFile 이라는 인터페이스로 멀티파트 파일을 매우 편리하게 지원한다.

<br>

SpringUploadController
```java
@Slf4j
@Controller
@RequestMapping("/spring/")
public class SpringUploadController {

    @Value("${file.dir}")
    private String fileDir;

    @GetMapping("/upload")
    public String newFile() {
        return "upload-form";
    }

    @PostMapping("/upload")
    public String saveFile(@RequestParam String itemName, @RequestParam MultipartFile file, HttpServletRequest request) throws IOException {
        log.info("request={}", request);
        log.info("itemName={}", itemName);
        log.info("multipartFile={}", file);

        if (!file.isEmpty()) {
            String fullPath = fileDir + file.getOriginalFilename();
            log.info("파일 저장 fullPath={}", fullPath);

            //실제 파일저장
            file.transferTo(new File(fullPath));
        }

        return "upload-form";
    }

    @Data
    public static class MultiPartData{
        private String itemName;
        private List<MultipartFile> file;
    }
}
```

코드를 보면 스프링 답게 딱 필요한 부분의 코드만 작성하면 된다. <br>
```
@RequestParam MultipartFile file
```
업로드하는 HTML Form의 name에 맞추어 @RequestParam 을 적용하면 된다.  <br>
추가로 @ModelAttribute 에서도 MultipartFile 을 동일하게 사용할 수 있다.<br>

## MultipartFile 주요 메서드

<br>
- file.getOriginalFilename() : 업로드 파일 명
- file.transferTo(new File(저장 풀 경로)) : 파일 저장

<br><br>

# 예제로 구현하는 파일 업로드, 다운로드

<br>

실제 파일이나 이미지를 업로드, 다운로드 할 때는 몇가지 고려할 점이 있는데, 구체적인 예제로 알아보자. <br>


## 요구 사항

- 상품을 관리
    - 상품 이름
    - 첨부파일 하나
    - 이미지 파일 여러개
- 첨부파일을 업로드 다운로드 할 수 있다.
- 업로드한 이미지를 웹 브라우저에서 확인할 수 있다.

<br><br>

## 예제 코드

<br>

Item - 상품 도메인
```java
@Data
public class Item {

    private Long id;

    private String itemName;

    private UploadFile attachFile;

    private List<UploadFile> imageFiles;
}
```

<br>

ItemRepository - 상품 리포지토리
```java
@Repository
public class ItemRepository {

    private final Map<Long, Item> store = new HashMap<>();

    private long sequence = 0L;

    public Item save(Item item) {
        item.setId(++sequence);
        store.put(sequence, item);
        return item;
    }

    public Item findById(Long id) {
        return store.get(id);
    }
}
```

<br>

UploadFile - 업로드 파일 정보 보관
```java
@Data
public class UploadFile {

    private String uploadFileName;

    private String storeFileName;

    public UploadFile(String uploadFileName, String storeFileName) {
        this.uploadFileName = uploadFileName;
        this.storeFileName = storeFileName;
    }
}
```
- uploadFileName : 고객이 업로드한 파일명
- storeFileName : 서버 내부에서 관리하는 파일명

> 주의! <br>
> 고객이 업로드한 파일명으로 서버 내부에 파일을 저장하면 안된다. 왜냐하면 서로 다른 고객이 같은 파일이름을 업로드 하는 경우 기존 파일 이름과 충돌이 날 수 있다. 서버에서는 저장할 파일명이 겹치지 않도록 내부에서 관리하는 별도의 파일명이 필요하다.

<br><br>

FileStore - 파일 저장과 관련된 업무 처리
```java
@Component
public class FileStore {

    @Value("${file.dir}")
    private String fileDir;

    public String getFullPath(String fileName) {
        return fileDir + fileName;
    }

    public List<UploadFile> storeFiles(List<MultipartFile> multipartFiles) throws IOException {
        List<UploadFile> storeFileResult = new ArrayList<>();

        for (MultipartFile multipartFile : multipartFiles) {
            if (!multipartFile.isEmpty()) {
                storeFileResult.add(storeFile(multipartFile));
            }
        }
        return storeFileResult;
    }

    // 서버에 파일 저장
    public UploadFile storeFile(MultipartFile multipartFile) throws IOException {
        if (multipartFile.isEmpty()) {
            return null;
        }

        String originalFilename = multipartFile.getOriginalFilename();
        String storeFileName = createStoreFileName(originalFilename);

        multipartFile.transferTo(new File(getFullPath(storeFileName)));

        return new UploadFile(originalFilename,storeFileName);
    }

    // DB에 저장할 유니크한 이름 생성
    private String createStoreFileName(String originalFilename) {
        String ext = extractExt(originalFilename);
        String uuid = UUID.randomUUID().toString();
        return uuid + "." + ext;
    }

    // .png .jpg 등의 확장자 이름 추출
    private String extractExt(String originalFilename) {
        int pos = originalFilename.lastIndexOf(".");
        return originalFilename.substring(pos + 1);
    }

}
```

멀티파트 파일을 서버에 저장하는 역할을 담당한다. <br>
- createStoreFileName() : 서버 내부에서 관리하는 파일명은 유일한 이름을 생성하는 UUID 를 사용해서 충돌하지 않도록 한다.
- extractExt() : 확장자를 별도로 추출해서 서버 내부에서 관리하는 파일명에도 붙여준다. 예를 들어서 고객이 a.png 라는 이름으로 업로드 하면 51041c62-86e4-4274-801d-614a7d994edb.png 와 같이 저장한다.

<br>

ItemForm
```java
@Data
public class ItemForm {

    private Long itemId;

    private String itemName;

    private List<MultipartFile> imageFiles;

    private MultipartFile attachFile;
}
```

상품 저장용 폼이다. <br>
List<MultipartFile> imageFiles : 이미지를 다중 업로드 하기 위해  MultipartFile 를 사용했다. <br>
MultipartFile attachFile : 멀티파트는 __@ModelAttribute__ 에서 사용할 수 있다. <br>

<br><br>

ItemController
```java
@Slf4j
@Controller
@RequiredArgsConstructor
public class ItemController {

    private final ItemRepository itemRepository;

    private final FileStore fileStore;

    @GetMapping("/items/new")
    public String newItem(ItemForm form) {
        return "item-form";
    }

    @PostMapping("/items/new")
    public String saveItem(ItemForm form, RedirectAttributes redirectAttributes) throws IOException {
        UploadFile attachFile = fileStore.storeFile(form.getAttachFile());
        List<UploadFile> storeImageFiles = fileStore.storeFiles(form.getImageFiles());

        Item item = new Item();
        item.setItemName(form.getItemName());
        item.setAttachFile(attachFile);
        item.setImageFiles(storeImageFiles);
        itemRepository.save(item);

        redirectAttributes.addAttribute("itemId",item.getId());

        return "redirect:/items/{itemId}";
    }

    @GetMapping("/items/{id}")
    public String items(@PathVariable Long id, Model model) {
        Item item = itemRepository.findById(id);
        model.addAttribute("item",item);
        return "item-view";
    }

    @ResponseBody
    @GetMapping("/images/{filename}")
    public Resource downLoadImage(@PathVariable String filename) throws MalformedURLException {
        // 이미지를 보여주기 위함
        return new UrlResource("file:" + fileStore.getFullPath(filename));
    }

    @GetMapping("/attach/{itemId}")
    public ResponseEntity<Resource> downloadAttach(@PathVariable Long itemId) throws MalformedURLException {
        Item item = itemRepository.findById(itemId);
        String storeFileName = item.getAttachFile().getStoreFileName();
        String uploadFileName = item.getAttachFile().getUploadFileName();

        UrlResource urlResource = new UrlResource("file:" + fileStore.getFullPath(storeFileName));

        log.info("uploadFileName= {}", uploadFileName);

        // 파일명이 깨지지 않도록 인코딩한다.
        String encodedUploadFileName = UriUtils.encode(uploadFileName, StandardCharsets.UTF_8);

        // 브라우저에서 첨부파일이라는 것을 인지시키기 위한 헤더
        // 이 헤더가 없으면 클릭했을때 내용이 보여진다.
        String contentDisposition = "attachment; filename=\"" + encodedUploadFileName + "\"";

        return ResponseEntity.ok()
                .header(HttpHeaders.CONTENT_DISPOSITION, contentDisposition)
                .body(urlResource);
    }
}
```

- @GetMapping("/items/new") : 등록 폼을 보여준다.
- @PostMapping("/items/new") : 폼의 데이터를 저장하고 보여주는 화면으로 리다이렉트 한다.
- @GetMapping("/items/{id}") : 상품을 보여준다.
- @GetMapping("/images/{filename}") : <img> 태그로 이미지를 조회할 때 사용한다. UrlResource 로 이미지 파일을 읽어서 @ResponseBody 로 이미지 바이너리를 반환한다.
- @GetMapping("/attach/{itemId}") : 파일을 다운로드 할 때 실행한다. 예제를 더 단순화 할 수 있지만, 파일 다운로드 시 권한 체크같은 복잡한 상황까지 가정한다 생각하고 이미지 id 를 요청하도록 했다. 파일 다운로드시에는 고객이 업로드한 파일 이름으로 다운로드 하는게 좋다. 이때는 Content-Disposition 해더에 attachment; filename="업로드 파일명" 값을 주면 된다.

<br><br>

## View 코드

<br>

등록 폼 뷰 <br>
resources/templates/item-form.html
```html
<!DOCTYPE HTML>
<html xmlns:th="http://www.thymeleaf.org">
<head>
  <meta charset="utf-8">
</head>
<body>
<div class="container">
  <div class="py-5 text-center">
    <h2>상품 등록</h2>
  </div>
  <form th:action method="post" enctype="multipart/form-data">
    <ul>
      <li>상품명 <input type="text" name="itemName"></li>
      <li>첨부파일<input type="file" name="attachFile" ></li>
      <li>이미지 파일들<input type="file" multiple="multiple"
                        name="imageFiles" ></li>
    </ul>
    <input type="submit"/>
  </form>
</div> <!-- /container -->
</body>
</html>
```

다중 파일 업로드를 하려면 multiple="multiple" 옵션을 주면 된다. <br>
ItemForm 의 다음 코드에서 여러 이미지 파일을 받을 수 있다. <br>
private List<MultipartFile> imageFiles; <br>

<br>

조회 뷰<br>
resources/templates/item-view.html
```html
<!DOCTYPE HTML>
<html xmlns:th="http://www.thymeleaf.org">
<head>
  <meta charset="utf-8">
</head>
<body>
<div class="container">
  <div class="py-5 text-center">
    <h2>상품 조회</h2>
  </div>
  상품명: <span th:text="${item.itemName}">상품명</span><br/>
  첨부파일: <a th:if="${item.attachFile}" th:href="|/attach/${item.id}|" th:text="${item.getAttachFile().getUploadFileName()}" /><br/>
  <img th:each="imageFile : ${item.imageFiles}" th:src="|/images/${imageFile.getStoreFileName()}|" width="300" height="300"/>
</div> <!-- /container -->
</body>
</html>
```

첨부 파일은 링크로 걸어두고, 이미지는 <img> 태그를 반복해서 출력한다.<br>

## 결과 확인

- http://localhost:8080/items/new 접속후 상품 등록

![그림](https://sehwan-choi.github.io/assets/img/spring/MVC2/Upload/image.jpg)

<br>

![그림](https://sehwan-choi.github.io/assets/img/spring/MVC2/Upload/image2.jpg)

<br>

> 위와 같이 사진이 나올수 있는 이유 <br>
> 아래 코드의 downLoadImage로 이미지 경로를 보내주기 때문이다. 만일 아래 코드가 없다면 아래 사진과 같이 X박스가 나올 것이다.

```java
public class ItemController {
    ...

    @ResponseBody
    @GetMapping("/images/{filename}")
    public Resource downLoadImage(@PathVariable String filename) throws MalformedURLException {
        // 이미지를 보여주기 위함
        return new UrlResource("file:" + fileStore.getFullPath(filename));
    }

    ...
}
```

![그림](https://sehwan-choi.github.io/assets/img/spring/MVC2/Upload/image3.jpg)


<br>

- 상품등록후 첨부파일 다운로드

![그림](https://sehwan-choi.github.io/assets/img/spring/MVC2/Upload/image4.jpg)

> 위와 같이 첨부파일을 다운로드 받을수 있는 이유<br>
> 아래 코드의 downloadAttach로 파일에 대한 헤더 정보와, 경로를 보내주기 때문이다. <br>
> 주의할 점은 헤더에 HttpHeaders.CONTENT_DISPOSITION 를 반드시 포함해야 한다. 하지 않으면 파일 다운로드가 아닌 아래 사진과 같이 파일안의 내용이 보일 것이다. <br>
> 또한, 파일 이름이 한글로 되었거나 특수문자가 포함되있는 경우 브라우저에서 깨질 가능성이 있기 때문에 UriUtils를 통해서 반드시 인코딩을 해서 보낸다.

```java
public class ItemController {
    ...
    @GetMapping("/attach/{itemId}")
    public ResponseEntity<Resource> downloadAttach(@PathVariable Long itemId) throws MalformedURLException {
        Item item = itemRepository.findById(itemId);
        String storeFileName = item.getAttachFile().getStoreFileName();
        String uploadFileName = item.getAttachFile().getUploadFileName();

        UrlResource urlResource = new UrlResource("file:" + fileStore.getFullPath(storeFileName));

        log.info("uploadFileName= {}", uploadFileName);

        // 파일명이 깨지지 않도록 인코딩한다.
        String encodedUploadFileName = UriUtils.encode(uploadFileName, StandardCharsets.UTF_8);

        // 브라우저에서 첨부파일이라는 것을 인지시키기 위한 헤더
        // 이 헤더가 없으면 클릭했을때 내용이 보여진다.
        String contentDisposition = "attachment; filename=\"" + encodedUploadFileName + "\"";

        return ResponseEntity.ok()
                .header(HttpHeaders.CONTENT_DISPOSITION, contentDisposition)
                .body(urlResource);
    }
}
```
![그림](https://sehwan-choi.github.io/assets/img/spring/MVC2/Upload/image5.jpg)


<br><br><br>
## References 및 사진 출처

> __김영한의 스프링 MVC 2편 - 백엔드 웹 개발 활용 기술__