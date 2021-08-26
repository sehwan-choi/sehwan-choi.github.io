---
layout: post
title:  "spring JPA JPQL Join"
subtitle:   "spring JPA JPQL Join"
date:   2021-08-25 21:00:27 +0900
categories: spring
tags: spring JPA ORM Mapping JPQL Join
comments: true
---


<br>

- 목차
	- [조인](#조인)
	    - [조인 - ON 절](#조인---on-절)
	    - [조인 대상 필터링](#조인-대상-필터링)
	    - [연관관계 없는 엔티티 외부 조인](#연관관계-없는-엔티티-외부-조인)
	- [서브쿼리](#서브쿼리)
	    - [서브 쿼리 지원 함수](#서브-쿼리-지원-함수)
	    - [JPA 서브 쿼리 한계](#jpa-서브-쿼리-한계)
    
<br>

# 조인

![그림1](https://sehwan-choi.github.io/assets/img/spring/JPQL/jpa1.jpg)

아래 join 설명시 위 객체모델 사용

<br>

- 내부 조인 :
    - SELECT m FROM Member m [INNER] JOIN m.team t

```java
String Query = "select m from Member m inner join m.team t";
List<Member> resultList = em.createQuery(Query, Member.class)
                    .getResultList();
```

- 외부 조인 :
    - SELECT m FROM Member m LEFT [OUTER] JOIN m.team t 

```java
String Query = "select m from Member m left outer join m.team t";
List<Member> resultList2 = em.createQuery(Query, Member.class)
                    .getResultList();
```

- 세타 조인 : 
    - select count(m) from Member m, Team t where m.username = t.name

```java
String Query = "select m from Member m, Team t where m.username = t.name";
List<Member> resultList3 = em.createQuery(Query, Member.class)
                    .getResultList();
```

<br>

## 조인 - ON 절

- ON절을 활용한 조인(JPA 2.1부터 지원) 

<br>

<br>

## 조인 대상 필터링

• 예) 회원과 팀을 조인하면서, 팀 이름이 team인 팀만 조인 <br>

```java
 String Query2 = "select m from Member m left outer join m.team t where t.name = :teamname";
List<Member> resultList2 = em.createQuery(Query2, Member.class)
        .setParameter("teamname","team")
        .getResultList();

SQL:
SELECT m.* FROM 
Member m LEFT JOIN Team t ON m.TEAM_ID=t.id and t.name='team' 
```

<br>

## 연관관계 없는 엔티티 외부 조인

• 예) 회원의 이름과 팀의 이름이 같은 대상 외부 조인

```java
String Query3 = "select m from Member m, Team t where m.username = t.name";
List<Member> resultList3 = em.createQuery(Query3, Member.class)
            .getResultList();

SQL:
SELECT m.*, t.* FROM 
Member m LEFT JOIN Team t ON m.username = t.name
```

<br>

# 서브쿼리

- 나이가 평균보다 많은 회원

```java
String Query = "select m from Member m where m.age > (select avg(m2.age) from Member m2) ";
List<Member> resultList = em.createQuery(Query, Member.class)
            .getResultList();
```


- 한 건이라도 주문한 고객

```java
String Query = "select m from Member m where (select count(o) from Order o where m = o.member) > 0 ";
List<Member> resultList = em.createQuery(Query, Member.class)
            .getResultList();
```

## 서브 쿼리 지원 함수
- [NOT] EXISTS (subquery): 서브쿼리에 결과가 존재하면 참

```java
String Query = "select m from Member m where exists (select t from m.team t where t.name = ‘팀A') ";
List<Member> resultList = em.createQuery(Query, Member.class)
            .getResultList();
```

- {ALL | ANY | SOME} (subquery) 
    - ALL 모두 만족하면 참
    - ANY, SOME: 같은 의미, 조건을 하나라도 만족하면 참
    
```java
// ALL
String Query = "select o from Order o where o.orderAmount > ALL (select p.stockAmount from Product p)";
List<Member> resultList = em.createQuery(Query, Member.class)
            .getResultList();

// ANY SOME
String Query = "select o from Order o where o.orderAmount > ALL (select p.stockAmount from Product p)";
List<Member> resultList = em.createQuery(Query, Member.class)
            .getResultList();
```

- [NOT] IN (subquery): 서브쿼리의 결과 중 하나라도 같은 것이 있으면 참

<br>

## JPA 서브 쿼리 한계

- JPA는 WHERE, HAVING 절에서만 서브 쿼리 사용 가능
- SELECT 절도 가능(하이버네이트에서 지원) 
- FROM 절의 서브 쿼리는 현재 JPQL에서 불가능
- 조인으로 풀 수 있으면 풀어서 해결

<br><br><br>
## References 및 사진 출처

> __김영한의 자바 ORM 표준 JPA 프로그래밍 - 기본편__