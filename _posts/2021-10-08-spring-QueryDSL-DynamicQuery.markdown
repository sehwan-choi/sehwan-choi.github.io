---
layout: post
title:  "spring Querydsl 동적 쿼리"
subtitle:   "spring Querydsl 동적 쿼리"
date:   2021-10-08 00:20:27 +0900
categories: spring
tags: spring JPA ORM Mapping QueryDSL DinamicQuery 동적쿼리
comments: true
---


<br>

- 목차
	- [동적 쿼리](#동적-쿼리)
	- [BooleanBuilder](#booleanbuilder)
	- [Where 다중 파라미터 사용](#where-다중-파라미터-사용)
	
<br>

# 동적 쿼리

동적 쿼리를 해결하는 두가지 방식이 있다. <br>
BooleanBuilder , Where 다중 파라미터 사용방법 이다.

<br><br>

# BooleanBuilder

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
public void dynamicQuery_BooleanBuilder() {
    String usernameParam = "member1";
    Integer ageParam = 10;

    List<Member> result = sertchMember1(usernameParam, ageParam);
    for (Member member1 : result) {
        System.out.println("member1 = " + member1);
    }
}

private List<Member> sertchMember1(String usernameParam, Integer ageParam) {
    
    BooleanBuilder builder = new BooleanBuilder();
    if(usernameParam != null) {
        builder.and(member.username.eq(usernameParam));
    }
    
    if(ageParam != null) {
        builder.and(member.age.eq(ageParam));
    }
    
    return queryFactory
            .selectFrom(member)
            .where(builder)
            .fetch();
}
```

- 결과

```sql
JPQL :
select
    member1 
from
    Member member1 
where
    member1.username = ?1 
    and member1.age = ?2
        
SQL :
select
    member0_.member_id as member_i1_0_,
    member0_.age as age2_0_,
    member0_.team_id as team_id4_0_,
    member0_.username as username3_0_ 
from
    member member0_ 
where
    member0_.username=? 
    and member0_.age=?

member1 = Member(id=3, username=member1, age=10)
```

<br>

- 위 코드에서 ageParam = null 이라면?

```sql
JPQL :
select
    member1 
from
    Member member1 
where
    member1.username = ?1

SQL :
select
    member0_.member_id as member_i1_0_,
    member0_.age as age2_0_,
    member0_.team_id as team_id4_0_,
    member0_.username as username3_0_ 
from
    member member0_ 
where
    member0_.username=?
```

위 결과와 같이 where절에 username으로만 쿼리가 실행된다.

<br><br>

# Where 다중 파라미터 사용

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
public void dynamicQuery_WhereParam() {
    String usernameParam = "member1";
    Integer ageParam = null;

    List<Member> result = sertchMember2(usernameParam, ageParam);
    for (Member member1 : result) {
        System.out.println("member1 = " + member1);
    }
}

private List<Member> sertchMember2(String usernameParam, Integer ageParam) {
    return queryFactory
            .selectFrom(member)
            .where(usernameEq(usernameParam), ageEq(ageParam))
            .fetch();
}

private BooleanExpression usernameEq(String username) {
    if (username == null) {
        return null;
    }
    return member.username.eq(username);
}

private BooleanExpression ageEq(Integer age) {
    if (age == null) {
        return null;
    }
    return member.age.eq(age);
}
```

- 결과

```sql
JPQL :
select
    member1 
from
    Member member1 
where
    member1.username = ?1 

SQL :
select
    member0_.member_id as member_i1_0_,
    member0_.age as age2_0_,
    member0_.team_id as team_id4_0_,
    member0_.username as username3_0_ 
from
    member member0_ 
where
    member0_.username=?

member1 = Member(id=3, username=member1, age=10)
```

위 코드와 같이 where절에 null이 들어오게 되면 무시가 된다. 또 usernameEq(), ageEq() 와같은 메서드를 다른 쿼리에서도 재활용 할 수 있고 쿼리 자체의 가독성이 높아진다.

- 조합

```java
private BooleanExpression usernameEq(String username) {
    if (username == null) {
        return null;
    }
    return member.username.eq(username);
}

private BooleanExpression ageEq(Integer age) {
    if (age == null) {
        return null;
    }
    return member.age.eq(age);
}

private BooleanExpression allEq(String username, Integer age) {
    return usernameEq(username).and(ageEq(age));
}
```

위와 같이 usernameEq()와 ageEq() 메서드를 가지고 allEq()라는 메서드로 조합해서 사용할 수 있다.

<br>

> 주의! <br>
null 체크는 주의해서 처리해야한다.

<br><br><br>
## References 및 사진 출처

> __김영한의 실전! Querydsl__