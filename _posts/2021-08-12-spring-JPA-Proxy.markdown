---
layout: post
title:  "spring JPA Proxy"
subtitle:   "spring JPA Proxy"
date:   2021-08-18 02:00:27 +0900
categories: spring
tags: spring JPA ORM Mapping Proxy
comments: true
---


<br>

- 목차
	- [Proxy](#proxy)
	- [프록시 특징](#프록시-특징)
	- [프록시 객체의 초기화](#프록시-객체의-초기화)
	- [프록시의 특징](#프록시의-특징)
	- [프록시 확인](#프록시-확인)
<br>

# Proxy

- em.find() vs em.getReference() 
- em.find(): 데이터베이스를 통해서 실제 엔티티 객체 조회
- em.getReference(): 데이터베이스 조회를 미루는 가짜(프록시) 엔티티 객체 조회

![그림1](https://sehwan-choi.github.io/assets/img/spring/JPA-MAPPING4/jpa1.jpg)

<br>

## 프록시 특징

- 실제 클래스를 상속 받아서 만들어짐

![그림1](https://sehwan-choi.github.io/assets/img/spring/JPA-MAPPING4/jpa2.jpg)

- 실제 클래스와 겉 모양이 같다. 
- 사용하는 입장에서는 진짜 객체인지 프록시 객체인지 구분하지 않고 사용하면 됨(이론상)
- 프록시 객체는 실제 객체의 참조(target)를 보관
- 프록시 객체를 호출하면 프록시 객체는 실제 객체의 메소드 호출

![그림1](https://sehwan-choi.github.io/assets/img/spring/JPA-MAPPING4/jpa3.jpg)

<br>

## 프록시 객체의 초기화

```java
Member member = em.getReference(Member.class, “id1”); 
member.getName();
```

![그림1](https://sehwan-choi.github.io/assets/img/spring/JPA-MAPPING4/jpa4.jpg)

getReference로 데이터를 가져온경우 객체 내부에는 아무값이 없는 상태이다.<br>
이때 객체의 데이터를 호출하게 되면 프록시의 초기화가 발생된다.<br>
영속성 컨텍스트를 통해 DB의 데이터를 가져오게 되며 실제 Entity를 생성하게 되고 아무 값이 없던 프록시 객체의 내부에 실제 Entity값이 맵핑된다.

<br>

## 프록시의 특징

- 프록시 객체는 처음 사용할 때 한 번만 초기화
- 프록시 객체를 초기화 할 때, 프록시 객체가 실제 엔티티로 바뀌는 것은 아님, 초
기화되면 프록시 객체를 통해서 실제 엔티티에 접근 가능

```java
{
    ...
    Member member = new Member();
    member.setName("Name");

    em.persist(member);

    em.flush();
    em.clear();

    Member findMember = em.getReference(Member.class, member.getId());
    System.out.println("before findMember = " + findMember.getClass());
    System.out.println("findMember.name = " + findMember.getName());
    System.out.println("after findMember = " + findMember.getClass());
    ...
}

--- result --- 
before findMember = class jpabook.jpashop.domain.Member$HibernateProxy$ChMRPTyA
findMember.name = Name
after findMember = class jpabook.jpashop.domain.Member$HibernateProxy$ChMRPTyA
```

위 코드에서 프록시객체가 초기화 되었다고 해서 실제 Entity로 변경되는 것이 아니다.

<br>

- 프록시 객체는 원본 엔티티를 상속받음, 따라서 타입 체크시 주의해야함 (== 비
교 실패, 대신 instance of 사용) 

```java
{   
    ...
    Member member = new Member();
    member.setName("Name");

    em.persist(member);

    em.flush();
    em.clear();

    Member refMember = em.getReference(Member.class, member.getId());
    System.out.println("refMember = " + refMember.getClass());
    System.out.println("member = " + member.getClass());
    System.out.println("member == refMember : " + (member == refMember));
    ...
}

--- result ---
refMember = class jpabook.jpashop.domain.Member$HibernateProxy$QQZzFeqB
member = class jpabook.jpashop.domain.Member
member == refMember : false
```

```java
{
    ...
    System.out.println("refMember = " + refMember.getClass());
    System.out.println("member = " + member.getClass());
    System.out.println("instanceof : " + (refMember instanceof Member));
    ...
}

--- result ---
refMember = class jpabook.jpashop.domain.Member$HibernateProxy$2HCUTJJp
member = class jpabook.jpashop.domain.Member
instanceof : true
```

위 코드와 같이 타입이 다르기 때문에 == 로 비교를 한다면 false가 나온다. <br> 그렇기 때문에 타입 비교시 instance of 를 사용한다.

<br>

> 여기서 재미있는 사실 하나! <br>
```
    Member refMember = em.getReference(Member.class, member.getId());
    Member findMember = em.find(Member.class, member.getId());
```
위 소스에서 JPA에서는 refMember와 findMember는 == 비교시 항상 true를 반환하는 메커니즘을 가지고 있다. 이전에 확인했던 내용에는 getClass로 클래스 타입 체크를 했을때 refMember는 proxy객체 find는 Member 객체인데 어떻게 == 비교시 true가 반환이 되는 것일까? <br>
JPA에서는 하나의 트랜잭션의 영속성 컨텍스트에서 가져온 데이터라면 가장 최초로 가져온 객체 타입으로 반환을 한다. 아래소스를 보자

- Proxy 객체 조회를 먼저한다면??

```java
{
    ...
    Member refMember = em.getReference(Member.class, member.getId());
    System.out.println("refMember = " + refMember.getClass());
    Member findMember = em.find(Member.class, member.getId());
    System.out.println("findMember = " + member.getClass());
    ...
}

--- result ---
refMember = class jpabook.jpashop.domain.Member$HibernateProxy$VkKhF8bm
findMember = class jpabook.jpashop.domain.Member$HibernateProxy$VkKhF8bm
```

- Find로 Member 객체 조회를 먼저한다면??

```java
{
    ...
    Member findMember = em.find(Member.class, member.getId());
    System.out.println("findMember = " + member.getClass());
    Member refMember = em.getReference(Member.class, member.getId());
    System.out.println("refMember = " + refMember.getClass());
    ...
}

--- result ---
findMember = class jpabook.jpashop.domain.Member
refMember = class jpabook.jpashop.domain.Member
```

위와 같이 __JPA에서는 하나의 트랜잭션의 영속성 컨텍스트에서 가져온 데이터라면 가장 최초로 가져온 데이터 타입으로 반환하기 때문에__ 최초 조회한 객체 타입으로 반환을 한다.

<br>


- 영속성 컨텍스트에 찾는 엔티티가 이미 있으면 em.getReference()를 호출해
도 실제 엔티티 반환
- 영속성 컨텍스트의 도움을 받을 수 없는 준영속 상태일 때, 프록시를 초기화하면
문제 발생
(하이버네이트는 org.hibernate.LazyInitializationException 예외를 터트림)

```java
{
    ...
    Member refMember = em.getReference(Member.class, member.getId());
    em.detach(refMember);   //  준영속 상태로 변경
    //em.clear();           //  준영속 상태로 변경
    System.out.println("refMember name = " + refMember.getName());
    ...
}

--- result ---
org.hibernate.LazyInitializationException: could not initialize proxy [jpabook.jpashop.domain.Member#1] - no Session
	at org.hibernate.proxy.AbstractLazyInitializer.initialize(AbstractLazyInitializer.java:170)
	at org.hibernate.proxy.AbstractLazyInitializer.getImplementation(AbstractLazyInitializer.java:310)
	at org.hibernate.proxy.pojo.bytebuddy.ByteBuddyInterceptor.intercept(ByteBuddyInterceptor.java:45)
	at org.hibernate.proxy.ProxyConfiguration$InterceptorDispatcher.intercept(ProxyConfiguration.java:95)
	at jpabook.jpashop.domain.Member$HibernateProxy$h5R9Fgm3.getName(Unknown Source)
	at jpabook.jpashop.jpaMain.main(jpaMain.java:33)

```

위 코드와 같이 refMember를 사용하기 전에 준영속 상태로 만든다면 Exception이 발생한다.(detach, clear 사용시)

<br>

## 프록시 확인

- 프록시 인스턴스의 초기화 여부 확인 PersistenceUnitUtil.isLoaded(Object entity) 

```java
{
    ...
    Member refMember = em.getReference(Member.class, member.getId());
    System.out.println("before isLoaded : " + em.getEntityManagerFactory().getPersistenceUnitUtil().isLoaded(refMember));   // 초기화 전
    System.out.println("refMember name = " + refMember.getName());  // 초기화 진행
    System.out.println("after isLoaded : " + em.getEntityManagerFactory().getPersistenceUnitUtil().isLoaded(refMember));    // 초기화 후
    ...
}

--- result ---
before isLoaded : false
refMember name = Name
after isLoaded : true
```

<br>

- 프록시 클래스 확인 방법
entity.getClass().getName() 출력(..javasist.. or 
HibernateProxy…) 
- 프록시 강제 초기화
org.hibernate.Hibernate.initialize(entity); 

```java
{
    ...
    Member refMember = em.getReference(Member.class, member.getId());
    Hibernate.initialize(refMember);    // 강제 초기화 진행
    System.out.println("isLoaded : " + em.getEntityManagerFactory().getPersistenceUnitUtil().isLoaded(refMember));
    ...
}

--- result ---
isLoaded : true
```

<br>

- 참고: JPA 표준은 강제 초기화 없음
강제 호출: member.getName()

<br><br><br>
## References 및 사진 출처

> __김영한의 자바 ORM 표준 JPA 프로그래밍 - 기본편__