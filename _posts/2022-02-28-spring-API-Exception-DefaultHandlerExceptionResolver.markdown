---
layout: post
title:  "Spring API 예외 처리 - DefaultHandlerExceptionResolver"
subtitle:   "Spring API 예외 처리 - DefaultHandlerExceptionResolver"
date:   2022-03-02 01:18:27 +0900
categories: spring
tags: spring DefaultHandlerExceptionResolver
comments: true
---


<br>

- 목차
    - [DefaultHandlerExceptionResolver](#defaulthandlerexceptionresolver)
    - [코드 확인](#코드-확인)


# DefaultHandlerExceptionResolver

<br>

DefaultHandlerExceptionResolver 는 스프링 내부에서 발생하는 스프링 예외를 해결한다. <br>
대표적으로 파라미터 바인딩 시점에 타입이 맞지 않으면 내부에서 TypeMismatchException 이 발생하는데, 이 경우 예외가 발생했기 때문에 그냥 두면 서블릿 컨테이너까지 오류가 올라가고, 결과적으로 500 오류가 발생한다. <br>
그런데 파라미터 바인딩은 대부분 클라이언트가 HTTP 요청 정보를 잘못 호출해서 발생하는 문제이다.  <br>
HTTP 에서는 이런 경우 HTTP 상태 코드 400을 사용하도록 되어 있다.
DefaultHandlerExceptionResolver 는 이것을 500 오류가 아니라 HTTP 상태 코드 400 오류로 변경한다. <br>
스프링 내부 오류를 어떻게 처리할지 수 많은 내용이 정의되어 있다. <br>

<br><br>

# 코드 확인

<br>

DefaultHandlerExceptionResolver
```java
public class DefaultHandlerExceptionResolver extends AbstractHandlerExceptionResolver {
    protected ModelAndView doResolveException(
                HttpServletRequest request, HttpServletResponse response, @Nullable Object handler, Exception ex) {

            try {
                    ...

                else if (ex instanceof TypeMismatchException) {
                    return handleTypeMismatch(
                            (TypeMismatchException) ex, request, response, handler);
                }
                
                    ...
            }
            catch (Exception handlerEx) {
                if (logger.isWarnEnabled()) {
                    logger.warn("Failure while trying to resolve exception [" + ex.getClass().getName() + "]", handlerEx);
                }
            }
            return null;
        }
    }

    protected ModelAndView handleTypeMismatch(TypeMismatchException ex,
                HttpServletRequest request, HttpServletResponse response, @Nullable Object handler) throws IOException {

            response.sendError(HttpServletResponse.SC_BAD_REQUEST);
            return new ModelAndView();
	}
}
```

DefaultHandlerExceptionResolver.handleTypeMismatch 를 보면 위와 같은 코드를 확인할 수 있다. <br>
response.sendError(HttpServletResponse.SC_BAD_REQUEST) (400)
결국 response.sendError() 를 통해서 문제를 해결한다. <br>
sendError(400) 를 호출했기 때문에 WAS에서 다시 오류 페이지( /error )를 내부 요청한다. <br>

<br>

ApiExceptionController
```java
@Slf4j
@RestController
public class ApiExceptionController {

    @GetMapping("/api/default-handler-ex")
    public String defaultException(@RequestParam Integer data) {
        return "ok";
    }
}
```

/api/default-handler-ex?data=qqq와 같이 API 요청을 시도해보자. <br>
Integer data 에 문자를 입력했기 때문에 내부에서 TypeMismatchException 이 발생한다.

<br>

```json
{
    "timestamp": "2022-03-01T16:23:28.491+00:00",
    "status": 400,
    "error": "Bad Request",
    "exception": "org.springframework.web.method.annotation.MethodArgumentTypeMismatchException",
    "message": "Failed to convert value of type 'java.lang.String' to required type 'java.lang.Integer'; nested exception is java.lang.NumberFormatException: For input string: \"qqq\"",
    "path": "/api/default-handler-ex"
}
```

실행 결과를 보면 HTTP 상태 코드가 400인 것을 확인할 수 있다.

<br><br><br>
## References 및 사진 출처

> __김영한의 스프링 MVC 2편 - 백엔드 웹 개발 활용 기술__