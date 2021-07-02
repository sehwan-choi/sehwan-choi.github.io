---
layout: post
title:  "스프링 싱글톤 패턴의 문제점"
subtitle:   "스프링 싱글톤 패턴의 문제점"
date:   2021-07-02 01:00:27 +0900
categories: spring
tags: spring java springboot SingletonPatton
comments: true
---

## 스프링 싱글톤 패턴의 문제점
<br>

- 목차
	- [싱글톤 패턴이란?](#싱글톤-패턴이란?) 
	- [싱글톤 패턴 문제점](#싱글톤-패턴-문제점)
	- [싱글톤 컨테이너란?](#싱글톤-컨테이너란?) 
	- [싱글톤 방식의 주의점](#싱글톤-방식의-주의점) 

<br>

### 싱글톤 패턴이란?
클래스의 인스턴스가 1개만 생성되는 것을 보장하는 디자인 패턴이다.
Client의 요청이 있을때마다 객체를 새로 생성하는 것이 아니라, 이미 만들어진 객체를 공유해서 효율적으로 사용할 수 있게 한다.
<br><br>

아래 소스를 보면 전형적인 싱글톤 패턴을 보여주고 있다.
```java
public class SingletonService {

    private static final SingletonService instance = new SingletonService();

    public static SingletonService getInstance(){
        return instance;
    }

    // 외부에서 new 할수 없도록 생성자를 private로 변경
    private SingletonService() {

    }
}
```
<br><br>

### 싱글톤 패턴 문제점
<br>

1. 싱글톤 패턴을 구현하는 코드 자체가 많이 들어간다. <br>
-> 위 소스 처럼 기본적인 코드가 추가 된다.

2. 의존관계상 클라이언트가 구체 클래스에 의존한다. DIP를 위반한다. <br>
-> 사용시 구체클래스.getInstance() 와 같이 사용해야 하므로 구체 클래스에 의존하게 된다.

3. 클라이언트가 구체 클래스에 의존해서 OCP 원칙을 위반할 가능성이 높다.
4. 테스트하기 어렵다.
5. 내부 속성을 변경하거나 초기화 하기 어렵다.
6. private 생성자로 자식 클래스를 만들기 어렵다.
7. 결론적으로 유연성이 떨어진다.
8. 안티패턴으로 불리기도 한다.

<br><br>

### 싱글톤 컨테이너란?
<br>

스프링 컨테이너는 싱글톤 패턴의 문제점을 해결하면서, 객체 인스턴스를 싱글톤(1개만 생성)으로 관리한다.
지금까지 우리가 학습한 스프링 빈이 바로 싱글톤으로 관리되는 빈이다.
- 스프링 컨테이너는 싱글턴 패턴을 적용하지 않아도, 객체 인스턴스를 싱글톤으로 관리한다.
이전에 설명한 컨테이너 생성 과정을 자세히 보자. 컨테이너는 객체를 하나만 생성해서 관리한다.
- 스프링 컨테이너는 싱글톤 컨테이너 역할을 한다. 이렇게 싱글톤 객체를 생성하고 관리하는 기능을 싱글톤 레지스트리라 한다.
- 스프링 컨테이너의 이런 기능 덕분에 싱글턴 패턴의 모든 단점을 해결하면서 객체를 싱글톤으로 유지할 수있다.
- 싱글톤 패턴을 위한 지저분한 코드가 들어가지 않아도 된다.
- DIP, OCP, 테스트, private 생성자로 부터 자유롭게 싱글톤을 사용할 수 있다.

```java
    @Test
    @DisplayName("스프링 컨테이너와 싱글톤")
    void springContainer() {
        ApplicationContext ac = new AnnotationConfigApplicationContext(AppConfig.class);

        MemberService memberService1 = ac.getBean("memberService",MemberService.class);
        MemberService memberService2 = ac.getBean("memberService",MemberService.class);

        System.out.println("memberService1 = " + memberService1);
        System.out.println("memberService2 = " + memberService2);

        Assertions.assertThat(memberService1).isSameAs(memberService2);
    }
```

위의 테스트 코드실행 결과 싱글톤 컨테이너를 사용하기 때문에 getBean으로 memberService호출 하게되면 같은 객체의 MemberService가 반환 될것이다.

```
memberService1 = hello.core.member.MemberServiceImpl@66fdec9
memberService2 = hello.core.member.MemberServiceImpl@66fdec9

Process finished with exit code 0
```

테스트를 실행하면 문제 없이 테스트가 진행되고, memberService1과 memberService2의 주소값이 같은것을 볼 수 있다.

<br><br>

### 싱글톤 방식의 주의점
<br>
싱글톤 패턴이든, 스프링 같은 싱글톤 컨테이너를 사용하든, 객체 인스턴스를 하나만 생성해서 공유하는 싱글톤 방식은 여러 클라이언트가 하나의 같은 객체 인스턴스를 공유하기 때문에 싱글톤 객체는 상태를 유지 (stateful)하게 설계하면 안된다.<br>
**무상태(stateless)로** 설계해야 한다!<br>
특정 클라이언트에 의존적인 필드가 있으면 안된다.
특정 클라이언트가 값을 변경할 수 있는 필드가 있으면 안된다!
가급적 읽기만 가능해야 한다.
필드 대신에 자바에서 공유되지 않는, 지역변수, 파라미터, ThreadLocal 등을 사용해야 한다.
스프링 빈의 필드에 공유 값을 설정하면 정말 큰 장애가 발생할 수 있다!!!

```java
public class StatefulService {

    private int price;  //  상태를 유지하는 필드

    public void order(String name, int price){
        System.out.println("name = " + name + " price = " + price);
        this.price = price; // 문제 발생!
    }

    public int getPrice() {
        return price;
    }
}
```

```java
    @Test
    void statefulServceSingleton(){
        AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(TestConfig.class);

        StatefulService statefulService1 = ac.getBean("statefulService", StatefulService.class);
        StatefulService statefulService2 = ac.getBean("statefulService", StatefulService.class);
        
        //ThreadA : A사용자 10000원 주문
        statefulService1.order("userA",10000);

        //ThreadB : B사용자 20000원 주문
        statefulService2.order("userA",20000);

        //ThreadA : 사용자A 주문 금액 조회
        int price = statefulService1.getPrice();
        System.out.println("price = " + price);
        Assertions.assertThat(statefulService1.getPrice()).isEqualTo(20000);
    }
```
위와 같은 StatefulService와 테스트 케이스가 있다고 하면, 사용자 A가 주문 하고나서 바로 사용자 B가 주문하게 된다면 이후에 사용자 A의 price값은 어떻게 될것인가 고민해야 한다.<br>
(ThreadA, ThreadB는 간단하게 설명하기 위해 실제로 스레드를 생성하지 않았습니다.)
객체를 공유하고 있기때문에 공유되는 필드가 있다면 심각한 버그가 발생할 것이다.

```java

public class StatefulService {
    public int order(String name, int price){
        System.out.println("name = " + name + " price = " + price);
        return price;
    }
}
```

```java
    @Test
    void statefulServceSingleton(){
        AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(TestConfig.class);

        StatefulService statefulService1 = ac.getBean("statefulService", StatefulService.class);
        StatefulService statefulService2 = ac.getBean("statefulService", StatefulService.class);
        
        //ThreadA : A사용자 10000원 주문
        int userAPrice = statefulService1.order("userA", 10000);

        //ThreadB : B사용자 20000원 주문
        int userBPrice = statefulService2.order("userB", 20000);

        //사용자A, B 주문 금액 조회
        System.out.println("priceA = " + userAPrice);
        System.out.println("priceB = " + userBPrice);

        Assertions.assertThat(userAPrice).isEqualTo(10000);
    }
```

위와같이 StatefulService안에 공유되는 필드를 없앨수 있으며, 위와같은 방법을 권장한다.
<br><br><br>
## References

> __김영한의 스프링 핵심 원리 - 기본편__