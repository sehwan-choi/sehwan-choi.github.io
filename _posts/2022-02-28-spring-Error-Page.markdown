---
layout: post
title:  "Spring Boot가 제공하는 Error Page"
subtitle:   "Spring Boot가 제공하는 Error Page"
date:   2022-03-01 19:00:27 +0900
categories: spring
tags: spring ErrorPage
comments: true
---


<br>

- 목차
    - [스프링 부트 - 오류 페이지1](#스프링-부트---오류-페이지1)
        - [개발자는 오류 페이지만 등록](#개발자는-오류-페이지만-등록)
        - [뷰 선택 우선순위](#뷰-선택-우선순위)
        - [오류 페이지 등록](#오류-페이지-등록)
    - [스프링 부트 - 오류 페이지2](#스프링-부트---오류-페이지2)
        - [BasicErrorController가 제공하는 기본 정보들](#basicerrorcontroller가-제공하는-기본-정보들)
    - [스프링 부트 오류 관련 옵션](#스프링-부트-오류-관련-옵션)
        - [옵션](#옵션)
        - [확장 포인트](#확장-포인트)
    - [정리](#정리)

     
<br>

# 스프링 부트 - 오류 페이지1

<br>

- 스프링 부트는 ErrorPage 를 자동으로 등록한다. 이때 /error 라는 경로로 기본 오류 페이지를 설정한다.
    - new ErrorPage("/error") , 상태코드와 예외를 설정하지 않으면 기본 오류 페이지로 사용된다.
    - 서블릿 밖으로 예외가 발생하거나, response.sendError(...) 가 호출되면 모든 오류는 /error 를 호출하게 된다. 
- BasicErrorController 라는 스프링 컨트롤러를 자동으로 등록한다.
    - ErrorPage 에서 등록한 /error 를 매핑해서 처리하는 컨트롤러다.

<br>

> 참고 <br>
> ErrorMvcAutoConfiguration 이라는 클래스가 오류 페이지를 자동으로 등록하는 역할을 한다.

<br>

이제 오류가 발생했을 때 오류 페이지로 /error 를 기본 요청한다. 스프링 부트가 자동 등록한 BasicErrorController 는 이 경로를 기본으로 받는다.

<br><br>

## 개발자는 오류 페이지만 등록

<br>

BasicErrorController 는 기본적인 로직이 모두 개발되어 있다. <br>
개발자는 오류 페이지 화면만 BasicErrorController 가 제공하는 룰과 우선순위에 따라서 등록하면 된다. 정적 HTML이면 정적 리소스, 뷰 템플릿을 사용해서 동적으로 오류 화면을 만들고 싶으면 뷰 템플릿 경로에 오류 페이지 파일을 만들어서 넣어두기만 하면 된다.

<br><br>

## 뷰 선택 우선순위

<br> 

1. 뷰 템플릿
    - 1). : resources/templates/error/500.html
    - 2). resources/templates/error/5xx.html
2. 정적 리소스( static , public )
    - 1). : resources/static/error/400.html
    - 2). : resources/static/error/404.html
    - 3). : resources/static/error/4xx.html
3. 적용 대상이 없을 때 뷰 이름( error )
    - resources/templates/error.html

<br>

해당 경로 위치에 HTTP 상태 코드 이름의 뷰 파일을 넣어두면 된다. <br>
뷰 템플릿이 정적 리소스보다 우선순위가 높고, 404, 500처럼 구체적인 것이 5xx처럼 덜 구체적인 것 보다 우선순위가 높다.<br>
5xx, 4xx 라고 하면 500대, 400대 오류를 처리해준다.

<br><br>

## 오류 페이지 등록

<br>

- resources/templates/error/4xx.html
- resources/templates/error/404.html
- resources/templates/error/500.html

<br>

위와 같이 오류페이지를 등록 했을때, 404번 에러 발생시 404.html을 , 500번 에러 발생시 500.html을 404번 이외에 에러 발생시 4xx.html을 스프링부트가 자동으로 호출해준다.

<br><br>

# 스프링 부트 - 오류 페이지2

<br>

## BasicErrorController가 제공하는 기본 정보들

<br>

BasicErrorController 컨트롤러는 다음 정보를 model에 담아서 뷰에 전달한다. 뷰 템플릿은 이 값을
활용해서 출력할 수 있다.

```xml
* timestamp: Fri Feb 05 00:00:00 KST 2021
* status: 400
* error: Bad Request
* exception: org.springframework.validation.BindException
* trace: 예외 trace
* message: Validation failed for object='data'. Error count: 1
* errors: Errors(BindingResult)
* path: 클라이언트 요청 경로 (`/hello`)
```

<br>

500.html
```html
<!DOCTYPE HTML>
<html xmlns:th="http://www.thymeleaf.org">
<head>
  <meta charset="utf-8">
</head>
<body>
<div class="container" style="max-width: 600px">
  <div class="py-5 text-center">
    <h2>500 오류 화면 스프링 부트 제공</h2>
  </div>
  <div>
    <p>오류 화면 입니다.</p>
  </div>
  <ul>
    <li>오류 정보</li>
    <ul>
      <li th:text="|timestamp: ${timestamp}|"></li>
      <li th:text="|path: ${path}|"></li>
      <li th:text="|status: ${status}|"></li>
      <li th:text="|message: ${message}|"></li>
      <li th:text="|error: ${error}|"></li>
      <li th:text="|exception: ${exception}|"></li>
      <li th:text="|errors: ${errors}|"></li>
      <li th:text="|trace: ${trace}|"></li>
    </ul>
    </li>
  </ul>
  <hr class="my-4">
</div> <!-- /container -->
</body>
</html>
```

위 에러 발생시 500.html을 호출하게 되면 아래와 같은 정보들이 나온다. <br>
하지만 모든 정보가 공개되는 것은 아니다. 오류 관련 내부 정보들을 고객에게 노출하는 것은 좋지 않다. 고객이 해당 정보를 읽어도 혼란만 더해지고, 보안상 문제가 될 수도 있다. <br>
그래서 BasicErrorController 오류 컨트롤러에서 다음 오류 정보를 model 에 포함할지 여부 선택할 수 있다.

<br>

application.properties
```properties
server.error.include-exception=false : true/false
server.error.include-message=never : never/always/on_param
server.error.include-stacktrace=never : never/always/on_param
server.error.include-binding-errors=never : never/always/on_param
```

기본 값이 never 인 부분은 다음 3가지 옵션을 사용할 수 있다. <br>
```
never, always, on_param
```

- never : 사용하지 않음
- always :항상 사용
- on_param : 파라미터가 있을 때 사용
- on_param 은 파라미터가 있으면 해당 정보를 노출한다. 디버그 시 문제를 확인하기 위해 사용할 수 있다. 

그런데 이 부분도 개발 서버에서 사용할 수 있지만, 운영 서버에서는 권장하지 않는다. <br>
on_param 으로 설정하고 다음과 같이 HTTP 요청시 파라미터를 전달하면 해당 정보들이 model 에 담겨서 뷰 템플릿에서 출력된다.

```
message=&errors=&trace=
ex). http://localhost:8080/error-ex?message=&errors=&trace=
```

실무에서는 이것들을 노출하면 안된다! 사용자에게는 이쁜 오류 화면과 고객이 이해할 수 있는 간단한 오류 메시지를 보여주고 오류는 서버에 로그로 남겨서 로그로 확인해야 한다. <br>

<br><br>

# 스프링 부트 오류 관련 옵션

<br>

## 옵션

<br>

- server.error.whitelabel.enabled=true : 오류 처리 화면을 못 찾을 시, 스프링 whitelabel 오류 페이지 적용
- server.error.path=/error : 오류 페이지 경로, 스프링이 자동 등록하는 서블릿 글로벌 오류 페이지 경로와 BasicErrorController 오류 컨트롤러 경로에 함께 사용된다.

<br>

## 확장 포인트

<br>

에러 공통 처리 컨트롤러의 기능을 변경하고 싶으면 ErrorController 인터페이스를 상속 받아서 구현하거나 BasicErrorController 상속 받아서 기능을 추가하면 된다.

<br><br>

# 정리

<br>

스프링 부트가 기본으로 제공하는 오류 페이지를 활용하면 오류 페이지와 관련된 대부분의 문제는 손쉽게 해결할 수 있다.

<br><br><br>
## References 및 사진 출처

> __김영한의 스프링 MVC 2편 - 백엔드 웹 개발 활용 기술__