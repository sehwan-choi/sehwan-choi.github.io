---
layout: post
title:  "spring Interceptor"
subtitle:   "spring Interceptor"
date:   2022-02-27 23:07:27 +0900
categories: spring
tags: spring Interceptor
comments: true
---


<br>

- 목차
    - [Spring Interceptor](#spring-interceptor)
        - [스프링 인터셉터 흐름](#스프링-인터셉터-흐름)
        - [스프링 인터셉터 제한](#스프링-인터셉터-제한)
        - [스프링 인터셉터 체인](#스프링-인터셉터-체인)
        - [스프링 인터셉터 인터페이스](#스프링-인터셉터-인터페이스)
    - [스프링 인터셉터 호출 흐름](#스프링-인터셉터-호출-흐름)
        - [정상 흐름](#정상-흐름)
        - [스프링 인터셉터 예외 상황](#스프링-인터셉터-예외-상황)
            - [예외가 발생시](#예외가-발생시)
            - [afterCompletion은 예외가 발생해도 호출](#aftercompletion은-예외가-발생해도-호출)
    - [스프링 인터셉터 - 요청 로그](#스프링-인터셉터---요청-로그)
        - [LogInterceptor - 요청 로그 인터셉터](#loginterceptor---요청-로그-인터셉터)
        - [참고](#참고)
            - [HandlerMethod](#handlermethod)
            - [ResourceHttpRequestHandler](#resourcehttprequesthandler)
            - [postHandle, afterCompletion](#posthandle-aftercompletion)
        - [WebConfig - 인터셉터 등록](#webconfig---인터셉터-등록)
        - [실행 로그](#실행-로그)
        - [스프링의 URL 경로](#스프링의-url-경로)
            - [PathPattern 공식 문서](#pathpattern-공식-문서)
    - [스프링 인터셉터 - 인증 체크](#스프링-인터셉터---인증-체크)
        - [LoginCheckInterceptor - 로그인 인증 인터셉터](#logincheckinterceptor---로그인-인증-인터셉터)
        - [WebConfig - 로그인 체크 인터셉터 등록](#webconfig---로그인-체크-인터셉터-등록)
    - [정리](#정리)
  
<br>

# Spring Interceptor

<br>

스프링 인터셉터도 서블릿 필터와 같이 웹과 관련된 공통 관심 사항을 효과적으로 해결할 수 있는 기술이다. <br>
서블릿 필터가 서블릿이 제공하는 기술이라면, 스프링 인터셉터는 스프링 MVC가 제공하는 기술이다. 둘다 웹과 관련된 공통 관심 사항을 처리하지만, 적용되는 순서와 범위, 그리고 사용방법이 다르다. <br>

<br><br>

## 스프링 인터셉터 흐름

<br>

```
HTTP 요청 -> WAS -> 필터 -> 서블릿 -> 스프링 인터셉터 -> 컨트롤러
```
<br>

- 스프링 인터셉터는 디스패처 서블릿과 컨트롤러 사이에서 컨트롤러 호출 직전에 호출 된다.
- 스프링 인터셉터는 스프링 MVC가 제공하는 기능이기 때문에 결국 디스패처 서블릿 이후에 등장하게 된다. 
- 스프링 MVC의 시작점이 디스패처 서블릿이라고 생각해보면 이해가 될 것이다.
- 스프링 인터셉터에도 URL 패턴을 적용할 수 있는데, 서블릿 URL 패턴과는 다르고, 매우 정밀하게 설정할 수 있다.

<br><br>

## 스프링 인터셉터 제한

<br>

```
HTTP 요청 -> WAS -> 필터 -> 서블릿 -> 스프링 인터셉터 -> 컨트롤러 //로그인 사용자
HTTP 요청 -> WAS -> 필터 -> 서블릿 -> 스프링 인터셉터(적절하지 않은 요청이라 판단, 컨트롤러 호출
X) // 비 로그인 사용자
```

<br>

인터셉터에서 적절하지 않은 요청이라고 판단하면 거기에서 끝을 낼 수도 있다. 그래서 로그인 여부를 체크하기에 딱 좋다.

<br><br>

## 스프링 인터셉터 체인

<br>

```
HTTP 요청 -> WAS -> 필터 -> 서블릿 -> 인터셉터1 -> 인터셉터2 -> 컨트롤러
```

<br>

스프링 인터셉터는 체인으로 구성되는데, 중간에 인터셉터를 자유롭게 추가할 수 있다. 예를 들어서 로그를 남기는 인터셉터를 먼저 적용하고, 그 다음에 로그인 여부를 체크하는 인터셉터를 만들 수 있다. <br>

지금까지 내용을 보면 서블릿 필터와 호출 되는 순서만 다르고, 제공하는 기능은 비슷해 보인다. 앞으로 설명하겠지만, 스프링 인터셉터는 서블릿 필터보다 편리하고, 더 정교하고 다양한 기능을 지원한다. <br>

<br><br>

## 스프링 인터셉터 인터페이스

<br>

스프링의 인터셉터를 사용하려면 HandlerInterceptor 인터페이스를 구현하면 된다.

<br>

```java
public interface HandlerInterceptor {

	default boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler)
			throws Exception {

		return true;
	}

	default void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler,
			@Nullable ModelAndView modelAndView) throws Exception {
	}

	default void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler,
			@Nullable Exception ex) throws Exception {
	}

}
```

<br>

- 서블릿 필터의 경우 단순하게 doFilter() 하나만 제공된다. 인터셉터는 컨트롤러 호출 전( preHandle ), 호출 후( postHandle ), 요청 완료 이후( afterCompletion )와 같이 단계적으로 잘 세분화 되어 있다.

- 서블릿 필터의 경우 단순히 request , response 만 제공했지만, 인터셉터는 어떤 컨트롤러( handler )가 호출되는지 호출 정보도 받을 수 있다. 그리고 어떤 modelAndView 가 반환되는지 응답 정보도 받을 수 있다.

<br><br>

# 스프링 인터셉터 호출 흐름

## 정상 흐름 

<br>

![그림](https://sehwan-choi.github.io/assets/img/spring/MVC2/Interceptor/image1.jpg)

<br>

- preHandle : 컨트롤러 호출 전에 호출된다. (더 정확히는 핸들러 어댑터 호출 전에 호출된다.)
    - preHandle 의 응답값이 true 이면 다음으로 진행하고, false 이면 더는 진행하지 않는다. false 인 경우 나머지 인터셉터는 물론이고, 핸들러 어댑터도 호출되지 않는다. 그림에서 1번에서 끝이 나버린다.

- postHandle : 컨트롤러 호출 후에 호출된다. (더 정확히는 핸들러 어댑터 호출 후에 호출된다.)

- afterCompletion : 뷰가 렌더링 된 이후에 호출된다.

<br><br>

## 스프링 인터셉터 예외 상황

<br>

![그림](https://sehwan-choi.github.io/assets/img/spring/MVC2/Interceptor/image2.jpg)

<br>

### 예외가 발생시 

<br>

- preHandle : 컨트롤러 호출 전에 호출된다.
- postHandle : 컨트롤러에서 예외가 발생하면 postHandle 은 호출되지 않는다.
- afterCompletion : afterCompletion 은 항상 호출된다. 이 경우 예외( ex )를 파라미터로 받아서 어떤 예외가 발생했는지 로그로 출력할 수 있다.

<br>

### afterCompletion은 예외가 발생해도 호출

<br>

- 예외가 발생하면 postHandle() 는 호출되지 않으므로 예외와 무관하게 공통 처리를 하려면 afterCompletion() 을 사용해야 한다.
- 예외가 발생하면 afterCompletion() 에 예외 정보( ex )를 포함해서 호출된다.

<br><br>

# 스프링 인터셉터 - 요청 로그

<br>

## LogInterceptor - 요청 로그 인터셉터

<br>

LogInterceptor
```java
@Slf4j
public class LogInterceptor implements HandlerInterceptor {


    public static final String LOG_ID = "logId";

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {

        String requestURI = request.getRequestURI();
        String uuid = UUID.randomUUID().toString();

        request.setAttribute(LOG_ID,uuid);

        // @RequestMapping : HandlerMethod
        // 정적 리소스 : ResourceHttpRequestHandler

        if ( handler instanceof HandlerMethod) {
            HandlerMethod hm = (HandlerMethod) handler;   //  호출한 컨트롤러 메서드의 모든 정보가 포함되어 있다.
        }

        log.info("request [{}][{}][{}]", uuid, requestURI, handler);
        return true;
    }

    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
        log.info("postHandle [{}]", modelAndView);
    }

    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
        String requestURI = request.getRequestURI();
        String uuid = (String)request.getAttribute(LogInterceptor.LOG_ID);

        log.info("afterCompletion [{}][{}][{}]", uuid, requestURI, handler);

        if (ex != null) {
            log.error("afterCompletion error!", ex);
        }
    }
}
```

- String uuid = UUID.randomUUID().toString()
    - 요청 로그를 구분하기 위한 uuid 를 생성한다.

- request.setAttribute(LOG_ID, uuid)
    - 서블릿 필터의 경우 지역변수로 해결이 가능하지만, 스프링 인터셉터는 호출 시점이 완전히 분리되어 있다. 따라서 preHandle 에서 지정한 값을 postHandle , afterCompletion 에서 함께 사용하려면 어딘가에 담아두어야 한다. LogInterceptor 도 싱글톤 처럼 사용되기 때문에 맴버변수를 사용하면 위험하다. 따라서 request 에 담아두었다. 이 값은 afterCompletion 에서 request.getAttribute(LOG_ID) 로 찾아서 사용한다.

- return true
    - true 면 정상 호출이다. 다음 인터셉터나 컨트롤러가 호출된다

<br>

## 참고

<br>

### HandlerMethod

<br>

핸들러 정보는 어떤 핸들러 매핑을 사용하는가에 따라 달라진다. 스프링을 사용하면 일반적으로 @Controller , @RequestMapping 을 활용한 핸들러 매핑을 사용하는데, 이 경우 핸들러 정보로 HandlerMethod 가 넘어온다.

<br>

### ResourceHttpRequestHandler

<br>

@Controller 가 아니라 /resources/static 와 같은 정적 리소스가 호출 되는 경우 ResourceHttpRequestHandler 가 핸들러 정보로 넘어오기 때문에 타입에 따라서 처리가 필요하다.

<br>

### postHandle, afterCompletion

<br>

종료 로그를 postHandle 이 아니라 afterCompletion 에서 실행한 이유는, 예외가 발생한 경우 postHandle 가 호출되지 않기 때문이다. afterCompletion 은 예외가 발생해도 호출 되는 것을 보장한다.

<br><br>

## WebConfig - 인터셉터 등록

<br>

WebConfigg
```java
@Configuration
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new LogInterceptor())
                .order(1)
                .addPathPatterns("/**") //  Interceptor가 호출될 URI
                .excludePathPatterns("/css/**", "/*.ico", "/error");   //  Interceptor가 호출안될 URI
}
```

<br>

WebMvcConfigurer 가 제공하는 addInterceptors() 를 사용해서 인터셉터를 등록할 수 있다. <br>

- registry.addInterceptor(new LogInterceptor()) : 인터셉터를 등록한다.
- order(1) : 인터셉터의 호출 순서를 지정한다. 낮을 수록 먼저 호출된다.
- addPathPatterns("/\*\*") : 인터셉터를 적용할 URL 패턴을 지정한다. excludePathPatterns("/css/\*\*", "/\*.ico", "/error") : 인터셉터에서 제외할 패턴을 지정한다.
- 필터와 비교해보면 인터셉터는 addPathPatterns , excludePathPatterns 로 매우 정밀하게 URL 패턴을 지정할 수 있다.

<br><br>

## 실행 로그

```xml
REQUEST [6234a913-f24f-461f-a9e1-85f153b3c8b2][/members/add]
[hello.login.web.member.MemberController#addForm(Member)]
postHandle [ModelAndView [view="members/addMemberForm"; model={member=Member(id=null, loginId=null, name=null, password=null), org.springframework.validation.BindingResult.member=org.springframework.validation.BeanPropertyBindingResult: 0 errors}]]
RESPONSE [6234a913-f24f-461f-a9e1-85f153b3c8b2][/members/add]
```

<br><br>

## 스프링의 URL 경로

<br>

스프링이 제공하는 URL 경로는 서블릿 기술이 제공하는 URL 경로와 완전히 다르다. 더욱 자세하고,  세밀하게 설정할 수 있다. <br>
자세한 내용은 다음을 참고하자.

### PathPattern 공식 문서

```
? 한 문자 일치

* 경로(/) 안에서 0개 이상의 문자 일치

** 경로 끝까지 0개 이상의 경로(/) 일치

{spring} 경로(/)와 일치하고 spring이라는 변수로 캡처

{spring:[a-z]+} matches the regexp [a-z]+ as a path variable named "spring"

{spring:[a-z]+} regexp [a-z]+ 와 일치하고, "spring" 경로 변수로 캡처

{*spring} 경로가 끝날 때 까지 0개 이상의 경로(/)와 일치하고 spring이라는 변수로 캡처

/pages/t?st.html — matches /pages/test.html, /pages/tXst.html but not /pages/toast.html

/resources/*.png — matches all .png files in the resources directory

/resources/** — matches all files underneath the /resources/ path, including /

resources/image.png and /resources/css/spring.css

/resources/{*path} — matches all files underneath the /resources/ path and captures their relative path in a variable named "path"; /resources/image.png 

will match with "path" → "/image.png", and /resources/css/spring.css will match with "path" → "/css/spring.css"

/resources/{filename:\\w+}.dat will match /resources/spring.dat and assign the value "spring" to the filename variable
```

### -> [PathPattern 공식 문서 링크](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/util/pattern/PathPattern.html) <-

<br><br>

# 스프링 인터셉터 - 인증 체크

서블릿 필터에서 사용했던 인증 체크 기능을 스프링 인터셉터로 개발해보자. 

<br><br>

## LoginCheckInterceptor - 로그인 인증 인터셉터

<br>

LoginCheckInterceptor
```java
@Slf4j
public class LoginCheckInterceptor implements HandlerInterceptor {
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {

        String requestURI = request.getRequestURI();
        log.info("인증 체크 인터셉터 실행 {}", requestURI);

        HttpSession session = request.getSession();

        if (session == null || session.getAttribute(SessionConst.LOGIN_MEMBER) == null) {
            log.info("미인증 사용자 요청");
            //로그인으로 redirect
            response.sendRedirect("/login?redirectURL=" + requestURI);
            return false;
        }

        return true;
    }
}
```

서블릿 필터와 비교해서 코드가 매우 간결하다. 인증이라는 것은 컨트롤러 호출 전에만 호출되면 된다. 따라서 preHandle 만 구현하면 된다.

<br><br>

## WebConfig - 로그인 체크 인터셉터 등록

<br>

```java
@Configuration
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new LogInterceptor())
                .order(1)
                .addPathPatterns("/**") //  Interceptor가 호출될 URI
                .excludePathPatterns("/css/**", "/*.ico", "/error");   //  Interceptor가 호출안될 URI

        registry.addInterceptor(new LoginCheckInterceptor())
                .order(2)
                .addPathPatterns("/**")
                .excludePathPatterns("/","/members/add","/login","logout", "/css/**", "/*.ico","/error");
    }
}
```

<br>

인터셉터를 적용하거나 하지 않을 부분은 addPathPatterns 와 excludePathPatterns 에 작성하면 된다. <br>
기본적으로 모든 경로에 해당 인터셉터를 적용하되 ( /** ), 홈( / ), 회원가입( /members/add ), 로그인( /login ), 리소스 조회( /css/** ), 오류( /error )와 같은 부분은 로그인 체크 인터셉터를 적용하지 않는다. 서블릿 필터와 비교해보면 매우 편리한 것을 알 수 있다. <br>

<br><br>

# 정리

서블릿 필터와 스프링 인터셉터는 웹과 관련된 공통 관심사를 해결하기 위한 기술이다. <br>
서블릿 필터와 비교해서 스프링 인터셉터가 개발자 입장에서 훨씬 편리하다는 것을 코드로 이해했을 것이다. 특별한 문제가 없다면 인터셉터를 사용하는 것이 좋다.


<br><br><br>
## References 및 사진 출처

> __김영한의 스프링 MVC 2편 - 백엔드 웹 개발 활용 기술__