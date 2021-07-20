---
layout: post
title:  "JPA"
subtitle:   "JPA"
date:   2021-07-20 18:00:27 +0900
categories: spring
tags: spring JPA ORM
comments: true
---

# JPA

<br>

- 목차
	- [SQL 중심적인 개발의 문제점](#sql-중심적인-개발의-문제점)
	- [1. 지루한 코드의 무한 반복](#1-지루한-코드의-무한-반복)
	- [2. 패러다임의 불일치](#2-패러다임의-불일치)
	- [3. 객체와 관계형 데이터베이스의 차이](#3-객체와-관계형-데이터베이스의-차이)
	    - [상속](#상속)
	    - [연관관계](#연관관계)
	    - [비교하기](#비교하기)
	- [JPA](#jpa)
    - [ORM이란?](#orm이란)
    - [JPA를 사용하는 이유](#jpa를-사용하는-이유)
        - [SQL 중심적인 개발에서 객체 중심으로 개발](#sql-중심적인-개발에서-객체-중심으로-개발)
        - [생산성](#생산성)
        - [유지보수](#유지보수)
        - [패러다임의 불일치 해결](#패러다임의-불일치-해결)
        - [JPA의 성능 최적화 기능](#jpa의-성능-최적화-기능)
<br>

<br>

# SQL 중심적인 개발의 문제점

## 1. 지루한 코드의 무한 반복
- CRUD
- INSERT INTO..
- UPDATE .. SET ...
- SELECT ... FROM
- DELETE ...

```java
public class Member {
 private String memberId;
 private String name;
 ...
}
```
```xml
INSERT INTO MEMBER(MEMBER_ID, NAME) VALUES

SELECT MEMBER_ID, NAME FROM MEMBER M

UPDATE MEMBER SET …
```
위와 같이 DB에 Member 테이블이 존재하고 Member 테이블의 CRUD Query가 있다. <br>
만약 여기서 tel이라는 컬럼이 추가된다면 어떻게 될까? <br><br>


```java
public class Member {
 private String memberId;
 private String name;
 private String tel; // 추가
 ...
}
```
```xml
INSERT INTO MEMBER(MEMBER_ID, NAME, TEL) VALUES

SELECT MEMBER_ID, NAME, TEL FROM MEMBER M

UPDATE MEMBER SET … TEL = ?
```

위와 같이 모든 Query에 tel이 추가 될것이다. <br>
혹여나 깜빡하고 쿼리문에서 TEL관련 내용을 추가 하지 못한다면, 버그가 발생할 것이다. <br>
__즉, SQL에 의존적인 개발을 피하기 어렵다.__

<br><br>

## 2. 패러다임의 불일치
객체지향 vs 관계형 데이터베이스<br><br>

- 객체지향
    - ‘객체 지향 프로그래밍은 추상화, 캡슐화, 정보은닉, 상속, 다형성 등 시스템의 복잡성을 제어할 수 있는 다양한 장치들을 제공한다.’ - 어느 객체지향 개발자
    - 필드와 메서드 등을 묶어서 잘 캡슐화해서 사용하는 것이 목표

<br>

- 관계형 데이터베이스 (RDB)
    - 데이터를 잘 정규화해서 보관하는 것이 목표

<br>

패러다임이 다른 두 가지를 가지고 억지로 매핑하기 때문에 여러 가지 문제가 발생한다.<br>
Object를 RDB에 넣으려고 하니까 문제가 발생한다. <br>
하지만, RDB가 인식할 수 있는 것은 SQL뿐이기 때문에 결국, Object를 SQL로 짜야한다.
Object -> [SQL 변환] -> RDB에 저장 <br>
[개발자 == SQL 매퍼] 라고 할만큼 SQL 작업을 너무 많이 하고 있다.
<br><br>

## 3. 객체와 관계형 데이터베이스의 차이

<br><br>

# 상속

![그림1](https://sehwan-choi.github.io/assets/img/spring/JPA-BASIC/jpa1.jpg)

- Object
    - 상속관계가 있다.
- RDB
    - 상속관계와 유사한 물리 모델로, 슈퍼타입 - 서브타입 관계가 존재한다.

<br>

## 객체를 DB에 저장

- Album 객체를 DB에 저장
    1. 객체를 ITEM, ALBUM으로 분해한다.
    2. 각각 다른 테이블에 대한 INSERT 쿼리를 두 번 날린다.
        1. INSERT INTO ITEM ...
        2. INSERT INTO ALBUM ...

<br>

- ALbum 객체를 DB에서 조회
    1. 각각의 테이블에 따른 Join SQL을 작성한다.
    2. 각각 Item과 Album 객체를 생성하고 모든 필드값을 세팅한다.

<br>
위와같은 일련의 과정은 상상만 해도 복잡하다. 그래서 DB에 저장할 객체에는 상속관계를 쓰지 않는다.

<br><br>

## 객체를 컬렉션에 저장

자바 컬렉션에 저장한다면 어떻게 될까?
```java
list.add(album);                    //  저장
Album album = list.get(albumId);    //  조회
Item item = list.get(albumId);      //  다형성 활용
```
위와 같은 코드 한줄로 저장, 조회, 다형성을 모두 활용할 수있다.
자바 컬렉션에 저장하면 굉장히 단순한 작업이 관계형 데이터베이스에 넣고 빼는 순간, Object와 RDB의 매핑 작업을 개발자가 직접 해줘야 하기 때문에 굉장히 복잡한 일이 된다.

<br><br>

# 연관관계

<br>

![그림1](https://sehwan-choi.github.io/assets/img/spring/JPA-BASIC/jpa2.jpg)

<br>

- Object
    - 참조(Reference)를 사용하여 연관 관계를 찾는다.
        - Ex) member.getTeam();
    - 단방향으로만 관계가 존재한다.
    - Member -> Team은 가능하지만, Team -> Member는 불가능하다.
- RDB
    - 외래키(FK)를 사용하여 Join 쿼리를 통해 연관 관계를 찾는다.
        - Ex) JOIN ON M.TEAM_ID = T.TEAM_ID
    - 양방향으로 모두 조회가 가능하다. 즉, 단방향이 존재하지 않는다.
    - Member -> Team 가능: MEMBER.TEAM_ID(FK)와 TEAM.ID(PK)를 Join
    - Team -> Member 가능: TEAM.ID(PK)와 MEMBER.TEAM_ID(FK)를 Join

<br><br>

# 비교하기

<br>

```java
// SQL 사용
String memberId = "100";
Member member1 = memberDAO.getMember(memberId);
Member member2 = memberDAO.getMember(memberId);
member1 == member2; //다르다.
class MemberDAO {
 
 public Member getMember(String memberId) {
 String sql = "SELECT * FROM MEMBER WHERE MEMBER_ID = ?";
 ...
 //JDBC API, SQL 실행
 return new Member(...);
 }
}
```
```java
// 컬렉션 사용
String memberId = "100";
Member member1 = list.get(memberId);
Member member2 = list.get(memberId);
member1 == member2; //같다.
```
SQL를 사용한 경우 DB에서 데이터를 가져오게 되면 new로 새로운 member 객체를 생성하기 때문에 같은 getMember()를 호출하면 새로운 주소값이 할당되게 되어 member1과 member2를 비교하면 다르다.
컬렉션을 사용하여 list.get()을 하게 되면 한번하던 두번하던 동일한 주소값을 반환한다.

<br><br>

# JPA
- 위와같은 문제점으로 인해 JPA가 탄생했다.
- JPA란 Java Persistence API로 자바 진영의 ORM 기술 표준이다.
- JPA는 애플리케이션과 JDBC 사이에서 동작한다.
- JPA는 인터페이스의 모음이며 JPA를 구현한 구현체로는 하이버네이트, EclipseLink, DataNucleus등이 있다.
<br>

![그림1](https://sehwan-choi.github.io/assets/img/spring/JPA-BASIC/jpa3.jpg)

![그림1](https://sehwan-choi.github.io/assets/img/spring/JPA-BASIC/jpa4.jpg)

![그림1](https://sehwan-choi.github.io/assets/img/spring/JPA-BASIC/jpa5.jpg)

![그림1](https://sehwan-choi.github.io/assets/img/spring/JPA-BASIC/jpa6.jpg)

## ORM이란?
ORM이란 객체와 DB의 테이블이 매핑을 이루는 것을 말한다. 즉, 객체가 테이블이 되도록 매핑 시켜주는 것을 말합니다. ORM을 이용하면 SQL Query가 아닌 직관적인 코드(메서드)로서 데이터를 조작할 수 있다.
<br><br>

## JPA를 사용하는 이유

1. SQL 중심적인 개발에서 객체 중심으로 개발
<br>

2. 생산성
    - 저장: jpa.persist(member)
    - 조회: Member member = jpa.find(memberId)
    - 수정: member.setName(“변경할 이름”)
    - 삭제: jpa.remove(member

<br>

3. 유지보수

![그림1](https://sehwan-choi.github.io/assets/img/spring/JPA-BASIC/jpa7.jpg)

![그림1](https://sehwan-choi.github.io/assets/img/spring/JPA-BASIC/jpa8.jpg)

<br>

4. 패러다임의 불일치 해결

- JPA와 상속

![그림1](https://sehwan-choi.github.io/assets/img/spring/JPA-BASIC/jpa9.jpg)

![그림1](https://sehwan-choi.github.io/assets/img/spring/JPA-BASIC/jpa10.jpg)

![그림1](https://sehwan-choi.github.io/assets/img/spring/JPA-BASIC/jpa11.jpg)

<br>

- JPA와 연관관계, 객체 그래프 탐색

![그림1](https://sehwan-choi.github.io/assets/img/spring/JPA-BASIC/jpa12.jpg)

<br>

- 신뢰할 수 있는 엔티티 계층

![그림1](https://sehwan-choi.github.io/assets/img/spring/JPA-BASIC/jpa13.jpg)

<br>

- JPA와 비교하기

![그림1](https://sehwan-choi.github.io/assets/img/spring/JPA-BASIC/jpa14.jpg)

5. JPA의 성능 최적화 기능

- 1차 캐시와 동일성 보장
    - 같은 트랜잭션 안에서는 같은 엔티티를 반환 - 약간의 조회 성능 향상
```java
String memberId = "100";
Member m1 = jpa.find(Member.class, memberId); //SQL
Member m2 = jpa.find(Member.class, memberId); //캐시
println(m1 == m2) //true
//  SQL 1번만 실행
```

<br>

- 트랜잭션을 지원하는 쓰기 지연 - Insert
    - 트랜잭션을 커밋할 때까지 INSERT SQL을 모음
    - JDBC BATCH SQL 기능을 사용해서 한번에 SQL 전송
```java
transaction.begin(); // [트랜잭션] 시작
em.persist(memberA);
em.persist(memberB);
em.persist(memberC);
//여기까지 INSERT SQL을 데이터베이스에 보내지 않는다.
//커밋하는 순간 데이터베이스에 INSERT SQL을 모아서 보낸다.
transaction.commit(); // [트랜잭션] 커밋
```

<br>

- 트랜잭션을 지원하는 쓰기 지연 - Update
    - UPDATE, DELETE로 인한 로우(ROW)락 시간 최소화
    - 트랜잭션 커밋 시 UPDATE, DELETE SQL 실행하고, 바로 커밋
```java
transaction.begin(); // [트랜잭션] 시작
changeMember(memberA); 
deleteMember(memberB); 
비즈니스_로직_수행(); //비즈니스 로직 수행 동안 DB 로우 락이 걸리지 않는다. 
//커밋하는 순간 데이터베이스에 UPDATE, DELETE SQL을 보낸다.
transaction.commit(); // [트랜잭션] 커밋
```

<br>

- 지연 로딩과 즉시 로딩
    - 지연 로딩: 객체가 실제 사용될 때 로딩
    - 즉시 로딩: JOIN SQL로 한번에 연관된 객체까지 미리 조회

![그림1](https://sehwan-choi.github.io/assets/img/spring/JPA-BASIC/jpa15.jpg)

<br><br><br>
## References 및 사진 출처

> __김영한의 자바 ORM 표준 JPA 프로그래밍 - 기본편__
https://gmlwjd9405.github.io/2019/08/03/reason-why-use-jpa.html