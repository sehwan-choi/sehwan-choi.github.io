---
layout: post
title:  "spring Querydsl 기본문법 - QType"
subtitle:   "spring Querydsl 기본문법 QType"
date:   2021-09-27 15:42:27 +0900
categories: spring
tags: spring JPA ORM Mapping QueryDSL QType
comments: true
---


<br>

- 목차
	- [Q클래스 인스턴스를 사용하는 2가지 방법](#q클래스-인스턴스를-사용하는-2가지-방법)
		- [별칭 직접 지정](#별칭-직접-지정)
		- [기본 인스턴스 사용](#기본-인스턴스-사용)
	- [기본 인스턴스를 static import와 함께 사용](#기본-인스턴스를-static-import와-함께-사용)
	- [JPQL 보기](#jpql-보기)
    
<br>

# Q클래스 인스턴스를 사용하는 2가지 방법

<br>

## 별칭 직접 지정

```java
QMember qMember = new QMember("aaaba"); //별칭 직접 지정
```

별칭을 직접 지정하게 되면, 지정한 별칭으로 쿼리가 실행된다.

```sql
select
	aaaba 
from
	Member aaaba 
where
	aaaba.username = ?1
```

<br><br>

## 기본 인스턴스 사용

```java
QMember qMember = QMember.member; //기본 인스턴스 사용
```

기본 인스턴스를 사용하게 되면 아래와 같이 QMember 클래스에 기본으로 설정된 별칭(member1)을 통해 쿼리가 실행된다.

```java
@Generated("com.querydsl.codegen.EntitySerializer")
public class QMember extends EntityPathBase<Member> {

	...

    public static final QMember member = new QMember("member1");

	...
}
```

```sql
select
	member1 
from
	Member member1 
where
	member1.username = ?1
```

<br>

> 참고: 같은 테이블을 조인해야 하는 경우가 아니면 기본 인스턴스를 사용하자.

<br><br>

# 기본 인스턴스를 static import와 함께 사용

```java
...

import static study.querydsl.entity.QMember.member;

@SpringBootTest
@Transactional
public class QuerydslBasicTest {

    @Autowired
    EntityManager em;

    JPAQueryFactory queryFactory;

    @BeforeEach
    public void before() {
        queryFactory = new JPAQueryFactory(em);

        Team teamA = new Team("teamA");
        Team teamB = new Team("teamB");
        em.persist(teamA);
        em.persist(teamB);

        Member member1 = new Member("member1", 10, teamA);
        Member member2 = new Member("member2", 10, teamA);
        Member member3 = new Member("member3", 10, teamB);
        Member member4 = new Member("member4", 10, teamB);

        em.persist(member1);
        em.persist(member2);
        em.persist(member3);
        em.persist(member4);
    }

	@Test
    public void testQuerydsl() {
        ///QMember m = new QMember("aaaba");
        //QMember m = QMember.member;

        Member findMember = queryFactory
                .select(member)
                .from(member)
                .where(member.username.eq("member1"))
                .fetchOne();

        Assertions.assertThat(findMember.getUsername()).isEqualTo("member1");
    }
}
```

스태틱 임포트(import static study.querydsl.entity.QMember.member;) 를 하게 되면 위의 testQuerydsl()처럼 member로만 사용 할수 있다.

<br><br>

# JPQL 보기

QueryDSL을 실행하면 SQL은 보이지만 JPQL은 따로 보이지 않는다. 보고싶다면 application.yml에 아래와 같이 설정한다.

```gradle
spring:
  datasource:
    url: jdbc:h2:tcp://localhost/~/datajpa
    username: sa
    password:
    driver-class-name: org.h2.Driver

  jpa:
    hibernate:
      ddl-auto: create
    properties:
      hibernate:
        # show_sql: true
        format_sql: true
        use_sql_comments: true		//	추가

server:
  port: 8080

logging.level:
  org.hibernate.SQL: debug
  # org.hibernate.type: trace
```

위와 같이 spring.jpa.properties.hibernate.use_sql_comments = true를 추가한다.

<br>

```sql
/* select
	member1 
from
	Member member1 
where
	member1.username = ?1 */ select
		member0_.member_id as member_i1_0_,
		member0_.age as age2_0_,
		member0_.team_id as team_id4_0_,
		member0_.username as username3_0_ 
	from
		member member0_ 
	where
		member0_.username=?
```

위와같이 JPQL과 SQL이 동시에 보이게 된다.

<br><br><br>
## References 및 사진 출처

> __김영한의 실전! Querydsl__