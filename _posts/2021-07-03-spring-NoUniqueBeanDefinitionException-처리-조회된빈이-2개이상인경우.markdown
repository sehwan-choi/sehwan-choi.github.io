---
layout: post
title:  "NoUniqueBeanDefinitionException처리 조회된빈이 2개이상인경우"
subtitle:   "NoUniqueBeanDefinitionException처리 조회된빈이 2개이상인경우"
date:   2021-07-03 01:54:27 +0900
categories: spring
tags: spring java springboot NoUniqueBeanDefinitionException
comments: true
---

## NoUniqueBeanDefinitionException처리 조회된빈이 2개이상인경우
<br>

- 목차
	- [조회 대상 빈이 2개 이상일 때 해결 방법](#조회-대상-빈이-2개-이상일-때-해결-방법) 
	- [문제상황](#문제상황)
	- [@Autowired 필드명 매칭](#autowired-필드명-매칭) 
	- [@Qualifier 사용](#qualifier-사용) 
	- [@Primary 사용](#primary-사용) 
	- [@우선순위](#우선순위) 

<br>

## 조회 대상 빈이 2개 이상일 때 해결 방법
1. @Autowired 필드 명 매칭
2. @Qualifier @Qualifier끼리 매칭 빈 이름 매칭
3. @Primary 사용

위의 3가지 방법이 있다.
<br><br>

### 문제상황
```java
@Component
public class FixDiscountPolicy implements DiscountPolicy{

    ...
}
```

```java
@Component
public class RateDiscountPolicy implements DiscountPolicy{

    ...
}
```

```java

@Component
@RequiredArgsConstructor
public class OrderServiceImpl implements OrderService{

    private final MemberRepository memberRepository;
    private final DiscountPolicy discountPolicy;

    // Lombok의 @RequiredArgsConstructor를 사용하면 final로 선언된 객체들을 생성자로 만들어 준다. @RequiredArgsConstructor 애노테이션은 아래 코드와 같다.
    //@Autowired
    //public OrderServiceImpl(MemberRepository memberRepository, DiscountPolicy //discountPolicy) {
    //    this.memberRepository = memberRepository;
    //    this.discountPolicy = discountPolicy;
    //}
}
```

위와 같이 DiscountPolicy를 implements하고 있는 FixDiscountPolicy, RateDiscountPolicy가 있다. 물론 두 클래스는 @Component로 스프링빈에 등록되있다.
위와 같은 상황에서, 아래의 테스트코드를 실행시킨다면?

```java
@Configuration
@ComponentScan(
        excludeFilters = @ComponentScan.Filter(type = FilterType.ANNOTATION, classes = Configuration.class)
)
public class AutoAppConfig {

}
```
```java
@Test
    void basicScan(){
        AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(AutoAppConfig.class);
    }
```

```
NoUniqueBeanDefinitionException: No qualifying bean of type 
'hello.core.discount.DiscountPolicy' available: expected single matching bean 
but found 2: fixDiscountPolicy,rateDiscountPolicy
```
위의 에러가 발생할 것이다. 테스트 코드에서 AutoAppConfig클래스에 @ComponentScan을 통해 모든 Component를 스프링 빈에 등록 시킨다. 이 과정에서 OrderServiceImpl클래스에 discountPolicy의존성 주입시 FixDiscountPolicy와 RateDiscountPolicy가 모두 스프링빈으로 등록되어있기 때문에 발생한다.
<br> 위의 에러를 해결하는 방법은 아래와 같다.
<br><br>

### @Autowired 필드명 매칭
위와같은 문제를 해결하는 방법중 Autowired 필드명을 주입하려는 클래스의 필드명으로 변경하면 된다.

```java

@Component
@RequiredArgsConstructor
public class OrderServiceImpl implements OrderService{

    private final MemberRepository memberRepository;
    private final DiscountPolicy rateDiscountPolicy;

    ...
}
```

위 처럼 @Autowired를 사용할때 DiscountPolicy가 FixDiscountPolicy, RateDiscountPolicy 두종류가 있기 때문에 에러를 반환 했지만, 필드명을 discountPolicy에서 rateDiscountPolicy로 변경을 하게 된다면 RateDiscountPolicy가 주입 될것이다.

@Autowired 주입은 아래와 같은 단계를 거친다.
1. 클래스타입으로 가져온다.<br>
```java
AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(AppConfig.class);
DiscountPolicy discountpolicy = ac.getBean(DiscountPolicy.class); // 여기서 스프링빈에 2개가 등록되어 있어 중복 에러 발생
```
2. 만일 클래스타입이 여러개인경우 필드 이름 + 클래스 타입으로 가져온다<br>
```java
AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(AppConfig.class);
DiscountPolicy discountpolicy = ac.getBean("rateDiscountPolicy",DiscountPolicy.class);  //  정확히 RateDiscountPolicy를 스프링빈에서 찾아서 가져온다
```

### @Qualifier 사용

```java
@Component
@Qualifier("mainDiscountPolicy")
public class RateDiscountPolicy implements DiscountPolicy{

    ...
}
```
```java
@Component
public class OrderServiceImpl implements OrderService{

    private final MemberRepository memberRepository;
    private final DiscountPolicy discountPolicy;

    @Autowired
    public OrderServiceImpl(MemberRepository memberRepository, @Qualifier("mainDiscountPolicy") DiscountPolicy discountPolicy) {
        this.memberRepository = memberRepository;
        this.discountPolicy = discountPolicy;
    }
}
```

위와 같이 주입하려는 곳에 @Qualifier를 설정해준다면 스프링빈에 등록된 Qualifier를 찾아 주입을 해줄것이다. @Qualifier는 빈의 이름을 바꾸는게 아니다. 부가적인 정보를 추가하는 것이다. 
@Qualifier 로 주입할 때 @Qualifier("mainDiscountPolicy") 를 못찾으면 어떻게 될까? 그러면 
mainDiscountPolicy라는 이름의 스프링 빈을 추가로 찾는다. 하지만 경험상 @Qualifier 는
@Qualifier 를 찾는 용도로만 사용하는게 명확하고 좋다.
<br><br>

### @Primary 사용
```java
@Component
@Primary
public class RateDiscountPolicy implements DiscountPolicy{

    ...
}
```
```java
@Component
@RequiredArgsConstructor
public class OrderServiceImpl implements OrderService{

    private final MemberRepository memberRepository;
    private final DiscountPolicy discountPolicy;
}
```

@Autowired를 통해 OrderServiceImpl클래스의 discountPolicy 의존성 주입시 여러개의 스프링 빈이 매칭된다면 @Primary가 우선권을 가진다.
<br><br><br>

### 우선순위
@Primary 는 기본값 처럼 동작하는 것이고, @Qualifier 는 매우 상세하게 동작한다. 이런 경우 어떤 것이
우선권을 가져갈까? 스프링은 자동보다는 수동이, 넒은 범위의 선택권 보다는 좁은 범위의 선택권이 우선 순
위가 높다. 따라서 여기서도 @Qualifier 가 우선권이 높다.
<br><br><br>
## References

> __김영한의 스프링 핵심 원리 - 기본편__