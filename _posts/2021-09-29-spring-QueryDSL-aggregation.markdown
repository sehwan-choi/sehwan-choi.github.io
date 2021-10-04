---
layout: post
title:  "spring Querydsl 집계함수와 group by, having"
subtitle:   "spring Querydsl 집계함수와 group by, having"
date:   2021-09-29 23:00:27 +0900
categories: spring
tags: spring JPA ORM Mapping QueryDSL aggregation groupby having
comments: true
---


<br>

- 목차
	- [집계함수](#집계함수)
	- [GROUP BY](#group-by)
	- [HAVING](#having)
    
<br>

# 집계함수

```java
@Test
public void aggregation() {
	List<Tuple> result = queryFactory
			.select(member.count(),		//	전체 수
					member.age.sum(),	//	멤버의 나이 합
					member.age.avg(),	//	멤버의 나이 평균
					member.age.max(),	//	멤버의 최대 나이
					member.age.min()	//	멤버의 최소 나이
			)
			.from(member)
			.fetch();

	Tuple tuple = result.get(0);
	System.out.println("count = " + tuple.get(member.count()));
	System.out.println("age sum = " + tuple.get(member.age.sum()));
	System.out.println("age avg = " + tuple.get(member.age.avg()));
	System.out.println("age max = " + tuple.get(member.age.max()));
	System.out.println("age min = " + tuple.get(member.age.min()));
}
```

- 결과

```SQL
JPQL :
select
	count(member1),
	sum(member1.age),
	avg(member1.age),
	max(member1.age),
	min(member1.age) 
from
	Member member1

SQL :
select
	count(member0_.member_id) as col_0_0_,
	sum(member0_.age) as col_1_0_,
	avg(cast(member0_.age as double)) as col_2_0_,
	max(member0_.age) as col_3_0_,
	min(member0_.age) as col_4_0_ 
from
	member member0_

count = 4
age sum = 100
age avg = 25.0
age max = 40
age min = 10
```

JPQL이 제공하는 모든 집합 함수를 제공한다.

<br><br>

# GROUP BY

```java
@Test
public void group() {
	List<Tuple> result = queryFactory
			.select(team.name, member.age.avg())
			.from(member)
			.join(member.team, team)
			.groupBy(team.name)
			.fetch();

	Tuple teamA = result.get(0);
	Tuple teamB = result.get(1);

	System.out.println("name = " + teamA.get(team.name));
	System.out.println("    age avg = " + teamA.get(member.age.avg()));

	System.out.println("name = " + teamB.get(team.name));
	System.out.println("    age avg = " + teamB.get(member.age.avg()));
}
```

- 결과

```SQL
JPQL :
select
	team.name,
	avg(member1.age) 
from
	Member member1   
inner join
	member1.team as team 
group by
	team.name

SQL :	
select
	team1_.name as col_0_0_,
	avg(cast(member0_.age as double)) as col_1_0_ 
from
	member member0_ 
inner join
	team team1_ 
		on member0_.team_id=team1_.team_id 
group by
	team1_.name

name = teamA
    age avg = 15.0
name = teamB
    age avg = 35.0
```

<br><br>

# HAVING

```java
@Test
public void group() {
	List<Tuple> result = queryFactory
			.select(member.name, item.price)
			.from(member)
			.join(member.item, item)
			.groupBy(item.price)
			.having(item.price.gt(1000))
			.fetch();
}
```

위와 같이 group by 에서 그룹화된 결과를 제한하려면 having을 사용한다.


<br><br><br>
## References 및 사진 출처

> __김영한의 실전! Querydsl__