---
layout: post
title:  "spring ArgumentResolver 활용"
subtitle:   "spring ArgumentResolver 활용"
date:   2022-02-27 23:48:27 +0900
categories: spring
tags: spring ArgumentResolver
comments: true
---


<br>

- 목차
    - [ArgumentResolver 활용](#argumentresolver-활용)
    - [@Login Annotation 생성](#login-annotation-생성)
    - [LoginMemberArgumentResolver 생성](#loginmemberargumentresolver-생성)
    - [WebMvcConfigurer에 설정 추가](#webmvcconfigurer에-설정-추가)
    - [실행](#실행)
  

<br>

아래 글에서 @SessionAttribute로 세션에서 로그인 정보를 가져와서 Member변수에 넣는 것을 확인할 수 있다. <br>

이것을 ArgumentResolver 통해서 좀더 간결하고 쉽게 변경해보자! <br>

## -> [Spring Interceptor](https://sehwan-choi.github.io/spring/2022/02/27/spring-Interceptor/) <-

<br><br>

# ArgumentResolver 활용

<br>

```java

@Slf4j
@Controller
@RequiredArgsConstructor
public class HomeController {

    private final MemberRepository memberRepository;
    private final SessionManager sessionManager;

    /**
     * @SessionAttribute 로 개발한 코드
    */
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


    /*
     * 우리가 이제 ArgumentResolver를 통해 @Login 어노테이션을 개발할 코드
     */
    @GetMapping("/")
    public String homeLoginArgumentResolver(@Login Member member, Model model) {

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

위 코드와 같이 이전에 @SessionAttribute로 개발된 homeLoginSpring 메서드를 ArgumentResolver @Login을 사용하도록 homeLoginArgumentResolver메서드로 변경해보자.

<br><br>

# @Login Annotation 생성

<br>

Login
```java
@Target(ElementType.PARAMETER)
@Retention(RetentionPolicy.RUNTIME)
public @interface Login {
}
```

- @Target(ElementType.PARAMETER) : 파라미터에만 사용
- @Retention(RetentionPolicy.RUNTIME) : 리플렉션 등을 활용할 수 있도록 런타임까지 애노테이션 정보가 남아있음

<br><br>

# LoginMemberArgumentResolver 생성

<br>

LoginMemberArgumentResolver
```java
@Slf4j
public class LoginMemberArgumentResolver implements HandlerMethodArgumentResolver {
    @Override
    public boolean supportsParameter(MethodParameter parameter) {
        log.info("supportsParameter 실행");

        boolean hasParameterAnnotation = parameter.hasParameterAnnotation(Login.class);
        boolean hasMemberType = Member.class.isAssignableFrom(parameter.getParameterType());

        return hasParameterAnnotation && hasMemberType;
    }

    @Override
    public Object resolveArgument(MethodParameter parameter, ModelAndViewContainer mavContainer, NativeWebRequest webRequest, WebDataBinderFactory binderFactory) throws Exception {

        log.info("resolveArgument 실행");

        HttpServletRequest request = (HttpServletRequest)webRequest.getNativeRequest();
        HttpSession session = request.getSession(false);

        if(session == null) {
            return null;
        }

        return session.getAttribute(SessionConst.LOGIN_MEMBER);
    }
}
```

- supportsParameter() : @Login 애노테이션이 있으면서 Member 타입이면 해당 ArgumentResolver 가 사용된다.

- resolveArgument() : 컨트롤러 호출 직전에 호출 되어서 필요한 파라미터 정보를 생성해준다. 여기서는 세션에 있는 로그인 회원 정보인 member 객체를 찾아서 반환해준다. 이후 스프링MVC는 컨트롤러의 메서드를 호출하면서 여기에서 반환된 member 객체를 파라미터에 전달해준다.

<br><br>

# WebMvcConfigurer에 설정 추가

<br>

```java
@Configuration
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void addArgumentResolvers(List<HandlerMethodArgumentResolver> resolvers) {
        resolvers.add(new LoginMemberArgumentResolver());
    }
}
```

addArgumentResolvers를 Override 받아서 resolvers에 등록하자.

<br><br>

# 실행

<br>

실행해보면,  @SessionAttribute를 사용한 것과 결과는 동일하지만, 더 편리하게 로그인 회원 정보를 조회할 수 있다. 이렇게 ArgumentResolver 를 활용하면 공통 작업이 필요할 때 컨트롤러를 더욱 편리하게 사용할 수 있다.


<br><br><br>
## References 및 사진 출처

> __김영한의 스프링 MVC 2편 - 백엔드 웹 개발 활용 기술__