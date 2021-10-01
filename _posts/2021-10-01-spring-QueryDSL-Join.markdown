---
layout: post
title:  "spring Querydsl 기본조인 inner, left, right, on"
subtitle:   "spring Querydsl 기본조인 inner, left, right, on"
date:   2021-10-01 23:30:27 +0900
categories: spring
tags: spring JPA ORM Mapping QueryDSL innerJoin leftJoin rightJoin on
comments: true
---


<br>

- 목차
	- [조인 - 기본 조인](#조인---기본-조인)
	- [세타 조인](#세타-조인)
    
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

<br><br><br>
## References 및 사진 출처

> [https://dev-coco.tistory.com/89](https://dev-coco.tistory.com/89)