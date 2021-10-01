---
layout: post
title:  "spring Querydsl 페이징"
subtitle:   "spring Querydsl 페이징"
date:   2021-09-29 22:00:27 +0900
categories: spring
tags: spring JPA ORM Mapping QueryDSL Pasing
comments: true
---


<br>

- 목차
	- [페이징](#페이징)
	- [조회 건수 제한](#조회-건수-제한)
	- [전체 조회](#전체-조회)
    
<br>

# 페이징

<br><br>

# 조회 건수 제한

```java
@Test
public void paging1() {
	List<Member> result = queryFactory
			.selectFrom(member)
			.orderBy(member.username.desc())	//	정렬
			.offset(1)	//	기준 위치를 정한다.
			.limit(2)	//	가져올 데이터의 개수를 설정한다.
			.fetch();

	for (Member member : result) {
		System.out.println("member = " + member);
		System.out.println("    team = " + member.getTeam().getName());
	}
}
```

위 코드와 같이 orderBy 정렬후, offset으로 기준 위치를 정하고 limit으로 가져올 데이터의 개수를 설정한다.

<br><br>

# 전체 조회

```java
@Test
public void paging2() {
	QueryResults<Member> result = queryFactory
			.selectFrom(member)
			.orderBy(member.username.desc())
			.offset(1)
			.limit(2)
			.fetchResults();

	System.out.println("result.getTotal() = " + result.getTotal());	//	데이터 총 개수
	System.out.println("result.getLimit() = " + result.getLimit());	//	가져올 데이터의 개수
	System.out.println("result.getOffset() = " + result.getOffset());	//	기준 위치
	for (Member member : result.getResults()) {	//	contents 실제 데이터
		System.out.println("member = " + member);
		System.out.println("    team = " + member.getTeam().getName());
	}
}
```

- 결과

```SQL
select
	count(member1) 
from
	Member member1

select
	member0_.member_id as member_i1_0_,
	member0_.age as age2_0_,
	member0_.team_id as team_id4_0_,
	member0_.username as username3_0_ 
from
	member member0_ 
order by
	member0_.username desc limit ? offset ?
```

위와같이 fetchResults는 count()쿼리와 contents 실제 데이터를 가져오는 쿼리 2번이 나가게 된다. count 쿼리는 성능상 주의해서 사용해야한다.

> 참고: 실무에서 페이징 쿼리를 작성할 때, 데이터를 조회하는 쿼리는 여러 테이블을 조인해야 하지만, count 쿼리는 조인이 필요 없는 경우도 있다. 그런데 이렇게 자동화된 count 쿼리는 원본 쿼리와 같이 모두 조인을 해버리기 때문에 성능이 안나올 수 있다. count 쿼리에 조인이 필요없는 성능 최적화가 필요하다면, count 전용 쿼리를 별도로 작성해야 한다.


<br><br><br>
## References 및 사진 출처

> __김영한의 실전! Querydsl__