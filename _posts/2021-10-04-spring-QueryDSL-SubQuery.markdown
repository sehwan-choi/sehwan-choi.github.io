---
layout: post
title:  "spring Querydsl SubQuery"
subtitle:   "spring Querydsl SubQuery"
date:   2021-10-04 17:55:27 +0900
categories: spring
tags: spring JPA ORM Mapping QueryDSL SubQuery
comments: true
---


<br>

- 목차
	- [SubQuery](#subquery)
	- [where절에 SubQuery 사용](#where절에-subquery-사용)
	- [select 절에 SubQuery 사용](#select-절에-subquery-사용)
	- [from 절의 SubQuery 한계](#from-절의-subquery-한계)
	
<br>

# SubQuery

SubQuery란 하나의 SQL 문 안에 포함되어 있는 또 다른 SQL문을 말한다. <br>
QueryDSL에서 SubQuery는 com.querydsl.jpa.JPAExpressions를 사용한다.

<br><br>

# where절에 SubQuery 사용

```java
    /**
     * 서브쿼리
     * 나이가 가장 많은 회원 조회
     */
    @Test
    public void subQuery() {

        QMember memberSub = new QMember("memberSub");   //  subQuery와 메인 Query절의 Member 별칭을 다르게 설정해야하기 때문에 메인 Query절의 Member는 기본값으로 사용하고 SubQuery의 Member 별칭을 설정한다.

        List<Member> result = queryFactory
                .selectFrom(member)
                .where(member.age.eq(
                        JPAExpressions  //  where절에 SubQuery 사용
                                .select(memberSub.age.max())
                                .from(memberSub)
                ))
                .fetch();
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
    member1.age = (
        select
            max(memberSub.age) 
        from
            Member memberSub
    )

SQL :
    select
    member0_.member_id as member_i1_0_,
    member0_.age as age2_0_,
    member0_.team_id as team_id4_0_,
    member0_.username as username3_0_ 
from
    member member0_ 
where
    member0_.age=(
        select
            max(member1_.age) 
        from
            member member1_
    )
```

<br><br>

```java
    /**
     * 서브쿼리
     * 여러건 처리, in절 사용
     */
    @Test
    public void subQueryIn() {

        QMember memberSub = new QMember("memberSub");   //  subQuery와 메인 Query절의 Member 별칭을 다르게 설정해야하기 때문에 메인 Query절의 Member는 기본값으로 사용하고 SubQuery의 Member 별칭을 설정한다.

        List<Member> result = queryFactory
                .selectFrom(member)
                .where(member.age.in(
                        JPAExpressions  //  where절에 SubQuery 사용
                                .select(memberSub.age)
                                .from(memberSub)
                                .where(memberSub.age.gt(10))
                ))
                .fetch();
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
    member1.age in (
        select
            memberSub.age 
        from
            Member memberSub 
        where
            memberSub.age > ?1
    )

SQL :        
select
    member0_.member_id as member_i1_0_,
    member0_.age as age2_0_,
    member0_.team_id as team_id4_0_,
    member0_.username as username3_0_ 
from
    member member0_ 
where
    member0_.age in (
        select
            member1_.age 
        from
            member member1_ 
        where
            member1_.age>?
    )
```

<br><br>

# select 절에 SubQuery 사용

```java
    /**
     * 서브쿼리
     * select 절에 subQuery 사용
     */
    @Test
    public void selectSubQuery() {
        QMember memberSub = new QMember("memberSub");   //  subQuery와 메인 Query절의 Member 별칭을 다르게 설정해야하기 때문에 메인 Query절의 Member는 기본값으로 사용하고 SubQuery의 Member 별칭을 설정한다.

        List<Tuple> result = queryFactory
                .select(member.username,
                        JPAExpressions
                                .select(memberSub.age.avg())
                                .from(memberSub))
                .from(member)
                .fetch();
    }
```

- 결과

```SQL
JPQL :
select
    member1.username,
    (select
        avg(memberSub.age) 
    from
        Member memberSub) 
from
    Member member1
        
SQL :
select
    member0_.username as col_0_0_,
    (select
        avg(cast(member1_.age as double)) 
    from
        member member1_) as col_1_0_ 
from
    member member0_
```

<br><br>

# from 절의 SubQuery 한계

JPA JPQL 서브쿼리의 한계점으로 from 절의 서브쿼리(인라인 뷰)는 지원하지 않는다. 당연히 Querydsl 도 지원하지 않는다. 하이버네이트 구현체를 사용하면 select 절의 서브쿼리는 지원한다. Querydsl도 하이버네이트 구현체를 사용하면 select 절의 서브쿼리를 지원한다. <br><br>

from 절의 서브쿼리 해결방안
1. 서브쿼리를 join으로 변경한다. (가능한 상황도 있고, 불가능한 상황도 있다.)
2. 애플리케이션에서 쿼리를 2번 분리해서 실행한다.
3. nativeSQL을 사용한다

<br><br><br>
## References 및 사진 출처

> __김영한의 실전! Querydsl__