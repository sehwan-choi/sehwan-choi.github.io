---
layout: post
title:  "Spring API 예외 처리 - ControllerAdvice"
subtitle:   "Spring API 예외 처리 - ControllerAdvice"
date:   2022-03-02 02:30:27 +0900
categories: spring
tags: spring ControllerAdvice
comments: true
---


<br>

- 목차
    - [API 예외 처리 - @ControllerAdvice](#api-예외-처리---controlleradvice)
    - [@ControllerAdvice](#controlleradvice)
    - [대상 컨트롤러 지정 방법](#대상-컨트롤러-지정-방법)
    - [정리](#정리)
    
<br>

# API 예외 처리 - @ControllerAdvice

<br>

@ExceptionHandler 를 사용해서 예외를 깔끔하게 처리할 수 있게 되었지만, 정상 코드와 예외 처리 코드가 하나의 컨트롤러에 섞여 있다. <br> @ControllerAdvice 또는 @RestControllerAdvice 를 사용하면 둘을 분리할 수 있다.

<br>

ExControllerAdvice
```java
@Slf4j
@RestControllerAdvice(basePackages = "hello.exception.api")
public class ExControllerAdvice {

    @ResponseStatus(HttpStatus.BAD_REQUEST)
    @ExceptionHandler(IllegalArgumentException.class)
    public ErrorResult illegalExHanlder(IllegalArgumentException e) {
        log.error("[exceptionHandler] ex",e);
        return new ErrorResult("BAD", e.getMessage());
    }

    @ExceptionHandler
    public ResponseEntity<ErrorResult> userExHandler(UserException e) {
        log.error("[exceptionHandler] ex",e);
        ErrorResult errorResult = new ErrorResult("USER-EX", e.getMessage());
        return new ResponseEntity<>(errorResult, HttpStatus.BAD_REQUEST);
    }

    @ResponseStatus(HttpStatus.INTERNAL_SERVER_ERROR)
    @ExceptionHandler
    public ErrorResult exHandler(Exception e) {
        log.error("[exceptionHandler] ex",e);
        return new ErrorResult("EX", "내부 오류");
    }
}
```

<br>

ApiExceptionController
```java
@RestController
@Slf4j
@RequestMapping("/api")
public class ApiExceptionController {

    @GetMapping("/members/{id}")
    public MemberDto getMember(@PathVariable("id") String id) {

        if (id.equals("ex")) {
            throw new RuntimeException("잘못된 사용자");
        }
        if (id.equals("bad")) {
            throw new IllegalArgumentException("잘못된 입력 값");
        }
        if (id.equals("user-ex")) {
            throw new UserException("사용자 오류");
        }
        return new MemberDto(id,"hello " + id);
    }

    @Data
    @AllArgsConstructor
    static class MemberDto {
        private String memberId;
        private String name;
    }
}
```

위 코드와 같이 ExControllerAdvice에서 @RestControllerAdvice를 통해 모든 예외를 공통으로 처리 할수 있다.

<br><br>

# @ControllerAdvice

<br>

- @ControllerAdvice 는 대상으로 지정한 여러 컨트롤러에 @ExceptionHandler , @InitBinder 기능을 부여해주는 역할을 한다.
- @ControllerAdvice 에 대상을 지정하지 않으면 모든 컨트롤러에 적용된다. (글로벌 적용) 
- @RestControllerAdvice 는 @ControllerAdvice 와 같고, @ResponseBody 가 추가되어 있다.
- @Controller , @RestController 의 차이와 같다.

<br><br>

# 대상 컨트롤러 지정 방법

```java
// RestController 어노테이션이 붙은 Controller만
// Target all Controllers annotated with @RestController
@ControllerAdvice(annotations = RestController.class)
public class ExampleAdvice1 {}

// 해당 패키지 하위의 모든 Controller만
// Target all Controllers within specific packages
@ControllerAdvice("org.example.controllers")
public class ExampleAdvice2 {}

//특정 Controller만
// Target all Controllers assignable to specific classes
@ControllerAdvice(assignableTypes = {ControllerInterface.class,
AbstractController.class})
public class ExampleAdvice3 {}
```

스프링 공식 문서 예제에서 보는 것 처럼 특정 애노테이션이 있는 컨트롤러를 지정할 수 있고, 특정 패키지를 직접 지정할 수도 있다. 패키지 지정의 경우 해당 패키지와 그 하위에 있는 컨트롤러가 대상이 된다. 그리고 특정 클래스를 지정할 수도 있다. <br>
대상 컨트롤러 지정을 생략하면 모든 컨트롤러에 적용된다.<br>

advice 공식문서를 보려면 아래 링크를 클릭해보자.

## [Spring - advice](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-controller-advice)

<br><br>

# 정리

@ExceptionHandler 와 @ControllerAdvice 를 조합하면 예외를 깔끔하게 해결할 수 있다.

<br><br><br>
## References 및 사진 출처

> __김영한의 스프링 MVC 2편 - 백엔드 웹 개발 활용 기술__