---
layout: post
title:  "spring Binding result"
subtitle:   "spring Binding result"
date:   2022-02-13 20:45:27 +0900
categories: spring
tags: spring Validation BindingResult
comments: true
---


<br>

- 목차
	- [BindingResult](#bindingresult)
	- [BindingResult를 사용하지 않는경우](#bindingresult를_사용하지_않는경우)
	- [BindingResult를 사용하는 경우](#bindingresult를_사용하는_경우)
	- [주의](#주의)
    - [결과](#결과)
    
<br>

# BindingResult

<br>

스프링이 제공하는 검증 오류 처리 방법중 하나이다.

![그림](https://sehwan-choi.github.io/assets/img/spring/MVC2/image1.jpg)

위 사진에서와 같이 ```가격``` 입력칸에 숫자가 아닌 문자가 입력되면 어떻게 처맇할것인가?

<br><br>

# BindingResult를 사용하지 않는경우

```java
@PostMapping("/add")
    public String addItemV1(@ModelAttribute Item item, RedirectAttributes redirectAttributes) {

        Map<String, String> errors = new HashMap<>();

        ...
        
        if (item.getPrice() == null || item.getPrice() < 1000 || item.getPrice() > 1000000) {
            errors.add("price","가격은 1,000 ~ 1,000,000 까지 허용합니다.");
        }
        ...

        //검증에 실패하면 다시 입력 폼으로
        if (!errors.isEmpty()) {
            log.info("errors = {} " , errors);
            return "validation/v2/addForm";
        }

    }
```

위와 같은 예제 코드가 있을때 결과는 어떻게 될까?

![그림](https://sehwan-choi.github.io/assets/img/spring/MVC2/image2.jpg)

위 사진과 같을것이다. Controller에 오기전에 이미 에러가 발생한 것이다.

그렇다면 BindingResult를 사용한다면? 어떻게 될까?

<br><br>

# BindingResult를 사용하는 경우

```java
 @PostMapping("/add")
    public String addItemV1(@ModelAttribute Item item, BindingResult bindingResult, RedirectAttributes redirectAttributes) {

        if (!StringUtils.hasText(item.getItemName())) {
            bindingResult.addError(new FieldError("item","itemName","상품 이름은 필수 입니다."));
        }
        if (item.getPrice() == null || item.getPrice() < 1000 || item.getPrice() > 1000000) {
            bindingResult.addError(new FieldError("item","price","가격은 1,000 ~ 1,000,000 까지 허용합니다."));
        }

        if (item.getQuantity() == null || item.getQuantity() >= 9999) {
            bindingResult.addError(new FieldError("item","quantity", "수량은 최대 9,999 까지 허용합니다."));
        }

        // 복합 룰 검증
        if( item.getPrice() != null && item.getQuantity() != null) {
            int result = item.getPrice() * item.getQuantity();
            if ( result < 10000) {
                bindingResult.addError(new ObjectError("item", "가격 * 수량의 합은 10,000원 이상이어야 합니다."));
            }
        }

        //검증에 실퍃하면 다시 입력 폼으로
        if (bindingResult.hasErrors()) {
            log.info("bindingResult = {} " , bindingResult);
            return "validation/v2/addForm";
        }

        //검증 성공시

        Item savedItem = itemRepository.save(item);
        redirectAttributes.addAttribute("itemId", savedItem.getId());
        redirectAttributes.addAttribute("status", true);
        return "redirect:/validation/v2/items/{itemId}";
    }
```

![그림](https://sehwan-choi.github.io/assets/img/spring/MVC2/image3.jpg)

위 사진과 같이 놀랍게도, Controller까지 호출이 되는 것을 볼 수 있다.

<br><br>

# 주의!
BindingResult bindingResult 파라미터의 위치는 @ModelAttribute Item item 다음에 와야 한다. 그래야 Item에 대한 바인딩 결과가 저장이 된다.

```java
public String addItemV1(@ModelAttribute Item item, BindingResult bindingResult, 
                            ( item 뒤에)               ( BindingResult 가 있어야지 item에 대한 바인딩 정보가 담긴다.)
```

<br><br>

Controller에서의 로그도 확인해보자

```
Field error in object 'item' on field 'price': rejected value [qqq]; codes [typeMismatch.item.price,typeMismatch.price,typeMismatch.java.lang.Integer,typeMismatch]; arguments [org.springframework.context.support.DefaultMessageSourceResolvable: codes [item.price,price]; arguments []; default message [price]]; default message [Failed to convert property value of type 'java.lang.String' to required type 'java.lang.Integer' for property 'price'; nested exception is java.lang.NumberFormatException: For input string: "qqq"]
Field error in object 'item' on field 'itemName': rejected value [null]; codes []; arguments []; default message [상품 이름은 필수 입니다.]
Field error in object 'item' on field 'price': rejected value [null]; codes []; arguments []; default message [가격은 1,000 ~ 1,000,000 까지 허용합니다.]
Field error in object 'item' on field 'quantity': rejected value [null]; codes []; arguments []; default message [수량은 최대 9,999 까지 허용합니다.]Field error in object 'item' on field 'price': rejected value [qqq]; codes [typeMismatch.item.price,typeMismatch.price,typeMismatch.java.lang.Integer,typeMismatch]; arguments [org.springframework.context.support.DefaultMessageSourceResolvable: codes [item.price,price]; arguments []; default message [price]]; default message [Failed to convert property value of type 'java.lang.String' to required type 'java.lang.Integer' for property 'price'; nested exception is java.lang.NumberFormatException: For input string: "qqq"]
Field error in object 'item' on field 'itemName': rejected value [null]; codes []; arguments []; default message [상품 이름은 필수 입니다.]
Field error in object 'item' on field 'price': rejected value [null]; codes []; arguments []; default message [가격은 1,000 ~ 1,000,000 까지 허용합니다.]
Field error in object 'item' on field 'quantity': rejected value [null]; codes []; arguments []; default message [수량은 최대 9,999 까지 허용합니다.]
```

<br><br><br>

# 결과

@ModelAttribute에 바인딩 시 타입 오류가 발생하면? 
<br><br>
___BindingResult 가 없으면 400 오류가 발생하면서 컨트롤러가 호출되지 않고, 오류 페이지로
이동한다.___
<br><br>
___BindingResult 가 있으면 오류 정보( FieldError )를 BindingResult 에 담아서 컨트롤러를
정상 호출한다.___


<br><br><br>
## References 및 사진 출처

> __김영한의 스프링 MVC 2편 - 백엔드 웹 개발 활용 기술__