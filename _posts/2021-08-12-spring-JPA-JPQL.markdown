---
layout: post
title:  "spring JPA JPQL 기본문법과 API"
subtitle:   "spring JPA JPQL 기본문법과 API"
date:   2021-08-25 17:22:27 +0900
categories: spring
tags: spring JPA ORM Mapping JPQL
comments: true
---


<br>

- 목차
	- [JPQL(Java Persistence Query Language)](#jpqljava-persistence-query-language)
	- [JPQL 문법](#jpql-문법)
	- [집합과 정렬](#집합과-정렬)
	- [TypeQuery, Query](#typequery-query)
	- [결과 조회 API](#결과-조회-api)
	- [파라미터 바인딩](#파라미터-바인딩)
    
<br>

# JPQL(Java Persistence Query Language)

• JPQL은 객체지향 쿼리 언어다.따라서 테이블을 대상으로 쿼리하는 것이 아니라 엔티티 객체를 대상으로 쿼리한다. 
• JPQL은 SQL을 추상화해서 특정데이터베이스 SQL에 의존하지 않는다. 
• JPQL은 결국 SQL로 변환된다.

<br>

# JPQL 문법

- select m from Member as m where m.age > 18 
- 엔티티와 속성은 대소문자 구분O (Member, age) 
- JPQL 키워드는 대소문자 구분X (SELECT, FROM, where) 
- 엔티티 이름 사용, 테이블 이름이 아님(Member) 
- 별칭은 필수(m) (as는 생략가능)

- select_문 :: = 
    - select_절
    - from_절
    - [where_절] 
    - [groupby_절] 
    - [having_절] 
    - [orderby_절] 
- update_문 :: = 
    - update_절 [where_절] 
- delete_문 :: = 
    - delete_절 [where_절]

<br>

# 집합과 정렬

- COUNT, SUM, AVG, MAX, MIN, HAVING, GROUP BY, ORDER BY 등 그대로 사용할 수 있다.

```
select
 COUNT(m), //회원수
 SUM(m.age), //나이 합
 AVG(m.age), //평균 나이
 MAX(m.age), //최대 나이
 MIN(m.age) //최소 나이
from Member m
```

<br>

# TypeQuery, Query

- TypeQuery: 반환 타입이 명확할 때 사용
- Query: 반환 타입이 명확하지 않을 때 사용

```java
TypedQuery<Member> query = 
 em.createQuery("SELECT m FROM Member m", Member.class); 

Query query = 
 em.createQuery("SELECT m.username, m.age from Member m"); 
```

<br>

# 결과 조회 API

- query.getResultList(): 결과가 하나 이상일 때, 리스트 반환
    - 결과가 없으면 빈 리스트 반환
- query.getSingleResult(): 결과가 정확히 하나, 단일 객체 반환
    - 결과가 없으면: javax.persistence.NoResultException 
    - 둘 이상이면: javax.persistence.NonUniqueResultException

```java
//getResultList()
List<Member> resultList = em.createQuery("select m from Member m where m.age >:age", Member.class).setParameter("age", 10).getResultList();

//getSingleResult()
Member member = em.createQuery("select m from Member m where m.age >:age", Member.class).setParameter("age", 10).getSingleResult();                    
```

<br>

# 파라미터 바인딩

```java
// 파라미터 바인딩 - 이름 기준
em.createQuery("select m from Member m where m.age >:age", Member.class)
                    .setParameter("age", 10)

```java
// 파라미터 바인딩 - 위치 기준
em.createQuery("select m from Member m where m.age >?1", Member.class)
                    .setParameter(1, 10)
```

<br><br><br>
## References 및 사진 출처

> __김영한의 자바 ORM 표준 JPA 프로그래밍 - 기본편__