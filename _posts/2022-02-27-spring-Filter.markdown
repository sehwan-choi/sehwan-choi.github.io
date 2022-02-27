---
layout: post
title:  "spring Filter"
subtitle:   "spring Filter"
date:   2022-02-27 21:23:27 +0900
categories: spring
tags: spring Filter
comments: true
---


<br>

- 목차
    - [Servlet Filter](#servlet-filter)
    - [Servlet Filter 소개](#servlet-filter-소개)
    - [Servlet Filter - 요청 로그](#servlet-filter---요청-로그)
        - [LogFilter - 로그필터 개발](#logfilter---로그필터-개발)
        - [WebConfig - 필터 설정](#webconfig---필터-설정)
        - [실행 로그](#실행-로그)
    - [Servlet Filter - 인증 체크](#servlet-filter---인증-체크)
        - [LoginCheckFilter - 인증 체크 필터](#logincheckfilter---인증-체크-필터)
        - [WebConfig - loginCheckFilter() 추가](#webconfig---logincheckfilter-추가)
        - [RedirectURL 처리](#redirecturl-처리)
    - [정리](#정리)
  
<br>

# Servlet Filter

<br>

예를 들어 요구사항으로 로그인 한 사용자만 특정 페이지에 들어갈 수 있어야 한다면, 앞에서 로그인을 하지 않은 사용자에게는 특정 페이지 버튼이 보이지 않기 때문에 문제가 없어 보인다. 그런데 문제는 로그인 하지 않은 사용자도   `URL을 직접 호출` 하면 상품 관리 화면에 들어갈 수 있다는 점이다. <br>

예를 들어 상품 관리라는 페이지가 있다고 하고, 상품 관리 컨트롤러에서 로그인 여부를 체크하는 로직을 하나하나 작성하면 되겠지만, 등록, 수정, 삭제, 조회 등등 상품관리의 모든 컨트롤러 로직에 공통으로 로그인 여부를 확인해야 한다. <br> 

더 큰 문제는 향후 로그인과 관련된 로직이 변경될 때 이다. 작성한 모든 로직을 다 수정해야 할 수 있다. <br>

이렇게 애플리케이션 여러 로직에서 공통으로 관심이 있는 있는 것을 공통 관심사(cross-cutting concern)라고 한다. 여기서는 등록, 수정, 삭제, 조회 등등 여러 로직에서 공통으로 인증에 대해서 관심을 가지고 있다.<br>

이러한 공통 관심사는 스프링의 AOP로도 해결할 수 있지만, 웹과 관련된 공통 관심사는 지금부터 설명할 서블릿 필터 또는 스프링 인터셉터를 사용하는 것이 좋다. 웹과 관련된 공통 관심사를 처리할 때는 HTTP의 헤더나 URL의 정보들이 필요한데, 서블릿 필터나 스프링 인터셉터는 HttpServletRequest 를 제공한다. <br>

<br><br>

# Servlet Filter 소개

<br>

필터는 서블릿이 지원하는 수문장이다. 필터의 특성은 다음과 같다.
<br><br>

- 필터 흐름
```
HTTP 요청 -> WAS -> 필터 -> 서블릿 -> 컨트롤러
```

필터를 적용하면 필터가 호출 된 다음에 서블릿이 호출된다. 그래서 모든 고객의 요청 로그를 남기는 요구사항이 있다면 필터를 사용하면 된다. 참고로 필터는 특정 URL 패턴에 적용할 수 있다. /* 이라고 하면 모든 요청에 필터가 적용된다. <br>
참고로 스프링을 사용하는 경우 여기서 말하는 서블릿은 스프링의 디스패처 서블릿으로 생각하면 된다.

<br>

- 필터 제한
```
HTTP 요청 -> WAS -> 필터 -> 서블릿 -> 컨트롤러 //로그인 사용자
HTTP 요청 -> WAS -> 필터(적절하지 않은 요청이라 판단, 서블릿 호출X) //비 로그인 사용자
```

필터에서 적절하지 않은 요청이라고 판단하면 거기에서 끝을 낼 수도 있다. 그래서 로그인 여부를 체크하기에 딱 좋다.

<br>

` 필터 체인
```
HTTP 요청 -> WAS -> 필터1 -> 필터2 -> 필터3 -> 서블릿 -> 컨트롤러
```

필터는 체인으로 구성되는데, 중간에 필터를 자유롭게 추가할 수 있다. 예를 들어서 로그를 남기는 필터를 먼저 적용하고, 그 다음에 로그인 여부를 체크하는 필터를 만들 수 있다.

<br>

- 필터 인터페이스

```java
public interface Filter {
    public default void init(FilterConfig filterConfig) throws ServletException {}

    public void doFilter(ServletRequest request, ServletResponse response,
            FilterChain chain) throws IOException, ServletException;

    public default void destroy() {}
}
```

필터 인터페이스를 구현하고 등록하면 서블릿 컨테이너가 필터를 싱글톤 객체로 생성하고, 관리한다.
- init(): 필터 초기화 메서드, 서블릿 컨테이너가 생성될 때 호출된다.
- doFilter(): 고객의 요청이 올 때 마다 해당 메서드가 호출된다. 필터의 로직을 구현하면 된다.
- destroy(): 필터 종료 메서드, 서블릿 컨테이너가 종료될 때 호출된다.

<br><br>

# Servlet Filter - 요청 로그

필터가 정말 수문장 역할을 잘 하는지 확인하기 위해 가장 단순한 필터인, 모든 요청을 로그로 남기는 필터를 개발하고 적용해보자.

<br>

## LogFilter - 로그필터 개발

<br>

LogFilter
```java
@Slf4j
public class LogFilter implements Filter {
    @Override
    public void init(FilterConfig filterConfig) throws ServletException {
        log.info("log filter 초기화");
    }

    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
        log.info("log doFilter");
        HttpServletRequest httpRequest = (HttpServletRequest) request;
        String requestURI = httpRequest.getRequestURI();

        String uuid = UUID.randomUUID().toString();

        try {
            log.info("Request [{}][{}]",uuid, requestURI);
            chain.doFilter(request,response);
        }catch(Exception e) {
            throw e;
        }finally {
            log.info("Response [{}][{}]", uuid, requestURI);
        }
    }

    @Override
    public void destroy() {
        log.info("log filter destroy");
    }
}

```

- public class LogFilter implements Filter {}
    - 필터를 사용하려면 필터 인터페이스를 구현해야 한다.

- doFilter(ServletRequest request, ServletResponse response, FilterChain chain)
    - HTTP 요청이 오면 doFilter 가 호출된다.
    - ServletRequest request 는 HTTP 요청이 아닌 경우까지 고려해서 만든 인터페이스이다. HTTP를 사용하면 HttpServletRequest httpRequest = (HttpServletRequest) request; 와 같이 다운 케스팅 하면 된다.

- String uuid = UUID.randomUUID().toString();
    - HTTP 요청을 구분하기 위해 요청당 임의의 uuid 를 생성해둔다.

- log.info("REQUEST [{}][{}]", uuid, requestURI);
    - uuid 와 requestURI 를 출력한다.

> 중요!
> - chain.doFilter(request, response);
    - 이 부분이 가장 중요하다. 다음 필터가 있으면 필터를 호출하고, 필터가 없으면 서블릿을 호출한다. 만약 이 로직을 호출하지 않으면 다음 단계로 진행되지 않는다.

<br><br>

## WebConfig - 필터 설정

<br>

WebConfig
```java
@Configuration
public class WebConfig {

    @Bean
    public FilterRegistrationBean logFilter() {
        FilterRegistrationBean<Filter> filterFilterRegistrationBean = new FilterRegistrationBean<>();
        filterFilterRegistrationBean.setFilter(new LogFilter());
        filterFilterRegistrationBean.setOrder(1);
        filterFilterRegistrationBean.addUrlPatterns("/*");

        return filterFilterRegistrationBean;
    }
}

```

필터를 등록하는 방법은 여러가지가 있지만, 스프링 부트를 사용한다면 FilterRegistrationBean 을 사용해서 등록하면 된다. <br>

- setFilter(new LogFilter()) : 등록할 필터를 지정한다.
- setOrder(1) : 필터는 체인으로 동작한다. 따라서 순서가 필요하다. 낮을 수록 먼저 동작한다.
- addUrlPatterns("/*") : 필터를 적용할 URL 패턴을 지정한다. 한번에 여러 패턴을 지정할 수 있다.

> 참고1 <br>
> URL 패턴에 대한 룰은 필터도 서블릿과 동일하다. 자세한 내용은 서블릿 URL 패턴으로 검색해보자.

<br>

> 참고2 <br>
> @ServletComponentScan @WebFilter(filterName = "logFilter", urlPatterns = "/*") 로 필터 등록이 가능하지만 필터 순서 조절이 안된다. 따라서 FilterRegistrationBean 을 사용하자.

<br><br>

## 실행 로그

```xml
hello.login.web.filter.LogFilter: REQUEST [0a2249f2-cc70-4db4-98d1-492ccf5629dd][/items]
hello.login.web.filter.LogFilter: RESPONSE [0a2249f2-cc70-4db4-98d1-492ccf5629dd][/items]
```

필터를 등록할 때 urlPattern 을 /* 로 등록했기 때문에 모든 요청에 해당 필터가 적용된다. <br>

<br>

> 참고 <br>
> 실무에서 HTTP 요청시 같은 요청의 로그에 모두 같은 식별자를 자동으로 남기는 방법은 logback mdc로
검색해보자.

<br><br>

# Servlet Filter - 인증 체크

<br>

사용자가 페이지 접근시 인증을 체크하는 로직을 개발해보자! <br>
로그인 되지 않은 사용자는 특정 페이지 뿐만 아니라 미래에 개발될
페이지에도 접근하지 못하도록 하자! <br>

## LoginCheckFilter - 인증 체크 필터

<br>

LoginCheckFilter
```java
@Slf4j
public class LoginCheckFilter implements Filter {

    private static final String[] whiteList = {"/","/members/add","/login","/logout","/css/*"};

    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
        HttpServletRequest httpRequest = (HttpServletRequest) request;
        String requestURI = httpRequest.getRequestURI();

        HttpServletResponse httpResponse = (HttpServletResponse) response;

        try {
            log.info("인증 체크 필터 시작 {}", requestURI);

            if (isLoginCheckPath(requestURI)) {
                log.info("인증 체크 로직 실행 {}", requestURI);
                HttpSession session = httpRequest.getSession(false);
                if(session == null || session.getAttribute(SessionConst.LOGIN_MEMBER) == null) {
                    log.info("미인증 사용자 요청 {}", requestURI);

                    httpResponse.sendRedirect("/login?redirectURL=" + requestURI);
                    return;
                }
            }

            chain.doFilter(request,response);
        } catch (Exception e) {
            throw e;    //  예외 로깅 가능하지만, 톰캣까지 예외를 보내주어야 함
        } finally {
            log.info("인증 체크 필터 종료 {}", requestURI);
        }

    }

    /**
     * 화이트 리스트의 경우 인증 체크 X
     */
    private boolean isLoginCheckPath(String requestURI) {
        return !PatternMatchUtils.simpleMatch(whiteList, requestURI);
    }

}
```

- whitelist = {"/", "/members/add", "/login", "/logout","/css/*"};
    - 인증 필터를 적용해도 홈, 회원가입, 로그인 화면, css 같은 리소스에는 접근할 수 있어야 한다. 이렇게 화이트 리스트 경로는 인증과 무관하게 항상 허용한다. 화이트 리스트를 제외한 나머지 모든 경로에는 인증 체크 로직을 적용한다.
    
<br>

- isLoginCheckPath(requestURI)
    - 화이트 리스트를 제외한 모든 경우에 인증 체크 로직을 적용한다.

<br>

- httpResponse.sendRedirect("/login?redirectURL=" + requestURI);
    - 미인증 사용자는 로그인 화면으로 리다이렉트 한다. 그런데 로그인 이후에 다시 홈으로 이동해버리면, 원하는 경로를 다시 찾아가야 하는 불편함이 있다. 예를 들어서 상품 관리 화면을 보려고 들어갔다가 로그인 화면으로 이동하면, 로그인 이후에 다시 상품 관리 화면으로 들어가는 것이 좋다. 이런 부분이 개발자 입장에서는 좀 귀찮을 수 있어도 사용자 입장으로 보면 편리한 기능이다. 이러한 기능을 위해 현재 요청한 경로인 requestURI 를 /login 에 쿼리 파라미터로 함께 전달한다. 물론 /login 컨트롤러에서 로그인 성공시 해당 경로로 이동하는 기능은 추가로 개발해야 한다.

<br>

> 중요! <br>
> - return; 여기가 중요하다. 필터를 더는 진행하지 않는다. 이후 필터는 물론 서블릿, 컨트롤러가 더는 호출되지 않는다. 앞서 redirect 를 사용했기 때문에 redirect 가 응답으로 적용되고 요청이 끝난다.

<br><br>

## WebConfig - loginCheckFilter() 추가

WebConfig
```java
@Configuration
public class WebConfig {

    // LogFilter
    @Bean
    public FilterRegistrationBean logFilter() {
        FilterRegistrationBean<Filter> filterFilterRegistrationBean = new FilterRegistrationBean<>();
        filterFilterRegistrationBean.setFilter(new LogFilter());
        filterFilterRegistrationBean.setOrder(1);
        filterFilterRegistrationBean.addUrlPatterns("/*");

        return filterFilterRegistrationBean;
    }

    // LoginCheckFilter
    @Bean
    public FilterRegistrationBean loginCheckFilter() {
        FilterRegistrationBean<Filter> filterFilterRegistrationBean = new FilterRegistrationBean<>();
        filterFilterRegistrationBean.setFilter(new LoginCheckFilter());
        filterFilterRegistrationBean.setOrder(2);
        filterFilterRegistrationBean.addUrlPatterns("/*");

        return filterFilterRegistrationBean;
    }
}
```

- setFilter(new LoginCheckFilter()) : 로그인 필터를 등록한다.
- setOrder(2) : 순서를 2번으로 잡았다. 로그 필터 다음에 로그인 필터가 적용된다.
- addUrlPatterns("/*") : 모든 요청에 로그인 필터를 적용한다.

<br><br>

## RedirectURL 처리

<br>

로그인에 성공하면 처음 요청한 URL로 이동하는 기능을 개발해보자.

<br>

```java
@Controller
@Slf4j
@RequiredArgsConstructor
public class LoginController {

    private final LoginService loginService;
    private final SessionManager sessionManager;

    @PostMapping("/login")
    public String loginV4(@Valid @ModelAttribute LoginForm form, BindingResult bindingResult,
                          @RequestParam(defaultValue = "/") String redirectURL,
                          HttpServletRequest request) {
        if (bindingResult.hasErrors()) {
            return "login/loginForm";
        }

        Member loginMember = loginService.login(form.getLoginId(), form.getPassword());

        if (loginMember == null) {
            bindingResult.reject("loginFail","아이디 또는 비밀번호가 일치하지 않습니다.");
            return "login/loginForm";
        }

        HttpSession session = request.getSession();
        session.setAttribute(SessionConst.LOGIN_MEMBER, loginMember);

        return "redirect:" + redirectURL;
    }
}

```

- 로그인 체크 필터에서, 미인증 사용자는 요청 경로를 포함해서 /login 에 redirectURL 요청 파라미터를 추가해서 요청했다. 이 값을 사용해서 로그인 성공시 해당 경로로 고객을 redirect 한다. 아래의 설명을 참고하자

<br>

- 로그인을해야 접근 가능한 페이지를 직접 URL 호출로 들어가려고 해보자


![그림](https://sehwan-choi.github.io/assets/img/spring/MVC2/Filter/image1.jpg)

<br>

- 아래 사진과 같이 Login 페이지로 redirect 되면서 URL을 보면 login?redirectURL=/items로 변경되있을 것이다.

![그림](https://sehwan-choi.github.io/assets/img/spring/MVC2/Filter/image2.jpg)

<br>

- 로그인을 해보자. 로그인이 성공되었다면 처음에 직접 URL 호출로 들어가려고 했던 페이지가 보일 것이다. 위에서 login?redirectURL=/items 가 나왔던 이유도 로그인 성공시 /items으로 redirect시키라는 의미이다.

![그림](https://sehwan-choi.github.io/assets/img/spring/MVC2/Filter/image3.jpg)


<br><br>

# 정리

서블릿 필터를 잘 사용한 덕분에 로그인 하지 않은 사용자는 나머지 경로에 들어갈 수 없게 되었다. 공통 관심사를 서블릿 필터를 사용해서 해결한 덕분에 향후 로그인 관련 정책이 변경되어도 이 부분만 변경하면 된다.

<br>

> 참고 <br>
> 필터에는 다음에 설명할 스프링 인터셉터는 제공하지 않는, 아주 강력한 기능이 있는데 chain.doFilter(request, response); 를 호출해서 다음 필터 또는 서블릿을 호출할 때 request , response 를 다른 객체로 바꿀 수 있다. ServletRequest , ServletResponse 를 구현한 다른 객체를 만들어서 넘기면 해당 객체가 다음 필터 또는 서블릿에서 사용된다. 잘 사용하는 기능은 아니니 참고만 해두자.

<br><br><br>
## References 및 사진 출처

> __김영한의 스프링 MVC 2편 - 백엔드 웹 개발 활용 기술__