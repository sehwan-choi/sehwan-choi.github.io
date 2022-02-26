---
layout: post
title:  "spring HttpSession Login"
subtitle:   "spring HttpSession Login"
date:   2022-02-27 04:00:27 +0900
categories: spring
tags: spring HttpSession Login
comments: true
---


<br>

- 목차
    - [서블릿 HttpSession](#서블릿-httpsession)
    - [HttpSession 소개](#httpsession-소개)
    - [로그인 처리하기 - HttpSession 사용](#로그인-처리하기---httpsession-사용)
        - [세션 생성과 조회 삭제](#세션-생성과-조회-삭제)
        - [세션의 create 옵션에 대해 알아보자](#세션의-create-옵션에-대해-알아보자)
        - [세션에 로그인 회원 정보 보관](#세션에-로그인-회원-정보-보관)
    - [@SessionAttribute](#sessionattribute)
    - [TrackingModes](#trackingmodes)
  
<br>

# 직접 만든 세션을 이용하여 로그인 처리

<br>

이전 게시물에서는 세션을 직접 만들었지만, 이번 게시글에서는 스프링에서 제공하는 HttpSession을 사용하여 로그인 처리를 해볼것이다. <br>

직접만든 세션을 이용한 로그인 처리가 궁금하다면 아래 링크에서 확인해볼수 있다.

### [직접만든 세션을 이용한 로그인](https://sehwan-choi.github.io/spring/2022/02/26/spring-Cookie-login/)

<br><br>


# 서블릿 HttpSession

세션이라는 개념은 대부분의 웹 애플리케이션에 필요한 것이다. 어쩌면 웹이 등장하면서 부터 나온 문제이다. <br>
서블릿은 세션을 위해 HttpSession 이라는 기능을 제공하는데, 지금까지 나온 문제들을 해결해준다.<br>
우리가 직접 구현한 세션의 개념이 이미 구현되어 있고, 더 잘 구현되어 있다.

<br><br>

# HttpSession 소개

서블릿이 제공하는 HttpSession 도 결국 우리가 직접 만든 SessionManager 와 같은 방식으로 동작한다.<br>
서블릿을 통해 HttpSession 을 생성하면 다음과 같은 쿠키를 생성한다. 쿠키 이름이 JSESSIONID 이고, 값은 추정 불가능한 랜덤 값이다.<br>
`Cookie: JSESSIONID=5B78E23B513F50164D6FDD8C97B0AD05`

<br><br>

# 로그인 처리하기 - HttpSession 사용

```java
public interface SessionConst {
    String LOGIN_MEMBER = "loginMember";

}
```

```java
public class LoginController {

    private final LoginService loginService;
    private final SessionManager sessionManager;


    @PostMapping("/login")
    public String loginV3(@Valid @ModelAttribute LoginForm form, BindingResult bindingResult, HttpServletRequest request) {
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

        return "redirect:/";
    }

    @PostMapping("/logout")
    public String logoutV3(HttpServletRequest request) {
        HttpSession session = request.getSession(false);
        if (session != null) {
            session.invalidate();   //  세션을 제거한다
        }
        return "redirect:/";
    }
}
```

<br>

## 세션 생성과 조회 삭제

세션을 생성하려면 request.getSession(true) 를 사용하면 된다. <br>

`public HttpSession getSession(boolean create);` <br><br>

세션을 삭제하려면 session.invalidate()를 사용하면 된다. <br>

<br><br>

## 세션의 create 옵션에 대해 알아보자

- request.getSession(true)

    - 세션이 있으면 기존 세션을 반환한다.
    - 세션이 없으면 새로운 세션을 생성해서 반환한다.

- request.getSession(false)

    - 세션이 있으면 기존 세션을 반환한다.
    - 세션이 없으면 새로운 세션을 생성하지 않는다. null 을 반환한다.

- request.getSession() : 신규 세션을 생성하는 request.getSession(true) 와 동일하다.

<br><br>

## 세션에 로그인 회원 정보 보관

`session.setAttribute(SessionConst.LOGIN_MEMBER, loginMember);` <br>

세션에 데이터를 보관하는 방법은 request.setAttribute(..) 와 비슷하다. 하나의 세션에 여러 값을 저장할 수 있다.

<br><br>

# @SessionAttribute

```java
public class HomeController {

    private final MemberRepository memberRepository;
    private final SessionManager sessionManager;

    @GetMapping("/")
    public String homeLoginSpring(
            @SessionAttribute(name = SessionConst.LOGIN_MEMBER, required = false) Member member, Model model) {

        //세션에 회원 데이터가 없으면 home
        if (member == null) {
            return "home";
        }

        //세션이 유지되면 로그인페이지로 이동
        model.addAttribute("member", member);
        return "loginHome";
    }
}
```

<br>

- 스프링은 세션을 더 편리하게 사용할 수 있도록 @SessionAttribute 을 지원한다.
- 세션을 찾고, 세션에 들어있는 데이터를 찾는 번거로운 과정을 스프링이 한번에 편리하게 처리해주는 것을 확인할 수 있다.
- 이미 로그인 된 사용자를 찾을 때는 다음과 같이 사용하면 된다. 참고로 이 기능은 세션을 생성하지 않는다. <br>
`@SessionAttribute(name = "loginMember", required = false) Member loginMember`

<br><br>

# TrackingModes

<br>

![그림](https://sehwan-choi.github.io/assets/img/spring/MVC2/Login/image8.jpg)

<br>

로그인을 처음 시도하면 URL이 위 사진과 같이 jsessionid 를 포함하고 있는 것을 확인할 수 있다. <br>

이것은 웹 브라우저가 쿠키를 지원하지 않을 때 쿠키 대신 URL을 통해서 세션을 유지하는 방법이다. 이 방법을 사용하려면 URL에 이 값을 계속 포함해서 전달해야 한다. <br>
타임리프 같은 템플릿은 엔진을 통해서 링크를 걸면 jsessionid 를 URL에 자동으로 포함해준다. 서버 입장에서 웹 브라우저가 쿠키를 지원하는지 하지 않는지 최초에는 판단하지 못하므로, 쿠키 값도 전달하고, URL에 jsessionid 도 함께 전달한다. <br>
URL 전달 방식을 끄고 항상 쿠키를 통해서만 세션을 유지하고 싶으면 다음 옵션을 넣어주면 된다. 이렇게 하면 URL에 jsessionid 가 노출되지 않는다.

<br>

application.properties
```properties
server.servlet.session.tracking-modes=cookie
```

<br>

![그림](https://sehwan-choi.github.io/assets/img/spring/MVC2/Login/image9.jpg)


<br><br><br>
## References 및 사진 출처

> __김영한의 스프링 MVC 2편 - 백엔드 웹 개발 활용 기술__