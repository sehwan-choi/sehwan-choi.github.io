---
layout: post
title:  "Spring API 예외 처리"
subtitle:   "Spring API 예외 처리"
date:   2022-03-02 00:56:27 +0900
categories: spring
tags: spring Exception
comments: true
---


<br>

- 목차
    - [API 예외 처리 - 스프링이 제공하는 ExceptionResolver](#api-예외-처리---스프링이-제공하는-exceptionresolver)
    - [ControllerAdvice](#controlleradvice)
    - [EexceptionHandlerExceptionResolver](#exceptionhandlerexceptionresolver)
    - [ResponseStatusExceptionResolver](#responsestatusexceptionresolver)
    - [DefaultHandlerExceptionResolver](#defaulthandlerexceptionresolver)


# API 예외 처리 - 스프링이 제공하는 ExceptionResolver

<br>

스프링 부트가 기본으로 제공하는 ExceptionResolver 는 다음과 같다. <br>

HandlerExceptionResolverComposite 에 다음 순서로 등록
1. ExceptionHandlerExceptionResolver
2. ResponseStatusExceptionResolver
3. DefaultHandlerExceptionResolver 우선 순위가 가장 낮다.
     
<br>

# ExceptionHandlerExceptionResolver

<br>

@ExceptionHandler 을 처리한다. API 예외 처리는 대부분 이 기능으로 해결한다. <br>

## [@ExceptionHandler 대한 설명 보러가기](https://sehwan-choi.github.io/spring/2022/03/01/spring-API-Exception-ExceptionHandler/)

<br><br>

# ControllerAdvice

<br>

@ExceptionHandler 를 사용해서 예외를 깔끔하게 처리할 수 있게 되었지만, 정상 코드와 예외 처리 코드가 하나의 컨트롤러에 섞여 있다. 이것을 해결시켜줄 기술이다.


## [@ControllerAdvice 대한 설명 보러가기](https://sehwan-choi.github.io/spring/2022/03/01/spring-API-Exception-ExceptionHandler/)



<br><br>


# ResponseStatusExceptionResolver

<br>

HTTP 상태 코드를 지정해준다. <br>
예) @ResponseStatus(value = HttpStatus.NOT_FOUND) <br>

## [@ResponseStatusExceptionResolver 대한 설명 보러가기](https://sehwan-choi.github.io/spring/2022/03/01/spring-API-Exception-ResponseStatusExceptionResolver/)

<br><br>

# DefaultHandlerExceptionResolver

<br>

스프링 내부 기본 예외를 처리한다. <br>

## [@DefaultHandlerExceptionResolver 대한 설명 보러가기](https://sehwan-choi.github.io/spring/2022/03/01/spring-API-Exception-DefaultHandlerExceptionResolver/)

<br><br><br>
## References 및 사진 출처

> __김영한의 스프링 MVC 2편 - 백엔드 웹 개발 활용 기술__