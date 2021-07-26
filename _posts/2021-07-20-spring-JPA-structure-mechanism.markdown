---
layout: post
title:  "JPA 내부구조와 동작방식"
subtitle:   "JPA 내부구조와 동작방식"
date:   2021-07-26 22:00:27 +0900
categories: spring
tags: spring JPA structure mechanism
comments: true
---

# JPA 내부구조와 동작방식

<br>

- 목차
	- [JPA 내부구조와 동작방식](#jpa-내부구조와-동작방식)
	- [영속성 컨텍스트](#영속성-컨텍스트)
    	- [엔티티 매니저 팩토리와 엔티티 매니저](#엔티티-매니저-팩토리와-엔티티-매니저)
    	- [엔티티 매니저? 영속성 컨텍스트?](#엔티티-매니저-영속성-컨텍스트)
    	- [엔티티의 생명주기](#엔티티의-생명주기)
    	- [영속성 컨텍스트의 이점](#영속성-컨텍스트의-이점)
    	- [엔티티 삭제](#엔티티-삭제)
    	- [플러시](#플러시)
	- [준영속 상태](#준영속-상태)
    	- [준영속 상태로 만드는 방법](#준영속-상태로-만드는-방법)
<br>

<br>


# 영속성 컨텍스트

- JPA를 이해하는데 가장 중요한 용어
- "엔티티를 영구 저장하는 환경" 이라는 뜻
<br><br>

## 엔티티 매니저 팩토리와 엔티티 매니저

<br>

![그림1](https://sehwan-choi.github.io/assets/img/spring/JPA-BASIC2\jpa1.jpg)

<br><br>

## 엔티티 매니저? 영속성 컨텍스트?

- 영속성 컨텍스트는 논리적인 개념 
- 눈에 보이지 않는다. 
- 엔티티 매니저를 통해서 영속성 컨텍스트에 접근

<br><br>

## 엔티티의 생명주기

- 비영속 (new/transient)

```java 
Member member = new Member(); 
member.setId("member1"); 
member.setUsername("회원1");
```
영속성 컨텍스트와 전혀 관계가 없는 새로운 상태이다.

![그림1](https://sehwan-choi.github.io/assets/img/spring/JPA-BASIC2\jpa3.jpg)

    
<br>

- 영속 (managed)

```java
//객체를 생성한 상태(비영속) 
Member member = new Member(); 
member.setId("member1"); 
member.setUsername(“회원1”);

EntityManager em = emf.createEntityManager();
em.getTransaction().begin();

//객체를 저장한 상태(영속)
em.persist(member);
```
영속성 컨텍스트에 관리되는 상태이다.

![그림1](https://sehwan-choi.github.io/assets/img/spring/JPA-BASIC2\jpa4.jpg)

<br>

- 준영속 (detached)
```java
em.detach(member); 
```
영속성 컨텍스트에 저장되었다가 분리된 상태로 영속성 컨텍스트에서 지워진 상태이다.__(DB에서 데이터가 삭제된 것이 아님!)__

<br>

- 삭제 (removed)
```java
em.remove(member);
```
영속성 컨텍스트에서 삭제된 상태로 DB에서 삭제되었음을 의미한다.

<br><br>

## 영속성 컨텍스트의 이점

- 1차 캐시 
```java
Member member = new Member();
member.setId("member1");
member.setUsername("회원1");
//1차 캐시에 저장됨
em.persist(member);
//1차 캐시에서 조회
Member findMember1 = em.find(Member.class, "member1");
Member findMember2 = em.find(Member.class, "member2");
```
Key값이 findMember1은 데이터는 1차캐시(영속성 컨텍스트)에 이미 저장되어 있기 때문에 DB에서 조회를 하지 않고 바로 데이터를 가져올 수 있다. <br>
하지만 findMember2는 영속성 컨텍스트가 생성된후 저장하거나(persist), 조회(find)를 한적이 없기 때문에 DB에서 조회를 해서 데이터를 가져와야 한다.

![그림1](https://sehwan-choi.github.io/assets/img/spring/JPA-BASIC2\jpa5.jpg)

![그림1](https://sehwan-choi.github.io/assets/img/spring/JPA-BASIC2\jpa6.jpg)

![그림1](https://sehwan-choi.github.io/assets/img/spring/JPA-BASIC2\jpa7.jpg)

<br>

- 동일성(identity) 보장 

```java
Member a = em.find(Member.class, "member1"); 
Member b = em.find(Member.class, "member1");
System.out.println(a == b); //동일성 비교 true
```
Collection에서 데이터를 꺼내는것과 같은 원리기 때문에 같은 key값으로 가져온 객체는 같은 참조값을 갖기때문에 동일성을 보장한다.

<br>

- 트랜잭션을 지원하는 쓰기 지연(transactional write-behind) 
```java
EntityManager em = emf.createEntityManager();
EntityTransaction transaction = em.getTransaction();
//엔티티 매니저는 데이터 변경시 트랜잭션을 시작해야 한다.
transaction.begin(); // [트랜잭션] 시작
em.persist(memberA);
em.persist(memberB);
//여기까지 INSERT SQL을 데이터베이스에 보내지 않는다.
//커밋하는 순간 데이터베이스에 INSERT SQL을 보낸다.
transaction.commit(); // [트랜잭션] 커밋
```
persist 할때마다 쿼리가 나가지 않는다. 쓰기 지연 SQL 저장소에 Query가 저장되며 commit() 호출시에 쓰기 지연 SQL 저장소에 저장되있던 Query가 실행된다.

![그림1](https://sehwan-choi.github.io/assets/img/spring/JPA-BASIC2\jpa8.jpg)

![그림1](https://sehwan-choi.github.io/assets/img/spring/JPA-BASIC2\jpa9.jpg)

<br>

- 변경 감지(Dirty Checking) 
```java
EntityManager em = emf.createEntityManager();
EntityTransaction transaction = em.getTransaction();
transaction.begin(); // [트랜잭션] 시작
// 영속 엔티티 조회
Member memberA = em.find(Member.class, "memberA");
// 영속 엔티티 데이터 수정
memberA.setUsername("hi");
memberA.setAge(10);
//em.update(member) 이런 코드가 있어야 하지 않을까? -> 필요없다
transaction.commit(); // [트랜잭션] 커밋
```
JPA에서는 영속성 컨텍스트가 관리하는 데이터를 수정했을 경우 변경 감지(Dirty Checking) 를 통해서 다시 persist를 하지 않아도 commit하는시점(더 자세히는 flush()가 호출되는 시점)에 자동적으로 쿼리가 실행된다.

![그림1](https://sehwan-choi.github.io/assets/img/spring/JPA-BASIC2\jpa10.jpg)

<br>

- 지연 로딩(Lazy Loading)

<br><br>

## 엔티티 삭제

```java
//삭제 대상 엔티티 조회 
Member memberA = em.find(Member.class, "memberA");
em.remove(memberA); //엔티티 삭제
```

<br><br>

## 플러시
영속성 컨텍스트의 변경내용을 데이터베이스에 반영

- 플러시는 언제 발생 할까?
    - 변경 감지 
    - 수정된 엔티티 쓰기 지연 SQL 저장소에 등록 
    - 쓰기 지연 SQL 저장소의 쿼리를 데이터베이스에 전송 (등록, 수정, 삭제 쿼리)

<br>

- 영속성 컨텍스트를 플러시하는 방법
    - em.flush() - 직접 호출 
    - 트랜잭션 커밋 - 플러시 자동 호출 
    - JPQL 쿼리 실행 - 플러시 자동 호출

<br>

- JPQL 쿼리 실행시 플러시가 자동으로 호출되는 이유
```java
em.persist(memberA);
em.persist(memberB);
em.persist(memberC);
//중간에 JPQL 실행
query = em.createQuery("select m from Member m", Member.class);
List<Member> members= query.getResultList();
```
만일 JPQL에서 플러시가 자동으로 호출되지 않는다면 memberA, memberB, memberC를 영속상태로 만들고 JPQL을 통해 DB에서 모든 Member를 조회할 때 memberA, memberB, memberC는 조회되지 않을것 이다. 왜냐하면 아직 commit이 호출 되지 않았기 때문에 DB에는 없는 데이터기 때문이다.

> __! 참고 - 플러시 모드 옵션 !__
>- FlushModeType.AUTO <br>
    - 커밋이나 쿼리를 실행할 때 플러시 (기본값) 
>- FlushModeType.COMMIT <br>
    - 커밋할 때만 플러시

<br>

- 플러시는!
    - 영속성 컨텍스트를 비우지 않음 
    - 영속성 컨텍스트의 변경내용을 데이터베이스에 동기화 
    - 트랜잭션이라는 작업 단위가 중요 -> 커밋 직전에만 동기화 하면 됨

<br><br>

# 준영속 상태
- 영속 -> 준영속 
- 영속 상태의 엔티티가 영속성 컨텍스트에서 분리(detached) 
- 영속성 컨텍스트가 제공하는 기능을 사용 못함

<br>

## 준영속 상태로 만드는 방법
- em.detach(entity)
    - 특정 엔티티만 준영속 상태로 전환 
- em.clear()
    - 영속성 컨텍스트를 완전히 초기화 
- em.close()
    - 영속성 컨텍스트를 종료

<br><br><br>
## References 및 사진 출처

> __김영한의 자바 ORM 표준 JPA 프로그래밍 - 기본편__
https://gmlwjd9405.github.io/2019/08/03/reason-why-use-jpa.html