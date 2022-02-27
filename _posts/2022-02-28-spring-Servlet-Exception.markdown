---
layout: post
title:  "Servlet Exception 처리"
subtitle:   "Servlet Exception 처리"
date:   2022-02-28 02:00:27 +0900
categories: spring
tags: spring Servlet Exception
comments: true
---


<br>

- 목차
    - [서블릿 예외 처리](#서블릿-예외-처리)
        - [Exception(예외)](#exception예외)
        - [ServletExController - 서블릿 예외 컨트롤러](#servletexController---서블릿-예외-컨트롤러)
            - [sendError 흐름](#senderror-흐름)
    - [서블릿 예외 처리 - 오류 화면 제공](#서블릿-예외-처리---오류-화면-제공)
        - [서블릿 오류 페이지 등록](#서블릿-오류-페이지-등록)
        - [에러페이지 처리 컨트롤러](#에러페이지-처리-컨트롤러)
    - [서블릿 예외 처리 - 오류 페이지 작동 원리](#서블릿-예외-처리---오류-페이지-작동-원리)
    - [오류 정보 추가](#오류-정보-추가)
        - [ErrorPageController - 오류 출력](#errorpagecontroller---오류-출력)
        - [로그 확인](#로그-확인)
  

# 서블릿 예외 처리

<br>

스프링이 아닌 순수 서블릿 컨테이너는 예외를 어떻게 처리하는지 알아보자.

<br><br>

서블릿은 다음 2가지 방식으로 예외 처리를 지원한다.

- Exception (예외)
- response.sendError(HTTP 상태 코드, 오류 메시지)

<br>

## Exception(예외)

<br>

- 자바 직접 실행 <br>
자바의 메인 메서드를 직접 실행하는 경우 main 이라는 이름의 쓰레드가 실행된다. <br>
실행 도중에 예외를 잡지 못하고 처음 실행한 main() 메서드를 넘어서 예외가 던져지면, 예외 정보를 남기고 해당 쓰레드는 종료된다.

<br>

- 웹 애플리케이션 <br>
웹 애플리케이션은 사용자 요청별로 별도의 쓰레드가 할당되고, 서블릿 컨테이너 안에서 실행된다. <br>
애플리케이션에서 예외가 발생했는데, 어디선가 try ~ catch로 예외를 잡아서 처리하면 아무런 문제가 없다. 그런데 만약에 애플리케이션에서 예외를 잡지 못하고, 서블릿 밖으로 까지 예외가 전달되면 어떻게
동작할까?

```
WAS(여기까지 전파) <- 필터 <- 서블릿 <- 인터셉터 <- 컨트롤러(예외발생)
```

결국 톰캣 같은 WAS 까지 예외가 전달된다. WAS는 예외가 올라오면 어떻게 처리해야 할까? <br>
한번 테스트 해보자.
먼저 스프링 부트가 제공하는 기본 예외 페이지가 있는데 이건 꺼두자

<br>

application.properties
```
server.error.whitelabel.enabled=false
```

<br><br>

## ServletExController - 서블릿 예외 컨트롤러

<br>

ServletExController
```java
@Slf4j
@Controller
public class ServletExceptionController {

    @GetMapping("/error-ex")
    public void errorEx() {
        throw new RuntimeException("예외 발생!");
    }

    @GetMapping("/error-404")
    public void error404(HttpServletResponse response) throws IOException {
        response.sendError(404, "404 오류!");
    }

    @GetMapping("/error-500")
    public void error500(HttpServletResponse response) throws IOException {
        response.sendError(500);
    }
}
```

<br>

/error-ex 를 실행해보자. <br>
실행해보면 다음처럼 tomcat이 기본으로 제공하는 오류 화면을 볼 수 있다. <br>
```
HTTP Status 500 – Internal Server Error
```
웹 브라우저에서 개발자 모드로 확인해보면 HTTP 상태 코드가 500으로 보인다. Exception 의 경우 서버 내부에서 처리할 수 없는 오류가 발생한 것으로 생각해서 HTTP 상태 코드 500을 반환한다.<br><br>

/error-404, /error-500 을 실행해보자. <br>
오류가 발생했을 때 HttpServletResponse 가 제공하는 sendError 라는 메서드를 사용해도 된다. <br>
이것을 호출한다고 당장 예외가 발생하는 것은 아니지만, 서블릿 컨테이너에게 오류가 발생했다는 점을 전달할 수 있다.<br>
이 메서드를 사용하면 HTTP 상태 코드와 오류 메시지도 추가할 수 있다.
- response.sendError(HTTP 상태 코드)
- response.sendError(HTTP 상태 코드, 오류 메시지)

<br>

### sendError 흐름

```
WAS(sendError 호출 기록 확인) <- 필터 <- 서블릿 <- 인터셉터 <- 컨트롤러
(response.sendError())
```

response.sendError() 를 호출하면 response 내부에는 오류가 발생했다는 상태를 저장해둔다. <br>
그리고 서블릿 컨테이너는 고객에게 응답 전에 response 에 sendError() 가 호출되었는지 확인한다. <br>
그리고 호출되었다면 설정한 오류 코드에 맞추어 기본 오류 페이지를 보여준다. <br>
실행해보면 다음처럼 서블릿 컨테이너가 기본으로 제공하는 오류 화면을 볼 수 있다.

## 정리

<br>

서블릿 컨테이너가 제공하는 기본 예외 처리 화면은 사용자가 보기에 불편하다. 의미 있는 오류 화면을 제공해보자. <br>

<br><br>

# 서블릿 예외 처리 - 오류 화면 제공

<br>

서블릿 컨테이너가 제공하는 기본 예외 처리 화면은 고객 친화적이지 않다.  <br> 서블릿이 제공하는 오류 화면 기능을 사용해보자. <br>
서블릿은 Exception (예외)가 발생해서 서블릿 밖으로 전달되거나 또는 response.sendError() 가 호출 되었을 때 각각의 상황에 맞춘 오류 처리 기능을 제공한다. <br>
이 기능을 사용하면 친절한 오류 처리 화면을 준비해서 고객에게 보여줄 수 있다. <br>
과거에는 web.xml 이라는 파일에 다음과 같이 오류 화면을 등록했다.<br>

web.xml

```xml
<web-app>
    <error-page>
        <error-code>404</error-code>
        <location>/error-page/404.html</location>
    </error-page>
    <error-page>
        <error-code>500</error-code>
        <location>/error-page/500.html</location>
    </error-page>
    <error-page>
        <exception-type>java.lang.RuntimeException</exception-type>
        <location>/error-page/500.html</location>
    </error-page>
</web-app>
```

<br>

하지만 지금은 스프링 부트를 통해서 서블릿 컨테이너를 실행하기 때문에, 스프링 부트가 제공하는 기능을 사용해서 서블릿 오류 페이지를 등록하면 된다.

<br>

## 서블릿 오류 페이지 등록

<br>

WebServerCustomizer
```java
@Component
public class WebServerCustomizer implements WebServerFactoryCustomizer<ConfigurableWebServerFactory> {

    @Override
    public void customize(ConfigurableWebServerFactory factory) {
        ErrorPage errorPage404 = new ErrorPage(HttpStatus.NOT_FOUND, "/error-page/404");
        ErrorPage errorPage500 = new ErrorPage(HttpStatus.INTERNAL_SERVER_ERROR, "/error-page/500");
        ErrorPage errorPageEx = new ErrorPage(RuntimeException.class, "/error-page/500");

        factory.addErrorPages(errorPage404,errorPage500,errorPageEx);
    }
}
```

<br>

위에 ServletExceptionController에서 /error-ex, /error-404, /error-500 가 있었다.

- /error-404 호출시 errorPage404(/error-page/404)가 호출
- /error-500 호출시 errorPage500(/error-page/500) 호출
- /error-ex 호출시 RuntimeException 또는 그 자식 타입의 예외인 errorPageEx(error-page/500) 호출 

<br>

ErrorPage에서 각각 HttpStatus코드로 맵핑 시켜놓은 것이다.

500 예외가 서버 내부에서 발생한 오류라는 뜻을 포함하고 있기 때문에 여기서는 예외가 발생한 경우도 500 오류 화면으로 처리했다. <br>
오류 페이지는 예외를 다룰 때 해당 예외와 그 자식 타입의 오류를 함께 처리한다. 예를 들어서 위의 경우 RuntimeException 은 물론이고 RuntimeException 의 자식도 함께 처리한다. <br>
오류가 발생했을 때 처리할 수 있는 컨트롤러가 필요하다. 예를 들어서 RuntimeException 예외가 발생하면 errorPageEx 에서 지정한 /error-page/500 이 호출된다.

<br>

이어서 errorPage404(/error-page/404), errorPage500(/error-page/500), errorPageEx(/error-page/500) 을 처리할 Controller를 만들어 보자.

<br>

## 에러페이지 처리 컨트롤러

<br>

ErrorPageController
```java
@Slf4j
@Controller
public class ErrorPageController {

    @RequestMapping("/error-page/404")
    public String errorPage404(HttpServletRequest request, HttpServletResponse response) {
        log.info("errorPage 404");
        return "error-page/404";
    }

    @RequestMapping("/error-page/500")
    public String errorPage500(HttpServletRequest request, HttpServletResponse response) {
        log.info("errorPage 500");
        return "error-page/500";
    }
}
```

오류 처리 view는 생략했다. <br>
이제 모든 errorPage 처리는 ErrorPageController에서 처리될 것이다. 

<br><br>

# 서블릿 예외 처리 - 오류 페이지 작동 원리

<br>

서블릿은 Exception (예외)가 발생해서 서블릿 밖으로 전달되거나 또는 response.sendError() 가 호출 되었을 때 설정된 오류 페이지를 찾는다.

<br>

- 예외 발생 흐름
```
WAS(여기까지 전파) <- 필터 <- 서블릿 <- 인터셉터 <- 컨트롤러(예외발생)
```

<br>

- sendError 흐름
```
WAS(sendError 호출 기록 확인) <- 필터 <- 서블릿 <- 인터셉터 <- 컨트롤러
(response.sendError())
```

<br>

WAS는 해당 예외를 처리하는 오류 페이지 정보를 확인한다. <br>
new ErrorPage(RuntimeException.class, "/error-page/500")
예를 들어서 RuntimeException 예외가 WAS까지 전달되면, WAS는 오류 페이지 정보를 확인한다. <br>
확인해보니 RuntimeException 의 오류 페이지로 /error-page/500 이 지정되어 있다. WAS는 오류 페이지를 출력하기 위해 /error-page/500 를 다시 요청한다. <br>

<br>

- 오류 페이지 요청 흐름
```
WAS `/error-page/500` 다시 요청 -> 필터 -> 서블릿 -> 인터셉터 -> 컨트롤러(/error-page/
500) -> View
```

<br>

- 예외 발생과 오류 페이지 요청 흐름
```
1. WAS(여기까지 전파) <- 필터 <- 서블릿 <- 인터셉터 <- 컨트롤러(예외발생)
2. WAS `/error-page/500` 다시 요청 -> 필터 -> 서블릿 -> 인터셉터 -> 컨트롤러(/errorpage/500) -> View
```

<br>

> 중요한 점은 웹 브라우저(클라이언트)는 서버 내부에서 이런 일이 일어나는지 전혀 모른다는 점이다. 오직 서버 내부에서 오류 페이지를 찾기 위해 추가적인 호출을 한다. <br>

<br>

정리하면 다음과 같다.
1. 예외가 발생해서 WAS까지 전파된다.
2. WAS는 오류 페이지 경로를 찾아서 내부에서 오류 페이지를 호출한다. 이때 오류 페이지 경로로 필터, 서블릿, 인터셉터, 컨트롤러가 모두 다시 호출된다.

<br><br>

# 오류 정보 추가

<br>

WAS는 오류 페이지를 단순히 다시 요청만 하는 것이 아니라, 오류 정보를 request 의 attribute 에 추가해서 넘겨준다. <br>
필요하면 오류 페이지에서 이렇게 전달된 오류 정보를 사용할 수 있다.<br>

<br>

## ErrorPageController - 오류 출력

ErrorPageController
```java
@Slf4j
@Controller
public class ErrorPageController {

    //RequestDispatcher 상수로 정의되어 있음
    public static final String ERROR_EXCEPTION = "javax.servlet.error.exception";
    public static final String ERROR_EXCEPTION_TYPE = "javax.servlet.error.exception_type";
    public static final String ERROR_MESSAGE = "javax.servlet.error.message";
    public static final String ERROR_REQUEST_URI = "javax.servlet.error.request_uri";
    public static final String ERROR_SERVLET_NAME = "javax.servlet.error.servlet_name";
    public static final String ERROR_STATUS_CODE = "javax.servlet.error.status_code";

    @RequestMapping("/error-page/404")
    public String errorPage404(HttpServletRequest request, HttpServletResponse response) {
        log.info("errorPage 404");
        printErrorInfo(request);
        return "error-page/404";
    }

    @RequestMapping("/error-page/500")
    public String errorPage500(HttpServletRequest request, HttpServletResponse response) {
        log.info("errorPage 500");
        printErrorInfo(request);
        return "error-page/500";
    }

    private void printErrorInfo(HttpServletRequest request) {
        log.info("ERROR_EXCEPTION : {} " , request.getAttribute(ERROR_EXCEPTION));
        log.info("ERROR_EXCEPTION_TYPE : {} " , request.getAttribute(ERROR_EXCEPTION_TYPE));
        log.info("ERROR_MESSAGE : {} " , request.getAttribute(ERROR_MESSAGE));
        log.info("ERROR_REQUEST_URI : {} " , request.getAttribute(ERROR_REQUEST_URI));
        log.info("ERROR_SERVLET_NAME : {} " , request.getAttribute(ERROR_SERVLET_NAME));
        log.info("ERROR_STATUS_CODE : {} " , request.getAttribute(ERROR_STATUS_CODE));
        log.info("dispatchType={}", request.getDispatcherType());
    }
}
```

request.attribute에 서버가 담아준 정보 <br>
- javax.servlet.error.exception : 예외
- javax.servlet.error.exception_type : 예외 타입
- javax.servlet.error.message : 오류 메시지
- javax.servlet.error.request_uri : 클라이언트 요청 URI
- javax.servlet.error.servlet_name : 오류가 발생한 서블릿 이름
- javax.servlet.error.status_code : HTTP 상태 코드

<br>

## 로그 확인

- /error-ex 호출시
```xml
ERROR_EXCEPTION_TYPE : class java.lang.RuntimeException 
ERROR_MESSAGE : Request processing failed; nested exception is java.lang.RuntimeException: 예외 발생! <- RuntimeException시 message가 출력 된다.
ERROR_REQUEST_URI : /error-ex 
ERROR_SERVLET_NAME : dispatcherServlet 
ERROR_STATUS_CODE : 500 
dispatchType=ERROR
```

<br>

- /error-404 호출시
```xml
errorPage 404
ERROR_EXCEPTION : null 
ERROR_EXCEPTION_TYPE : null 
ERROR_MESSAGE : 404 오류! <- response.sendError시 message가 출력된다.
ERROR_REQUEST_URI : /error-404 
ERROR_SERVLET_NAME : dispatcherServlet 
ERROR_STATUS_CODE : 404 
dispatchType=ERROR
```

<br>

- /error-500 호출시
```xml
errorPage 500
ERROR_EXCEPTION : null 
ERROR_EXCEPTION_TYPE : null 
ERROR_MESSAGE :  
ERROR_REQUEST_URI : /error-500 
ERROR_SERVLET_NAME : dispatcherServlet 
ERROR_STATUS_CODE : 500 
dispatchType=ERROR
```

위와같이 확인 할 수 있다.


<br><br><br>
## References 및 사진 출처

> __김영한의 스프링 MVC 2편 - 백엔드 웹 개발 활용 기술__