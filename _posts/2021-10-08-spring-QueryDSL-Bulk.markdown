---
layout: post
title:  "spring Querydsl 수정, 삭제 벌크 연산"
subtitle:   "spring Querydsl 수정, 삭제 벌크 연산"
date:   2021-10-08 00:40:27 +0900
categories: spring
tags: spring JPA ORM Mapping QueryDSL Bulk
comments: true
---


<br>

- 목차
	- [수정 벌크연산](#수정-벌크연산)
	- [수정 벌크연산(사칙연산)](#수정-벌크연산사칙연산)
	- [삭제 벌크연산](#삭제-벌크연산)
	
<br>

JPA에서는 Dirty Cheking이라고 값이 변경되면 영속성 컨텍스트에 반영되고 SQL이 실행되어 DB에 저장된다. 하지만 이런 Dirty Cheking을 사용할 경우 대량의 데이터를 수정한다면 건건이 쿼리가 발생하므로 비효율 적이다. 벌크연산을 사용하면 대량의 데이터를 하나의 쿼리로 수정할수 있다.

<br><br>

# 수정 벌크연산

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
public void bulkUpdate() {
    long count = queryFactory
            .update(member)
            .set(member.username, "비회원")
            .where(member.age.lt(25))
            .execute();

    List<Member> fetch = queryFactory
            .selectFrom(member)
            .fetch();
    for (Member fetch1 : fetch) {
        System.out.println("fetch1 = " + fetch1);
    }
}
```

- 결과 

```sql
JPQL :
select
        member1 
    from
        Member member1

SQL :  
select
    member0_.member_id as member_i1_0_,
    member0_.age as age2_0_,
    member0_.team_id as team_id4_0_,
    member0_.username as username3_0_ 
from
    member member0_

fetch1 = Member(id=3, username=member1, age=10)
fetch1 = Member(id=4, username=member2, age=20)
fetch1 = Member(id=5, username=member3, age=30)
fetch1 = Member(id=6, username=member4, age=40)
```

위 코드에서 빠진것이 있다. 심각한 장애로 이어질수 있는 것인데, JPQL 배치와 마찬가지로, 영속성 컨텍스트에 있는 엔티티를 무시하고 실행되기 때문에 배치 쿼리를 실행하고 나면 영속성 컨텍스트를 초기화 하는 것이 안전하다. 


```java
@Test
public void bulkUpdate() {
    long count = queryFactory
            .update(member)
            .set(member.username, "비회원")
            .where(member.age.lt(25))
            .execute();

    em.flush();
    em.clear();

    List<Member> fetch = queryFactory
            .selectFrom(member)
            .fetch();
    for (Member fetch1 : fetch) {
        System.out.println("fetch1 = " + fetch1);
    }
}
```

- 결과

```sql
fetch1 = Member(id=3, username=비회원, age=10)
fetch1 = Member(id=4, username=비회원, age=20)
fetch1 = Member(id=5, username=member3, age=30)
fetch1 = Member(id=6, username=member4, age=40)
```

영속성 컨텍스트에 member1 이란 데이터가 있는 상태로, 어떤 sql이 실행되어 member1을 가져오게 되면 영속성 컨텍스트는 이미 member1이란 데이터가 있기 때문에 sql에서 가져온 member1은 버리게 된다. 그래서 반드시 bulk연산후 다른 비지니스 로직이 있다면 같이 em.flush(), em.clear()를 호출하여 영속성 컨텍스트를 비워야 한다.

<br><br>

# 수정 벌크연산(사칙연산)

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
public void bulkCalculations() {
    long execute = queryFactory
            .update(member)
            .set(member.age, member.age.add(1))         //  더하기
            //.set(member.age, member.age.add(-1))      //  빼기
            //.set(member.age, member.age.multiply(2))  //  곱하기
            //.set(member.age, member.age.divide(2))    //  나누기
            .execute();

    em.flush();
    em.clear();

    List<Member> fetch = queryFactory
            .selectFrom(member)
            .fetch();
    for (Member fetch1 : fetch) {
        System.out.println("fetch1 = " + fetch1);
    }
}
```

- 결과

```sql
JPQL :
update
    Member member1 
set
    member1.age = member1.age + ?1
        
SQL :
update
    member 
set
    age=age+?

fetch1 = Member(id=3, username=member1, age=11)
fetch1 = Member(id=4, username=member2, age=21)
fetch1 = Member(id=5, username=member3, age=31)
fetch1 = Member(id=6, username=member4, age=41)
```

위 코드와 같이 add, multiply, divide를 통해서 사칙연산이 가능하다.

<br><br>

# 삭제 벌크연산

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
public void bulkDeldete() {
    long execute = queryFactory
            .delete(member)
            .where(member.age.gt(18))
            .execute();

    List<Member> result = queryFactory
            .selectFrom(member)
            .fetch();

    for (Member member1 : result) {
        System.out.println("member1 = " + member1);
    }
}
```

- 결과

```sql
JPQL :
delete 
    from
        Member member1 
    where
        member1.age > ?1

SQL :
delete 
    from
        member 
    where
        age>?

member1 = Member(id=3, username=member1, age=10)
```

<br><br><br>
## References 및 사진 출처

> __김영한의 실전! Querydsl__