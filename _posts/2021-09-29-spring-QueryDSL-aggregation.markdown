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