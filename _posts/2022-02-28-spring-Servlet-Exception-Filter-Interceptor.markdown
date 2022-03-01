---
layout: post
title:  "Servlet Exception 처리 - Filter"
subtitle:   "Servlet Exception 처리 - Filter"
date:   2022-03-01 18:00:27 +0900
categories: spring
tags: spring Servlet Filter
comments: true
---


<br>

- 목차
    - [서블릿 예외 처리 - 필터](#서블릿-예외-처리---필터)
    - [예외 발생과 오류 페이지 요청 흐름](#예외-발생과-오류-페이지-요청-흐름)
    - [DispatcherType](#dispatchertype)
    - [필터와 DispatcherType](#필터와-dispatchertype)
    - [로그 확인](#로그-확인)

     
<br>

# 서블릿 예외 처리 - 필터

<br><br>

# 예외 발생과 오류 페이지 요청 흐름

<br>

```
1. WAS(여기까지 전파) <- 필터 <- 서블릿 <- 인터셉터 <- 컨트롤러(예외발생)

2. WAS `/error-page/500` 다시 요청 -> 필터 -> 서블릿 -> 인터셉터 -> 컨트롤러(/error-page/500) -> View
```

오류가 발생하면 오류 페이지를 출력하기 위해 WAS 내부에서 다시 한번 호출이 발생한다. 이때 필터, 서블릿, 인터셉터도 모두 다시 호출된다. <br>
그런데 로그인 인증 체크 같은 경우를 생각해보면, 이미 한번 필터나, 인터셉터에서 로그인 체크를 완료했다. 따라서 서버 내부에서 오류 페이지를 호출한다고 해서 해당 필터나 인터셉트가 한번 더 호출되는 것은 __매우 비효율적이다.__ <br>
결국 클라이언트로 부터 발생한 정상 요청인지, 아니면 오류 페이지를 출력하기 위한 내부 요청인지 구분할 수 있어야 한다. 서블릿은 이런 문제를 해결하기 위해 DispatcherType 이라는 추가 정보를 제공한다. <br>

<br><br>

# DispatcherType

<br>

필터는 이런 경우를 위해서 dispatcherTypes 라는 옵션을 제공한다.
다음 아래의 로그를 추가 하고 출력해보면 오류 페이지에서 dispatchType=ERROR 로 나오는 것을 확인할 수 있다.
```
log.info("dispatchType={}", request.getDispatcherType())
```
고객이 처음 요청하면 dispatcherType=REQUEST 이다.
이렇듯 서블릿 스펙은 실제 고객이 요청한 것인지, 서버가 내부에서 오류 페이지를 요청하는 것인지 DispatcherType 으로 구분할 수 있는 방법을 제공한다.

<br>

javax.servlet.DispatcherType
```
public enum DispatcherType {
    FORWARD,
    INCLUDE,
    REQUEST,
    ASYNC,
    ERROR
}
```
<br>

- REQUEST : 클라이언트 요청

- ERROR : 오류 요청

- FORWARD : MVC에서 배웠던 서블릿에서 다른 서블릿이나 JSP를 호출할 때 RequestDispatcher.forward(request, response);

- INCLUDE : 서블릿에서 다른 서블릿이나 JSP의 결과를 포함할 때

- RequestDispatcher.include(request, response);

- ASYNC : 서블릿 비동기 호출

<br><br>

# 필터와 DispatcherType

<br>

- LogFilter 추가

<br>

LogFilter
```java
@Slf4j
public class LogFilter implements Filter {
    @Override
    public void init(FilterConfig filterConfig) throws ServletException {
        log.info("log filter init");
    }
    @Override
    public void doFilter(ServletRequest request, ServletResponse response,
                         FilterChain chain) throws IOException, ServletException {
        HttpServletRequest httpRequest = (HttpServletRequest) request;
        String requestURI = httpRequest.getRequestURI();
        String uuid = UUID.randomUUID().toString();
        try {
            log.info("REQUEST [{}][{}][{}]", uuid,
                    request.getDispatcherType(), requestURI);
            chain.doFilter(request, response);
        } catch (Exception e) {
            log.info("filter exception {}", e.getMessage());
            throw e;
        } finally {
            log.info("RESPONSE [{}][{}][{}]", uuid,
                    request.getDispatcherType(), requestURI);
        }
    }
    @Override
    public void destroy() {
        log.info("log filter destroy");
    }
}
```

<br>

- WebConfig에 LogFilter 추가

<br>

WebConfig
```java
@Configuration
public class WebConfig implements WebMvcConfigurer {

    @Bean
    public FilterRegistrationBean logFilter() {
        FilterRegistrationBean<Filter> filterFilterRegistrationBean = new FilterRegistrationBean<Filter>();
        filterFilterRegistrationBean.setFilter(new LogFilter());
        filterFilterRegistrationBean.setOrder(1);
        filterFilterRegistrationBean.addUrlPatterns("/*");
        filterFilterRegistrationBean.setDispatcherTypes(DispatcherType.REQUEST, DispatcherType.ERROR);
        return filterFilterRegistrationBean;
    }
}
```

<br>

```
filterRegistrationBean.setDispatcherTypes(DispatcherType.REQUEST, 
DispatcherType.ERROR);
```
위와 같이 두 가지를 모두 넣으면 클라이언트 요청은 물론이고, 오류 페이지 요청에서도 필터가 호출된다. <br>
아무것도 넣지 않으면 기본 값이 DispatcherType.REQUEST 이다. 즉 클라이언트의 요청이 있는 경우에만 필터가 적용된다. 특별히 오류 페이지 경로도 필터를 적용할 것이 아니면, 기본 값을 그대로 사용하면 된다. <br>
물론 오류 페이지 요청 전용 필터를 적용하고 싶으면 DispatcherType.ERROR 만 지정하면 된다.

<br>

# 로그 확인
```xml
2022-03-01 18:10:40.719  INFO 37632 --- [nio-8080-exec-1] hello.exception.Filter.LogFilter         : REQUEST [fc8b7f5f-9999-46be-a8ff-a6b77f5acd99][REQUEST][/error-ex] <- DispatcherType이 REQUEST이다
2022-03-01 18:10:40.741  INFO 37632 --- [nio-8080-exec-1] hello.exception.Filter.LogFilter         : filter exception Request processing failed; nested exception is java.lang.RuntimeException: 예외 발생!
2022-03-01 18:10:40.742  INFO 37632 --- [nio-8080-exec-1] hello.exception.Filter.LogFilter         : RESPONSE [fc8b7f5f-9999-46be-a8ff-a6b77f5acd99][REQUEST][/error-ex]
2022-03-01 18:10:40.746 ERROR 37632 --- [nio-8080-exec-1] o.a.c.c.C.[.[.[/].[dispatcherServlet]    : Servlet.service() for servlet [dispatcherServlet] in context with path [] threw exception [Request processing failed; nested exception is java.lang.RuntimeException: 예외 발생!] with root cause

java.lang.RuntimeException: 예외 발생!
    ...

2022-03-01 18:10:40.753  INFO 37632 --- [nio-8080-exec-1] hello.exception.Filter.LogFilter         : REQUEST [40667582-2ee1-4d94-aa64-1f6e588e14c9][ERROR][/error-page/500] <- DispatcherType이 Error이다
2022-03-01 18:10:40.758  INFO 37632 --- [nio-8080-exec-1] h.exception.servlet.ErrorPageController  : errorPage 500
2022-03-01 18:10:40.759  INFO 37632 --- [nio-8080-exec-1] h.exception.servlet.ErrorPageController  : ERROR_EXCEPTION : {} 
```

<br><br><br>
## References 및 사진 출처

> __김영한의 스프링 MVC 2편 - 백엔드 웹 개발 활용 기술__