---
layout: post
title:  "spring Cookie Login"
subtitle:   "spring Cookie Login"
date:   2022-02-27 01:52:27 +0900
categories: spring
tags: spring Cookie Login
comments: true
---


<br>

- 목차
    - [아직도 쿠키를 모른다면 아래 페이지에서 보고 오자!](#아직도-쿠키를-모른다면-아래-페이지에서-보고-오자)
    - [쿠키의 종류](#쿠키의-종류)
    - [로그인 상태 유지 - 쿠키](#로그인-상태-유지---쿠키)
    - [로그아웃](#로그아웃)
    - [쿠키와 보안 문제](#쿠키와-보안-문제)
    - [대안](#대안)
  
<br>

<br>

# 아직도 쿠키를 모른다면 아래 페이지에서 보고 오자!

## -> [쿠키에 대한 설명 보러가기](https://sehwan-choi.github.io/spring/2021/07/07/spring-HTTP-%EC%BF%A0%ED%82%A4/) <-

<br><br>


# 쿠키의 종류

쿠키에는 영속 쿠키와 세션 쿠키가 있다. <br>
영속 쿠키: 만료 날짜를 입력하면 해당 날짜까지 유지 <br>
세션 쿠키: 만료 날짜를 생략하면 브라우저 종료시 까지만 유지 <br>
브라우저 종료시 로그아웃이 되길 기대하므로, 우리에게 필요한 것은 세션 쿠키이다. <br>

<br><br>

# 로그인 상태 유지 - 쿠키

로그인의 상태를 어떻게 유지할 수 있을까? <br>
서버에서 로그인에 성공하면 HTTP 응답에 쿠키를 담아서 브라우저에 전달하자. <br> 
그러면 브라우저는 앞으로 해당 쿠키를 지속해서 보내준다.

<br>

```java
public class LoginController {

    private final LoginService loginService;

    ...

    @PostMapping("/login")
    public String login(@Valid @ModelAttribute LoginForm form, BindingResult bindingResult, HttpServletResponse response) {
        if (bindingResult.hasErrors()) {
            return "login/loginForm";
        }

        Member loginMember = loginService.login(form.getLoginId(), form.getPassword());

        if (loginMember == null) {
            bindingResult.reject("loginFail","아이디 또는 비밀번호가 일치하지 않습니다.");
            return "login/loginForm";
        }

        //쿠키에 시간 정보를 주지 않으면 세션 쿠키(브라우저 종료시 모두 종료)
        Cookie idCookie = new Cookie("memberId", String.valueOf(loginMember.getId()));
        response.addCookie(idCookie);

        return "redirect:/";
    }

    ...
```


![그림](https://sehwan-choi.github.io/assets/img/spring/MVC2/Login/image1.jpg)

위 코드와 같이 로그인에 성공하면 쿠키를 생성하고 HttpServletResponse 에 담는다. <br>
쿠키 이름은 memberId 이고, 값은 회원의 id 를 담아둔다. 웹 브라우저는 종료 전까지 회원의 id 를 서버에 계속 보내줄 것이다.로그인에 성공하면 쿠키를 생성하고 HttpServletResponse 에 담는다. <br>
쿠키 이름은 memberId 이고, 값은 회원의 id 를 담아둔다. 웹 브라우저는 종료 전까지 회원의 id 를 서버에 계속 보내줄 것이다. <br>
또 한 세션 쿠키로 만들었기 때문에 Expire란에 Session으로 되어있는 것을 볼수 있다.<br>

<br>

```java
public class HomeController {

    private final MemberRepository memberRepository;

    ...

    @GetMapping("/")
    public String homeLogin(@CookieValue(name = "memberId", required = false) Long memberId, Model model) {

        if (memberId == null) {
            return "home";
        }

        Member loginMember = memberRepository.findById(memberId);
        if (loginMember == null) {
            return "home";
        }

        model.addAttribute("member", loginMember);
        return "loginHome";
    }

    ...
}
```

<br>

위의 소스는 메인페이지 접속시 이전에 로그인을 했는지 확인한다.<br>
@CookieValue 를 사용하면 편리하게 쿠키를 조회할 수 있다.<br>
로그인 하지 않은 사용자도 홈에 접근할 수 있기 때문에 required = false 를 사용한다.<br>

<br>

# 로그아웃


```java
public class LoginController {

    private final LoginService loginService;

    ...

    @PostMapping("/logout")
    public String logout(HttpServletResponse response) {
        expireCooke(response,"memberId");
        return "redirect:/";
    }

    private void expireCooke(HttpServletResponse response, String cookieName) {
        Cookie cookie = new Cookie(cookieName, null);
        cookie.setMaxAge(0);    //  해당 쿠키를 즉시 종료
        response.addCookie(cookie);
    }

    ...
}
```

<br><br>

세션 쿠키이므로 웹 브라우저 종료시 쿠키가 삭제 된다. <br>
혹은 서버에서 해당 쿠키의 종료 날짜를 0으로 지정하게 되면 로그아웃시 쿠키가 즉사 삭제된다.<br>

<br><br>

# 쿠키와 보안 문제

![그림](https://sehwan-choi.github.io/assets/img/spring/MVC2/Login/image2.jpg)

위 사진과 같이 개발자 도구에서 쿠키 값은 임의로 변경할 수 있다. <br>

쿠키는 클라이언트가 보내는 것이기 때문에 쿠키를 강제로 변경하면 다른 사용자가 된다. <br>

실제 웹브라우저 개발자모드에서 MemberId를 변경하게 되면 다른 사용자게 보이제 되며 쿠키에 보관된 정보를 훔쳐갈 수 있다.<br><br>

___만약 쿠키에 개인정보나, 신용카드 정보가 있다면?___ <br>

이 정보가 웹 브라우저에도 보관되고, 네트워크 요청마다 계속 라이언트에서 서버로 전달된다.<br>

쿠키의 정보가 나의 로컬 PC가 털릴 수도 있고, 네트워크 전송 구간에서 털릴 수도 있다. <br>

해커가 쿠키를 한번 훔쳐가면 평생 사용할 수 있다.<br>

해커가 쿠키를 훔쳐가서 그 쿠키로 악의적인 요청을 계속 시도할 수 있다.<br><br><br>

# 대안

쿠키에 중요한 값을 노출하지 않고, 사용자 별로 예측 불가능한 임의의 토큰(랜덤 값)을 노출하고, 서버에서 토큰과 사용자 id를 매핑해서 인식한다. <br>

그리고 서버에서 토큰을 관리한다.
토큰은 해커가 임의의 값을 넣어도 찾을 수 없도록 예상 불가능 해야 한다.
해커가 토큰을 털어가도 시간이 지나면 사용할 수 없도록 서버에서 해당 토큰의 만료시간을 짧게(예: 30분) 유지한다. <br>

또는 해킹이 의심되는 경우 서버에서 해당 토큰을 강제로 제거하면 된다.

<br><br>

아래 글을 참조하여 쿠키의 보안 문제를 해결해보자.

## [세션을 이용하여 쿠키의 보안 문제 해결하기](https://sehwan-choi.github.io/spring/2022/02/26/spring-Session-login-copy/)

<br><br><br>
## References 및 사진 출처

> __김영한의 스프링 MVC 2편 - 백엔드 웹 개발 활용 기술__