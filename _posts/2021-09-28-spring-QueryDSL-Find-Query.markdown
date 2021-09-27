---
layout: post
title:  "spring Querydsl 기본문법 - 검색 조건 쿼리"
subtitle:   "spring Querydsl 검색 조건 쿼리"
date:   2021-09-28 02:22:27 +0900
categories: spring
tags: spring JPA ORM Mapping QueryDSL
comments: true
---


<br>

- 목차
	- [기본 검색 쿼리](#기본-검색-쿼리)
	- [검색 조건](#검색-조건)
	- [AND 조건을 파라미터로 처리](#and-조건을-파라미터로-처리)
    
<br>

# 기본 검색 쿼리

```java
@Test
public void search() {
	Member findMember = queryFactory
			//.select(member)
			//.from(member)	//	select, from을 아래의 selectFrom으로 합칠 수 있다.
			.selectFrom(member)
			.where(
					member.username.eq("member1")
					.and(member.age.eq(10))	//	.and() , .or() 를 메서드 체인으로 연결할 수 있다.
			)
			.fetchOne();

	Assertions.assertThat(findMember.getUsername()).isEqualTo("member1");
	Assertions.assertThat(findMember.getAge()).isEqualTo(10);
}
```

- 결과

```
JPQL :
select
        member1 
    from
        Member member1 
    where
        member1.username = 'member1'
        and member1.age = 10
		
SQL :
select
	member0_.member_id as member_i1_0_,
	member0_.age as age2_0_,
	member0_.team_id as team_id4_0_,
	member0_.username as username3_0_ 
from
	member member0_ 
where
	member0_.username= 'member1'
	and member0_.age=10
```

검색 조건은 .and() , . or() 를 메서드 체인으로 연결할 수 있다.

<br>

> 참고: select , from 을 selectFrom 으로 합칠 수 있음

<br><br>

# 검색 조건

```java
member.username.eq("member1") // username = 'member1'
member.username.ne("member1") //username != 'member1'
member.username.eq("member1").not() // username != 'member1'
member.username.isNotNull() //이름이 is not null
member.age.in(10, 20) // age in (10,20)
member.age.notIn(10, 20) // age not in (10, 20)
member.age.between(10,30) //between 10, 30
member.age.goe(30) // age >= 30
member.age.gt(30) // age > 30
member.age.loe(30) // age <= 30
member.age.lt(30) // age < 30
member.username.like("member%") //like 검색
member.username.contains("member") // like ‘%member%’ 검색
member.username.startsWith("member") //like ‘member%’ 검색
...
```

<br><br>

# AND 조건을 파라미터로 처리

```java
@Test
public void searchAndParam() {
	Member findMember = queryFactory
			.selectFrom(member)
			.where(
					member.username.eq("member1"),	//	.and가 아닌 ,를 사용해서 and 조건을 추가할 수 있다.
					member.age.eq(10)
			)
			.fetchOne();

	Assertions.assertThat(findMember.getUsername()).isEqualTo("member1");
	Assertions.assertThat(findMember.getAge()).isEqualTo(10);
}
```

- 결과

```
JPQL :
select 
	member1
from 
	Member member1
where 
	member1.username = 'member1' 
	and member1.age = 10

SQL :
select 
	member0_.member_id as member_i1_0_, 
	member0_.age as age2_0_, 
	member0_.team_id as team_id4_0_, 
	member0_.username as username3_0_ 
from 
	member member0_ 
where 
	member0_.username='member1' 
	and member0_.age=10;

```

where() 에 파라미터로 검색조건을 추가하면 AND 조건이 추가된다. <br>
이 경우 null 값은 무시 메서드 추출을 활용해서 동적 쿼리를 깔끔하게 만들 수 있다.

<br><br><br>
## References 및 사진 출처

> __김영한의 실전! Querydsl__