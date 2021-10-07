---
layout: post
title:  "spring Querydsl SQL Function 호출"
subtitle:   "spring Querydsl SQL Function 호출"
date:   2021-10-08 01:20:27 +0900
categories: spring
tags: spring JPA ORM Mapping QueryDSL SQLFunction
comments: true
---


<br>

- 목차
	- [SQL function 호출하기](#sql-function-호출하기)
	- [replace 함수 사용](#replace-함수-사용)
	- [lower, upper 함수 사용](#lower-upper-함수-사용)
	
<br>

# SQL function 호출하기

SQL function은 JPA와 같이 Dialect에 등록된 내용만 호출할 수 있다.

<br><br>

# replace 함수 사용

```java
@BeforeEach
public void before() {
    queryFactory = new JPAQueryFactory(em);

    Team teamA = new Team("teamA");
    Team teamB = new Team("teamB");
    em.persist(teamA);
    em.persist(teamB);

    Member member1 = new Member("member1", 10, teamA);
    Member member2 = new Member("member2", 20, teamA);
    Member member3 = new Member("member3", 30, teamB);
    Member member4 = new Member("member4", 40, teamB);

    em.persist(member1);
    em.persist(member2);
    em.persist(member3);
    em.persist(member4);
}

@Test
public void sqlFunction() {
    List<String> result = queryFactory
            .select(Expressions.stringTemplate(
                    "function('replace', {0}, {1}, {2})",
                    member.username, "member", "M"))
            .from(member)
            .fetch();

    for (String s : result) {
        System.out.println("s = " + s);
    }
}
```

- 결과

```sql
JPQL :
select
    function('replace',
    member1.username,
    ?1,
    ?2) 
from
    Member member1

SQL :  
select
    replace(member0_.username,
    ?,
    ?) as col_0_0_ 
from
    member member0_

s = M1 (member1)
s = M2 (member2)
s = M3 (member3)
s = M4 (member4)
```

위 코드에서 Expressions.stringTemplate를 통해 SQL Function을 사용 할 수 있다. {0} 에는 원본 문자열, {1} 에는 변경할 문자열, {2} 에는 변경될 문자열을 차례로 넣는다.

<br><br>

# lower, upper 함수 사용

```java
@BeforeEach
public void before() {
    queryFactory = new JPAQueryFactory(em);

    Team teamA = new Team("teamA");
    Team teamB = new Team("teamB");
    em.persist(teamA);
    em.persist(teamB);

    Member member1 = new Member("member1", 10, teamA);
    Member member2 = new Member("member2", 20, teamA);
    Member member3 = new Member("member3", 30, teamB);
    Member member4 = new Member("member4", 40, teamB);

    em.persist(member1);
    em.persist(member2);
    em.persist(member3);
    em.persist(member4);
}

@Test
public void sqlFunction2() {
    List<String> result = queryFactory
            .select(member.username.upper())
            .from(member)
            .where(member.username.eq(
                    //Expressions.stringTemplate("function('lower', {0})", member.username))
                    member.username.lower())
            )
            .fetch();

    for (String s : result) {
        System.out.println("s = " + s);
    }
}
```

- 결과

```sql
JPQL :
select
    upper(member1.username) 
from
    Member member1 
where
    member1.username = lower(member1.username)

SQL :
select
    upper(member0_.username) as col_0_0_ 
from
    member member0_ 
where
    member0_.username=lower(member0_.username)

s = MEMBER1
s = MEMBER2
s = MEMBER3
s = MEMBER4
```

기본적으로 lower 같은 ansi 표준 함수들은 querydsl이 상당부분 내장하고 있기 때문에 .upper(), .lower()로 사용할수 있다. 또, 사용자가 직접 정의해서 함수를 사용 할 수 있다. <br>
아래의 블로그를 참조. <br>
[spring JPA JQPL 기본함수와 사용자 정의함수](https://sehwan-choi.github.io/spring/2021/09/01/spring-JPA-JPQL6/#%EC%82%AC%EC%9A%A9%EC%9E%90-%EC%A0%95%EC%9D%98%ED%95%A8%EC%88%98)

<br><br><br>
## References 및 사진 출처

> __김영한의 실전! Querydsl__