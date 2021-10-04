---
layout: post
title:  "spring Querydsl Case"
subtitle:   "spring Querydsl Case"
date:   2021-10-05 01:55:27 +0900
categories: spring
tags: spring JPA ORM Mapping QueryDSL Case
comments: true
---


<br>

- 목차
	- [Case](#case)
	- [단순한 조건 Case](#단순한-조건-case)
	- [복잡한 조건 Case](#복잡한-조건-case)
	- [orderBy에서 Case 문 함께 사용하기](#orderby에서-case-문-함께-사용하기)
	
<br>

# Case

select, 조건절(where), order by에서 사용 가능하다.

<br><br>

# 단순한 조건 Case

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
    public void basicCase() {
        List<String> result = queryFactory
                .select(member.age
                        .when(10).then("열살")
                        .when(20).then("스무살")
                        .otherwise("기타"))
                .from(member)
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
    case 
        when member1.age = ?1 then ?2 
        when member1.age = ?3 then ?4 
        else '기타' 
    end 
from
    Member member1

SQL :  
select
    case 
        when member0_.age=? then ? 
        when member0_.age=? then ? 
        else '기타' 
    end as col_0_0_ 
from
    member member0_

s = 열살
s = 스무살
s = 기타
s = 기타
```

<br><br>

# 복잡한 조건 Case

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
    public void complexCase() {
        List<String> result = queryFactory
                .select(new CaseBuilder()
                        .when(member.age.between(0, 20)).then("0~20살")
                        .when(member.age.between(21, 30)).then("21~30살")
                        .otherwise("기타"))
                .from(member)
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
    case 
        when (member1.age between ?1 and ?2) then ?3 
        when (member1.age between ?4 and ?5) then ?6 
        else '기타' 
    end 
from
    Member member1

SQL :   
select
    case 
        when member0_.age between ? and ? then ? 
        when member0_.age between ? and ? then ? 
        else '기타' 
    end as col_0_0_ 
from
    member member0_

s = 0~20살
s = 0~20살
s = 21~30살
s = 기타
```

<br><br>

# orderBy에서 Case 문 함께 사용하기

다음과 같은 임의의 순서로 회원을 출력하고 싶다면?
1. 0 ~ 30살이 아닌 회원을 가장 먼저 출력
2. 0 ~ 20살 회원 출력
3. 21 ~ 30살 회원 출력

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
    public void orderByCase() {
        NumberExpression<Integer> rankPath = new CaseBuilder()
                .when(member.age.between(0,20)).then(2)
                .when(member.age.between(21,30)).then(1)
                .otherwise(3);

        List<Tuple> result = queryFactory
                .select(member.username, member.age, rankPath)
                .from(member)
                .orderBy(rankPath.desc())
                .fetch();

        for (Tuple tuple : result) {
            String username = tuple.get(member.username);
            Integer age = tuple.get(member.age);
            Integer rank = tuple.get(rankPath);
            System.out.println("username = " + username + " age = " + age + " rank = "
                    + rank);
        }
    }
```

- 결과

```SQL
JPQL :
select
    member1.username,
    member1.age,
    case 
        when (member1.age between ?1 and ?2) then ?3 
        when (member1.age between ?4 and ?5) then ?6 
        else 3 
    end 
from
    Member member1 
order by
    case 
        when (member1.age between ?1 and ?2) then ?3 
        when (member1.age between ?4 and ?5) then ?6 
        else 3 
    end desc

SQL :
select
    member0_.username as col_0_0_,
    member0_.age as col_1_0_,
    case 
        when member0_.age between ? and ? then ? 
        when member0_.age between ? and ? then ? 
        else 3 
    end as col_2_0_ 
from
    member member0_ 
order by
    case 
        when member0_.age between ? and ? then ? 
        when member0_.age between ? and ? then ? 
        else 3 
    end desc

username = member4 age = 40 rank = 3
username = member1 age = 10 rank = 2
username = member2 age = 20 rank = 2
username = member3 age = 30 rank = 1
```

위 코드와 같이 Querydsl은 자바 코드로 작성하기 때문에 rankPath 처럼 복잡한 조건을 변수로 선언해서 select 절, orderBy 절에서 함께 사용할 수 있다.

<br><br><br>
## References 및 사진 출처

> __김영한의 실전! Querydsl__