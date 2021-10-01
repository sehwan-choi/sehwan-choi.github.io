---
layout: post
title:  "spring Querydsl 기본조인 inner, left, right, theta on"
subtitle:   "spring Querydsl 기본조인 inner, left, right, theta on"
date:   2021-10-01 23:30:27 +0900
categories: spring
tags: spring JPA ORM Mapping QueryDSL innerJoin leftJoin rightJoin thetaJoin on
comments: true
---


<br>

- 목차
	- [조인 - 기본 조인](#조인---기본-조인)
		- [세타 조인](#세타-조인)
	- [조인 - on절](#조인---on절)
		- [조인 대상 필터링](#조인-대상-필터링)
		- [연관관계 없는 엔티티 외부 조인](#연관관계-없는-엔티티-외부-조인)
    
<br>

# 조인 - 기본 조인

조인의 기본 문법은 첫 번째 파라미터에 조인 대상을 지정하고, 두 번째 파라미터에 별칭(alias)으로 사용할 Q 타입을 지정하면 된다.

```java
join(조인 대상, 별칭으로 사용할 Q타입)
leftjoin(조인 대상, 별칭으로 사용할 Q타입)
rightjoin(조인 대상, 별칭으로 사용할 Q타입)
```

<br><br>

```java
/**
* 팀 A에 소속된 모든 회원 찾기
*/
@Test
public void join() {
	List<Member> result = queryFactory
			.selectFrom(member)
			.join(member.team, team)
            //.leftJoin(member.team, team)	//	leftJoin 사용
			//.rightJoin(member.team, team)	//	rightJoin 사용
			.where(team.name.eq("teamA"))
			.fetch();

	for (Member member : result) {
		System.out.println("member = " + member);
		System.out.println("    TeamName = " + member.getTeam().getName());
	}
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
	member1.team as team 
where
	team.name = ?1
		
SQL :
select
	member0_.member_id as member_i1_0_,
	member0_.age as age2_0_,
	member0_.team_id as team_id4_0_,
	member0_.username as username3_0_ 
from
	member member0_ 
inner join
	team team1_ 
		on member0_.team_id=team1_.team_id 
where
	team1_.name=?

member = Member(id=3, username=member1, age=10)
    TeamName = teamA
member = Member(id=4, username=member2, age=10)
    TeamName = teamA
```

위 결과와 같이 join을 사용하는 경우 inner join으로 쿼리가 실행되며 leftJoin, rightJoin 사용 가능하다.

<br><br>

# 세타 조인

연관관계가 없는 필드로 조인

```java
@BeforeEach
public void before() {
	queryFactory = new JPAQueryFactory(em);

	Team teamA = new Team("teamA");
	Team teamB = new Team("teamB");
	em.persist(teamA);
	em.persist(teamB);

	Member member1 = new Member("member1", 10, teamA);
	Member member2 = new Member("member2", 10, teamA);
	Member member3 = new Member("member3", 10, teamB);
	Member member4 = new Member("member4", 10, teamB);

	em.persist(member1);
	em.persist(member2);
	em.persist(member3);
	em.persist(member4);
}

/**
 * 세타 조인(연관관계가 없는 필드로 조인)
 * 회원의 이름이 팀 이름과 같은 회원 조회
 */
@Test
public void thetaJoin() {
	em.persist(new Member("teamA"));
	em.persist(new Member("teamB"));

	List<Member> result = queryFactory
			.select(member)
			.from(member, team)
			.where(member.username.eq(team.name))
			.fetch();

	for (Member member : result) {
		System.out.println("member = " + member.getUsername());
	}
}
```

- 결과

```SQL
JPQL :
select
	member1 
from
	Member member1,
	Team team 
where
	member1.username = team.name
		
SQL :
select
	member0_.member_id as member_i1_0_,
	member0_.age as age2_0_,
	member0_.team_id as team_id4_0_,
	member0_.username as username3_0_ 
from
	member member0_ cross 
join
	team team1_ 
where
	member0_.username=team1_.name
```

위 코드에서 from 절에 여러 엔티티를 선택해서 세타 조인을 할 수 있다.
조건절이 없기때문에 결과는 카티션 곱으로 데이터가 생성된다. 즉 member의 데이터 개수 n, team의 데이터 개수 m 라고 했을때, n x m 개의 데이터가 생성되며 이 데이터 중 where절에 맞는 데이터를 정제하는 방식으로 결과를 가져온다.

<br><br>

# 조인 - on절

ON절을 활용한 조인(JPA 2.1부터 지원)
1. 조인 대상 필터링
2. 연관관계 없는 엔티티 외부 조인

<br>

## 조인 대상 필터링

```java

@BeforeEach
public void before() {
	queryFactory = new JPAQueryFactory(em);

	Team teamA = new Team("teamA");
	Team teamB = new Team("teamB");
	em.persist(teamA);
	em.persist(teamB);

	Member member1 = new Member("member1", 10, teamA);
	Member member2 = new Member("member2", 10, teamA);
	Member member3 = new Member("member3", 10, teamB);
	Member member4 = new Member("member4", 10, teamB);

	em.persist(member1);
	em.persist(member2);
	em.persist(member3);
	em.persist(member4);
}

/**
 * 예) 회원과 팀을 조인하면서, 팀 이름이 teamA인 팀만 조인, 회원은 모두 조회
 */
@Test
public void join_on_filtering() {
	List<Tuple> result = queryFactory
			.select(member, team)
			.from(member)
			.leftJoin(member.team, team)
			.on(team.name.eq("teamA"))
			.fetch();

	for (Tuple tuple : result) {
		System.out.println("tuple = " + tuple);
	}
}
```

- 결과

```SQL
JPQL : 
select
	member1,
	team 
from
	Member member1   
left join
	member1.team as team with team.name = ?1
	
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
left outer join
	team team1_ 
		on member0_.team_id=team1_.team_id 
		and (
			team1_.name= 'teamA'
		)

tuple = [Member(id=3, username=member1, age=10), Team(id=1, name=teamA)]
tuple = [Member(id=4, username=member2, age=10), Team(id=1, name=teamA)]
tuple = [Member(id=5, username=member3, age=10), null]
tuple = [Member(id=6, username=member4, age=10), null]
```

on 절을 활용해 조인 대상을 필터링 할 때, 외부조인이 아니라 내부조인(inner join)을 사용하면, where 절에서 필터링 하는 것과 기능이 동일하다. 따라서 on 절을 활용한 조인 대상 필터링을 사용할 때, 내부조인 이면 익숙한 where 절로 해결하고, 정말 외부조인이 필요한 경우에만 이 기능을 사용하자

<br><br>

## 연관관계 없는 엔티티 외부 조인

```java
/**
	* 연관관계 없는 엔티티 외부 조인
	* 예) 회원의 이름과 팀의 이름이 같은 대상 외부 조인
	*/
@Test
public void thetaJoin_on() {
	em.persist(new Member("teamA"));
	em.persist(new Member("teamB"));

	List<Tuple> result = queryFactory
			.select(member, team)
			.from(member)
			.leftJoin(team).on(member.username.eq(team.name))
			.fetch();

	for (Tuple data : result) {
		System.out.println("member = " + data);
	}
}
```

- 결과

```SQL
JPQL :
select
	member1,
	team 
from
	Member member1   
left join
	Team team with member1.username = team.name

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
left outer join
	team team1_ 
		on (
			member0_.username=team1_.name
		)

member = [Member(id=3, username=member1, age=10), null]
member = [Member(id=4, username=member2, age=10), null]
member = [Member(id=5, username=member3, age=10), null]
member = [Member(id=6, username=member4, age=10), null]
member = [Member(id=7, username=teamA, age=0), Team(id=1, name=teamA)]
member = [Member(id=8, username=teamB, age=0), Team(id=2, name=teamB)]
```

위 코드에서 주의할 점이있다. 문법을 잘 봐야 한다. leftJoin() 부분에 일반 조인과 다르게 엔티티 하나만 들어간다. <br>
일반조인: leftJoin(member.team, team) <br>
위 코드의 theta on조인: leftJoin(__team__) <br>
그리고 일반조인과 theta on조인의 SQL 차이를 보자면.

- 일반조인

```SQL
...
left outer join
	team team1_ 
		on member0_.team_id=team1_.team_id 	// 연관관계있는 테이블 PK, FK를 맵핑한다.
		and (
			team1_.name= 'teamA'
		)
...
```

<br>

- theta on 조인

```SQL
...
left outer join
	team team1_ 
		on (	// PK, FK를 맵핑하지 않는다.
			member0_.username=team1_.name
		)
...
```

PK, FK를 맵핑 유무의 차이가 있다.

<br><br><br>
## References 및 사진 출처

> __김영한의 실전! Querydsl__