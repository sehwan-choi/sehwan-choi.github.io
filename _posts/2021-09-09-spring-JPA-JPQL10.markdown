---
layout: post
title:  "spring JPA JQPL 엔티티 직접 사용"
subtitle:   "spring JPA JPQL 엔티티 직접 사용"
date:   2021-09-09 15:16:27 +0900
categories: spring
tags: spring JPA ORM Mapping JPQL 엔티티 직접 사용
comments: true
---


<br>

- 목차
	- [엔티티 직접 사용 - 기본 키 값](#엔티티-직접-사용---기본-키-값)
	- [엔티티 직접 사용 - 외래 키 값](#엔티티-직접-사용---외래-키-값)
    
<br>

# 엔티티 직접 사용 - 기본 키 값

- JPQL에서 엔티티를 직접 사용하면 SQL에서 해당 엔티티의 기본 키 값을 사용

```
[JPQL]
select count(m.id) from Member m //엔티티의 아이디를 사용
select count(m) from Member m //엔티티를 직접 사용 

[SQL](JPQL 둘다 같은 다음 SQL 실행)
select count(m.id) as cnt from Member m
```

- 엔티티를 파라미터로 전달

```java
/**
 * 엔티티 직접 사용 - 기본 키 값
 * 엔티티를 파라미터로 전달
*/
String query = "select m from Member m where m = :member";
Member findMember = em.createQuery(query, Member.class).setParameter("member", member2).getSingleResult();

System.out.println("findMember = " + findMember);
 ```

- 식별자를 직접 전달

```java
/**
 * 엔티티 직접 사용 - 기본 키 값
 * 식별자를 직접 전달
*/
String query = "select m from Member m where m.id = :memberid";
Member findMember = em.createQuery(query, Member.class).setParameter("memberid", member2.getId()).getSingleResult();

System.out.println("findMember = " + findMember);
```

- 결과

```sql
select m.* from Member m where m.id=?
```

위의 두 JQPL 둘다 같은 쿼리문이 실행된다.

<br><br>

# 엔티티 직접 사용 - 외래 키 값

- 엔티티를 파라미터로 전달

```java
/**
 * 엔티티 직접 사용 - 외래 키 값
 * 엔티티를 파라미터로 전달
*/
String query = "select m from Member m where m.team = :team";
List<Member> resultList = em.createQuery(query, Member.class).setParameter("team",team).getResultList();

for (Member member : resultList) {
    System.out.println("member = " + member);
}
```

- 식별자를 직접 전달

```java
/**
 * 엔티티 직접 사용 - 외래 키 값
 * 식별자를 직접 전달
*/
String query = "select m from Member m where m.team.id = :teamid";
List<Member> teamid = em.createQuery(query, Member.class).setParameter("teamid", team.getId()).getResultList();

for (Member member : teamid) {
    System.out.println("member = " + member);
}
```

- 결과

```sql
select m.* from Member m where m.team_id=?
```

위의 두 JQPL 둘다 같은 쿼리문이 실행된다.

<br><br><br>
## References 및 사진 출처

> __김영한의 자바 ORM 표준 JPA 프로그래밍 - 기본편__