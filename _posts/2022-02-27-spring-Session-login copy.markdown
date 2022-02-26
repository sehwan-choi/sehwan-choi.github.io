---
layout: post
title:  "spring Session Login"
subtitle:   "spring Session Login"
date:   2022-02-27 03:30:27 +0900
categories: spring
tags: spring Session Login
comments: true
---


<br>

- 목차
    - [쿠키 보안이슈](#쿠키-보안이슈)
    - [세션 동작 방식](#세션-동작-방식)
        - [중요](#중요)
        - [정리](#정리)
    - [로그인 처리하기 - 세션 직접 만들기](#로그인-처리하기---세션-직접-만들기)
        - [Test 코드](#test-코드)
    - [직접만든 세션 적용](#직접만든-세션-적용)
    - [정리](#정리)
  
<br>

# 쿠키 보안이슈

## [쿠키를 이용한 로그인](https://sehwan-choi.github.io/spring/2022/02/26/spring-Cookie-login/)

위 글에서 쿠키를 사용한 로그인에서는 심각한 보안이슈가 발생했다. <br>

이 문제를 해결하려면 결국 중요한 정보를 모두 서버에 저장해야 한다. <br>

그리고 클라이언트와 서버는 추정 불가능한 임의의 식별자 값으로 연결해야 한다. <br>

이렇게 서버에 중요한 정보를 보관하고 연결을 유지하는 방법을 세션이라 한다.

<br><br>

# 세션 동작 방식


![그림](https://sehwan-choi.github.io/assets/img/spring/MVC2/Login/image3.jpg)

<br>

- 사용자가 loginId , password 정보를 전달하면 서버에서 해당 사용자가 맞는지 확인한다.

<br>


![그림](https://sehwan-choi.github.io/assets/img/spring/MVC2/Login/image4.jpg)

<br>

- 세션 ID를 생성하는데, 추정 불가능해야 한다.
- UUID는 추정이 불가능하다.
    - Cookie: mySessionId=zz0101xx-bab9-4b92-9b32-dadb280f4b61
- 생성된 세션 ID와 세션에 보관할 값( memberA )을 서버의 세션 저장소에 보관한다.

<br>

![그림](https://sehwan-choi.github.io/assets/img/spring/MVC2/Login/image5.jpg)

클라이언트와 서버는 결국 쿠키로 연결이 되어야 한다.

- 서버는 클라이언트에 mySessionId 라는 이름으로 세션ID 만 쿠키에 담아서 전달한다.
- 클라이언트는 쿠키 저장소에 mySessionId 쿠키를 보관한다.

<br>

## 중요

- 여기서 중요한 포인트는 회원과 관련된 정보는 전혀 클라이언트에 전달하지 않는다는 것이다.
- 오직 추정 불가능한 세션 ID만 쿠키를 통해 클라이언트에 전달한다.

<br>

![그림](https://sehwan-choi.github.io/assets/img/spring/MVC2/Login/image6.jpg)


- 클라이언트는 요청시 항상 mySessionId 쿠키를 전달한다.
- 서버에서는 클라이언트가 전달한 mySessionId 쿠키 정보로 세션 저장소를 조회해서 로그인시 보관한 세션 정보를 사용한다.

<br>

## 정리
세션을 사용해서 서버에서 중요한 정보를 관리하게 되었다. 덕분에 다음과 같은 보안 문제들을 해결할 수 있다.
- 쿠키 값을 변조 가능, 예상 불가능한 복잡한 세션Id를 사용한다.
- 쿠키에 보관하는 정보는 클라이언트 해킹시 털릴 가능성이 있다. 세션Id가 털려도 여기에는 중요한 정보가 없다.
- 쿠키 탈취 후 사용 해커가 토큰을 털어가도 시간이 지나면 사용할 수 없도록 서버에서 세션의 만료시간을 짧게(예: 30분) 유지한다. 또는 해킹이 의심되는 경우 서버에서 해당 세션을 강제로 제거하면 된다.

<br><br>

# 로그인 처리하기 - 세션 직접 만들기

세션 관리는 크게 다음 3가지 기능을 제공하면 된다. <br>

- 세션 생성
    - sessionId 생성 (임의의 추정 불가능한 랜덤 값)
    - 세션 저장소에 sessionId와 보관할 값 저장
    - sessionId로 응답 쿠키를 생성해서 클라이언트에 전달

    <br>

- 세션 조회

    - 클라이언트가 요청한 sessionId 쿠키의 값으로, 세션 저장소에 보관한 값 조회

<br>

- 세션 만료
    - 클라이언트가 요청한 sessionId 쿠키의 값으로, 세션 저장소에 보관한 sessionId와 값 제거


```java
/**
 * 세션 관리
 */
@Component
public class SessionManager {

    public static final String SESSION_COOKE_NAME = "mySessionId";
    private Map<String, Object> sessionStore = new ConcurrentHashMap<>();

    /**
     * 세션 생성
     * sessionId 생성 (임의의 추정 불가능한 랜덤 값)
     * 세션 저장소에 sessionId와 보관할 값 저장
     * sessionId로 응답 쿠키를 생성해서 클라이언트에 전달
     */

    public void createSession(Object value, HttpServletResponse response) {

        //세션 id를 생성하고, 값을 세션에 저장
        String sessionId = UUID.randomUUID().toString();
        sessionStore.put(sessionId, value);

        //쿠키 생성
        Cookie mySessioCookie = new Cookie(SESSION_COOKE_NAME, sessionId);
        response.addCookie(mySessioCookie);
    }

    /**
     * 세션 조회
     */
    public Object getSession(HttpServletRequest request) {
        Cookie sessionCookie = findCookie(request, SESSION_COOKE_NAME);
        if (sessionCookie == null) {
            return null;
        }

        return sessionStore.get(sessionCookie.getValue());
    }

    /**
     * 세션 만료
     */
    public void expire(HttpServletRequest request) {
        Cookie sessionCookie = findCookie(request, SESSION_COOKE_NAME);
        if (sessionCookie != null){
            sessionStore.remove(sessionCookie.getValue());
        }
    }

    public Cookie findCookie(HttpServletRequest request, String cookieName) {
        Cookie[] cookies = request.getCookies();
        if (cookies == null) {
            return null;
        }

        return Arrays.stream(cookies)
                .filter(m -> m.getName().equals(cookieName))
                .findAny()
                .orElse(null);
    }
}

```

HashMap 은 동시 요청에 안전하지 않기 때문에 동시 요청에 안전한
ConcurrentHashMap 를 사용했다.

<br><br>

## Test 코드

```java
class SessionManagerTest {

    SessionManager sessionManager = new SessionManager();

    @Test
    void sessionTest() {
        // 세션 생성
        Member member = new Member();
        MockHttpServletResponse response = new MockHttpServletResponse();
        sessionManager.createSession(member, response);

        //요청에 응답 쿠키 저장
        MockHttpServletRequest request = new MockHttpServletRequest();
        request.setCookies(response.getCookies());

        //세션 조회
        Object result = sessionManager.getSession(request);
        Assertions.assertThat(result).isEqualTo(member);

        //세션 만료
        sessionManager.expire(request);
        Object expired = sessionManager.getSession(request);
        Assertions.assertThat(expired).isEqualTo(null);
    }
}
```

<br>

# 직접만든 세션 적용

```java
public class LoginController {

    private final LoginService loginService;
    private final SessionManager sessionManager;

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
        sessionManager.createSession(loginMember,response);

        return "redirect:/";
    }

    @PostMapping("/logout")
    public String logout(HttpServletRequest request) {
        sessionManager.expire(request);
        return "redirect:/";
    }
}

```

로그인 성공시 sessionManager를 통해서 새로운 새션을 생성해서 store에 저장시킨다. <br>
로그아웃시에는 sessionManager store에 된 세션을 삭제한다.<br>

<br>

![그림](https://sehwan-choi.github.io/assets/img/spring/MVC2/Login/image7.jpg)

위 사진과 같이 세션의 값이 예측하기 힘든 문자열로 생성된다.

<br><br>

# 정리

세션과 쿠키의 개념을 명확하게 이해하기 위해서 직접 만들어보았다. 
사실 세션이라는 것이 뭔가 특별한 것이 아니라 단지 쿠키를 사용하는데, 서버에서 데이터를 유지하는 방법일 뿐이라는 것을 이해했을 것이다. <br>
그런데 프로젝트마다 이러한 세션 개념을 직접 개발하는 것은 상당히 불편할 것이다. 그래서 서블릿도 세션개념을 지원한다. <br>
이제 직접 만드는 세션 말고, 서블릿이 공식 지원하는 세션을 알아보자. 서블릿이 공식 지원하는 세션은 우리가 직접 만든 세션과 동작 방식이 거의 같다. 추가로 세션을 일정시간 사용하지 않으면 해당 세션을 삭제하는 기능을 제공한다.



<br><br><br>
## References 및 사진 출처

> __김영한의 스프링 MVC 2편 - 백엔드 웹 개발 활용 기술__