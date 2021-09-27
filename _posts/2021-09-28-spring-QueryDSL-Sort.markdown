---
layout: post
title:  "spring Querydsl 기본문법 - 정렬"
subtitle:   "spring Querydsl 기본문법 - 정렬"
date:   2021-09-24 03:15:27 +0900
categories: spring
tags: spring JPA ORM Mapping QueryDSL Sort
comments: true
---


<br>

- 목차
	- [정렬](#정렬)
    
<br>

# 정렬

- 예제 <br>

회원 정렬 순서 <br>
1). 회원 나이 내림차순(desc) <br>
2). 회원 이름 올림차순(asc) <br>
 단 2에서 회원 이름이 없으면 마지막에 출력(nulls last) <br><br>

```java
/**
 * 회원 정렬 순서
 * 1. 회원 나이 내림차순(desc)
 * 2. 회원 이름 올림차순(asc)
 * 단 2에서 회원 이름이 없으면 마지막에 출력(nulls last)
 */
@Test
public void sort() {
	em.persist(new Member(null, 100));
	em.persist(new Member("member5", 100));
	em.persist(new Member("member6", 100));

	List<Member> result = queryFactory
			.selectFrom(member)
			.where(member.age.eq(100))
			.orderBy(member.age.desc(), member.username.asc().nullsLast())
			.fetch();

	for (Member member : result) {
		System.out.println("member = " + member);
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
    where
        member1.age = 100 
    order by
        member1.age desc,
        member1.username asc nulls last
		
SQL :
select
	member0_.member_id as member_i1_0_,
	member0_.age as age2_0_,
	member0_.team_id as team_id4_0_,
	member0_.username as username3_0_ 
from
	member member0_ 
where
	member0_.age=100 
order by
	member0_.age desc,
	member0_.username asc nulls last

member = Member(id=8, username=member5, age=100)
member = Member(id=9, username=member6, age=100)
member = Member(id=7, username=null, age=100)
```

위 결과와 같이 order by절에 <br>
1). age 내림차순 <br>
2). username 오름차순 <br>
2-1). username이 null이라면 가장 마지막에 출력한다. <br><br>


> 만일 nullsLast()가 아닌 nullsFirst()를 사용한다면? <br>
username이 null인 데이터가 가장 처음에 출력된다.



<br><br><br>
## References 및 사진 출처

> __김영한의 실전! Querydsl__