---
layout: post
title:  "spring Querydsl Fetch Join"
subtitle:   "spring Querydsl Fetch JOin"
date:   2021-10-04 16:55:27 +0900
categories: spring
tags: spring JPA ORM Mapping QueryDSL FetchJoin
comments: true
---


<br>

- 목차
	- [FetchJoin](#fetchjoin)
	- [FetchJoin 미적용](#fetchjoin-미적용)
	- [FetchJoin 적용](#fetchjoin-적용)
	
<br>

# FetchJoin

페치 조인은 SQL에서 제공하는 기능은 아니다. SQL조인을 활용해서 연관된 엔티티를 SQL 한번에 조회하는 기능이다. 주로 성능 최적화에 사용하는 방법이다.

> 자세한 내용은 아래에서 확인할 수 있다. <br>
FetchJoin : [https://sehwan-choi.github.io/spring/2021/09/12/spring-JPA-Optimazation/](https://sehwan-choi.github.io/spring/2021/09/08/spring-JPA-JPQL8/) <br>
성능 최적화 : [https://sehwan-choi.github.io/spring/2021/09/12/spring-JPA-Optimazation/](https://sehwan-choi.github.io/spring/2021/09/12/spring-JPA-Optimazation/)


<br><br>

- 사용방법

```
Member findMember = queryFactory
                .selectFrom(member)
                .join(member.team,team).fetchJoin() //  member조회시 연관됨 team을 가져오기위해 fetchJoin을 사용한다.
                .where(member.username.eq("member1"))
                .fetchOne();
```

join(), leftJoin() 등 조인 기능 뒤에 fetchJoin() 이라고 추가하면 된다.

<br><br>

# FetchJoin 미적용

지연로딩으로 Member, Team SQL 쿼리 각각 실행된다.

```java
	@PersistenceUnit
    EntityManagerFactory emf;

    @Test
    public void fetchJoin() {
        // 결과를 제대로 보기위해 영속성 컨텍스트를 초기화한다.
        em.flush();
        em.clear();

        Member findMember = queryFactory
                .selectFrom(member)
                .where(member.username.eq("member1"))
                .fetchOne();

        boolean loaded = emf.getPersistenceUnitUtil().isLoaded(findMember.getTeam());
        Assertions.assertThat(loaded).as("페치 조인 미적용").isFalse();
    }
```

- 결과

```sql
JPQL :
select
	member1 
from
	Member member1 
where
	member1.username = ?

SQL :
select
	member0_.member_id as member_i1_0_,
	member0_.age as age2_0_,
	member0_.team_id as team_id4_0_,
	member0_.username as username3_0_ 
from
	member member0_ 
where
	member0_.username=?
```

위 결과 쿼리와 같이 Member 데이터만 가져오고 연관된 Team 데이터는 가져오지 않는다.

<br><br>

# FetchJoin 적용

즉시로딩으로 Member, Team SQL 쿼리 조인으로 한번에 조회한다.

```java
	@PersistenceUnit
    EntityManagerFactory emf;

    @Test
    public void fetchJoin2() {
        // 결과를 제대로 보기위해 영속성 컨텍스트를 초기화한다.
        em.flush();
        em.clear();

        Member findMember = queryFactory
                .selectFrom(member)
                .join(member.team,team).fetchJoin() //  member조회시 연관됨 team을 가져오기위해 fetchJoin을 사용한다.
                .where(member.username.eq("member1"))
                .fetchOne();

        boolean loaded = emf.getPersistenceUnitUtil().isLoaded(findMember.getTeam());
        Assertions.assertThat(loaded).as("페치 조인 적용").isTrue();
    }
```

- 결과

```SQL
JPQL :
select
	member1 
from
	Member member1   
inner join
	fetch member1.team as team 
where
	member1.username = ?1 

SQL :		
select
	member0_.member_id as member_i1_0_0_,
	team1_.team_id as team_id1_1_1_,
	member0_.age as age2_0_0_,
	member0_.team_id as team_id4_0_0_,
	member0_.username as username3_0_0_,
	team1_.name as name2_1_1_ 
from
	member member0_ 
inner join
	team team1_ 
		on member0_.team_id=team1_.team_id 
where
	member0_.username=?
```

위 결과 쿼리와 같이 inner join을 사용하여 Team의 데이터를 가져오는 것을 볼수있다.

<br><br><br>
## References 및 사진 출처

> __김영한의 실전! Querydsl__