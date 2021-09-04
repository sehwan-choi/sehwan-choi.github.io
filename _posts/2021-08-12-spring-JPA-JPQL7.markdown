---
layout: post
title:  "spring JPA JQPL 경로 표현식"
subtitle:   "spring JPA JPQL 경로 표현식"
date:   2021-09-04 14:30:27 +0900
categories: spring
tags: spring JPA ORM Mapping JPQL 경로 표현식
comments: true
---


<br>

- 목차
	- [경로 표현식](#경로-표현식)
	- [경로 표현식 용어 정리](#경로-표현식-용어-정리)
	- [경로 표현식 특징](#경로-표현식-특징)
	- [경로 탐색](#경로-탐색)
	- [명시직 조인, 묵시적 조인](#명시직-조인-묵시적-조인)
	- [경로 표현식 - 예제](#경로-표현식---예제)
	- [경로 탐색을 사용한 묵시적 조인 시 주의사항](#경로-탐색을-사용한-묵시적-조인-시-주의사항)
	- [결론](#결론)
    
<br>

# 경로 표현식

- .(점)을 찍어 객체 그래프를 탐색하는 것

```sql
select m.username -> 상태 필드
    from Member m 
    join m.team t -> 단일 값 연관 필드
    join m.orders o -> 컬렉션 값 연관 필드
where t.name = '팀A' 
```

<br>

# 경로 표현식 용어 정리

- 상태 필드(state field): 단순히 값을 저장하기 위한 필드 (ex: m.username) 
 

<br>

- 연관 필드(association field): 연관관계를 위한 필드
    - 단일 값 연관 필드:
        - @ManyToOne, @OneToOne, 대상이 엔티티(ex: m.team) 
    - 컬렉션 값 연관 필드:
        - @OneToMany, @ManyToMany, 대상이 컬렉션(ex: m.orders)

<br>

- 단일 값 연관 필드 예제

```java
public class JpqlPathExpressionMain {

    public static void main(String[] args) {
        ...
        /**
         * 단일 값 연관 경로
         * 묵시적 내부 조인 발생, 탐색 가능
        */
        String query = "select m.team from Member m";

        List<Team> resultList = em.createQuery(query, Team.class).getResultList();
        for (Team team1 : resultList) {
            System.out.println("team1 = " + team1.getName());
        }
        ...
    }
}
```

- 결과

```
JPQL :
select
    m.team
from
    Member m

SQL :
select
    team1_.TEAM_ID as team_id1_4_,
            team1_.name as name2_4_
from
    Member member0_
inner join      <-- 묵시적 내부 조인 발생!!
    Team team1_
        on member0_.TEAM_ID=team1_.TEAM_ID
```

위 결과와 같이 JPQL상 쿼리에서는 join을 하지 않지만 실제 SQL문에서는 묵시적으로 join문이 나가게 되며, m.team, m.username 과 같이 탐색이 가능하다.

<br>

- 컬렉션 값 연관 필드 예제(묵시적 내부조인, 탐색 불가)

```java
public class JpqlPathExpressionMain {

    public static void main(String[] args) {
        ...
        /**
         * 컬렉션 값 연관 경로
         * 묵시적 내부 조인 발생, 탐색 불가
        */
        String query = "select t.members from Team t";

        List<Collection> resultList = em.createQuery(query, Collection.class).getResultList();
        System.out.println("resultList = " + resultList);
        ...
    }
}
```

- 결과

```
JPQL :
select
    t.members
from
    Team t

SQL :
select
    members1_.MEMBER_ID as member_i1_1_,
    members1_.age as age2_1_,
    members1_.TEAM_ID as team_id5_1_,
    members1_.type as type3_1_,
    members1_.username as username4_1_
from
    Team team0_
inner join  <-- 묵시적 내부 조인 발생!!
    Member members1_
        on team0_.TEAM_ID=members1_.TEAM_ID
```

위 결과와 같이 JPQL상 쿼리에서는 join을 하지 않지만 실제 SQL문에서는 묵시적으로 join문이 나가게 되며, t.members 는 Collection이기 때문에 탐색이 불가능하다

<br>

- 컬렉션 값 연관 필드 예제(명시적 내부조인, 탐색 가능)

```java
public class JpqlPathExpressionMain {

    public static void main(String[] args) {
        ...
        /**
         * 컬렉션 값 연관 경로(명시적 내부 조인 사용)
         * 탐색 가능
        */
        String query = "select m from Team t inner join t.members m"; // 명시적 내부 조인 사용하기 때문에 m.username 과 같이 탐색도 가능하다.

        List<Member> resultList = em.createQuery(query, Member.class).getResultList();
        for (Member member1 : resultList) {
            System.out.println("member1.getUsername() = " + member1.getUsername());
        }
        ...
    }
}
```

- 결과

```
JPQL :
select
    m
from
    Team t
inner join
    t.members m

SQL :
select
    members1_.MEMBER_ID as member_i1_1_,
    members1_.age as age2_1_,
    members1_.TEAM_ID as team_id5_1_,
    members1_.type as type3_1_,
    members1_.username as username4_1_
from
    Team team0_
inner join
    Member members1_
        on team0_.TEAM_ID=members1_.TEAM_ID
```

위 결과와 같이 JPQL상 쿼리에서 명시적으로 join을 하기 때문에 탐색이 가능하다.

<br>

# 경로 표현식 특징

- 상태 필드(state field): 경로 탐색의 끝, 탐색X 
- 단일 값 연관 경로: 묵시적 내부 조인(inner join) 발생, 탐색O 
- 컬렉션 값 연관 경로: 묵시적 내부 조인 발생, 탐색X 
- FROM 절에서 명시적 조인을 통해 별칭을 얻으면 별칭을 통해 탐색 가능

<br>

# 경로 탐색

- 상태 필드 경로 탐색

```
• JPQL: select m.username, m.age from Member m 
• SQL: select m.username, m.age from Member m
```

- 단일 값 연관 경로 탐색

```
• JPQL: select o.member from Order o 
• SQL:
select m.* 
 from Orders o 
 inner join Member m on o.member_id = m.id
```

<br>

# 명시직 조인, 묵시적 조인

- 명시적 조인: join 키워드 직접 사용
    - select m from Member m join m.team t
- 묵시적 조인: 경로 표현식에 의해 묵시적으로 SQL 조인 발생 (내부 조인만 가능) 
    - select m.team from Member m

<br>

# 경로 표현식 - 예제

- select o.member.team from Order o -> 성공
- select t.members from Team -> 성공
- select t.members.username from Team t -> 실패
    - t.members 가 컬렉션형태이기 때문에 탐색이 불가능하다.
- select m.username from Team t join t.members m -> 성공
    - 명시적 내부 조인으로 멤버의 상태필드에 접근 가능하다.

<br>

# 경로 탐색을 사용한 묵시적 조인 시 주의사항

- 항상 내부 조인
- 컬렉션은 경로 탐색의 끝, 명시적 조인을 통해 별칭을 얻어야함
- 경로 탐색은 주로 SELECT, WHERE 절에서 사용하지만 묵시
적 조인으로 인해 SQL의 FROM (JOIN) 절에 영향을 줌

# 결론 

- 가급적 묵시적 조인 대신에 명시적 조인 사용
- 조인은 SQL 튜닝에 중요 포인트
- 묵시적 조인은 조인이 일어나는 상황을 한눈에 파악하기 어려움


<br><br><br>
## References 및 사진 출처

> __김영한의 자바 ORM 표준 JPA 프로그래밍 - 기본편__