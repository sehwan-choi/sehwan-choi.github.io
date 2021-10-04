---
layout: post
title:  "spring Querydsl 상수, 문자 더하기"
subtitle:   "spring Querydsl 상수, 문자 더하기"
date:   2021-10-05 02:35:27 +0900
categories: spring
tags: spring JPA ORM Mapping QueryDSL 상수,문자더하기
comments: true
---


<br>

- 목차
	- [상수, 문자 더하기](#상수-문자-더하기)
	- [문자 더하기 concat](#문자-더하기-concat)
	
<br>

# 상수, 문자 더하기

상수가 필요하면 Expressions.constant(xxx) 사용한다.

```java
Tuple result = queryFactory
    .select(member.username, Expressions.constant("A"))
    .from(member)
    .fetchFirst();
 ```

> 참고: 위와 같이 최적화가 가능하면 SQL에 constant 값을 넘기지 않는다. 상수를 더하는 것 처럼 최적화가 어려우면 SQL에 constant 값을 넘긴다.

<br><br>

```java
    JPAQueryFactory queryFactory;

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
    public void constant() {
        List<Tuple> result = queryFactory
                .select(member.username, Expressions.constant("A"))
                .from(member)
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
    member1.username 
from
    Member member1

SQL :  
select
    member0_.username as col_0_0_ 
from
    member member0_

tuple = [member1, A]
tuple = [member2, A]
tuple = [member3, A]
tuple = [member4, A]
```

<br><br>

# 문자 더하기 concat

```java
    JPAQueryFactory queryFactory;

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
    public void concat() {
        List<String> result = queryFactory
                .select(member.username.concat("_").concat(member.age.stringValue()))
                .from(member)
                .where(member.username.eq("member1"))
                .fetch();

        for (String s : result) {
            System.out.println("s = " + s);
        }
    }
```

- 결과

```SQL
JPQL :
select
    concat(concat(member1.username,
    ?1),
    str(member1.age)) 
from
    Member member1 
where
    member1.username = ?2 

SQL :  
select
    ((member0_.username||?)||cast(member0_.age as char)) as col_0_0_ 
from
    member member0_ 
where
    member0_.username=?

s = member1_10
```

위 코드에서 stringValue() 사용한 이유는 member.age가 int 타입이기 때문에 String으로 변환을 해서 사용해야 한다.

>  참고: member.age.stringValue() 부분이 중요한데, 문자가 아닌 다른 타입들은 stringValue() 로 문자로 변환할 수 있다. 이 방법은 ENUM을 처리할 때도 자주 사용한다.

<br><br><br>
## References 및 사진 출처

> __김영한의 실전! Querydsl__