---
layout: post
title:  "spring JPQL Querydsl 문법비교"
subtitle:   "spring JPQL vs Querydsl"
date:   2021-09-27 19:29:27 +0900
categories: spring
tags: spring JPA ORM Mapping QueryDSL
comments: true
---


<br>

- 목차
	- [JPQL](#jpql)
	- [QueryDSL](#querydsl)
	- [비교](#비교)
    
<br>

# JPQL

```java
@SpringBootTest
@Transactional
public class QuerydslBasicTest {

    @Autowired
    EntityManager em;

    @BeforeEach
    public void before() {
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
    public void testJPQL() {
        Member findMember = em.createQuery("select m from Member m where m.username = :username", Member.class)
                .setParameter("username", "member1")
                .getSingleResult();

        Assertions.assertThat(findMember.getUsername()).isEqualTo("member1");
    }
}
```

<br><br>

# QueryDSL

```java
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
        QMember m = new QMember("m");

        Member findMember = queryFactory
                .select(m)
                .from(m)
                .where(m.username.eq("member1"))
                .fetchOne();

        Assertions.assertThat(findMember.getUsername()).isEqualTo("member1");
    }
}
```

<br><br>

> 만일 QMember가 없다고 에러가 나오는경우!

![그림1](https://sehwan-choi.github.io/assets/img/spring/QueryDSL/jpa1.jpg)

위 사진과 같이 우측 Gradle -> qureydsl -> Tasks -> other -> compileQuerydsl을 더블클릭 하게 되면 

![그림1](https://sehwan-choi.github.io/assets/img/spring/QueryDSL/jpa2.jpg)

위 사진과 같이 프로젝트 build -> generated -> querydsl 하위에 QMember가 생긴다.

<br><br>

# 비교

JQPL은 query문이 문자열로 되어있기 때문에, 컴파일 시점에서 에러를 확인 하기 어렵다. 또한 파라미터 바인딩도 문자열로 되어있기 때문에 사용자가 직접 API를 호출하기 전까지는 에러가 발생하는지 모른다. <br>
반면 QueryDSL은 query문이 메서드형식으로 제공하기 때문에 컴파일 시점에서 에러를 확인 할 수 있다. 또한 파라미터 바인딩도 자동으로 처리하고 동적 쿼리를 쉽게 사용할 수 있다.

<br>

> JPAQueryFactory를 필드로 제공하면 동시성 문제는 어떻게 될까? <br> 동시성 문제는 JPAQueryFactory를 생성할 때 제공하는 EntityManager(em)에 달려있다. 스프링 프레임워크는 여러 쓰레드에서 동시에 같은 EntityManager에 접근해도, 트랜잭션 마다 별도의 영속성 컨텍스트를 제공하기 때문에, 동시성 문제는 걱정하지 않아도 된다.

<br><br><br>
## References 및 사진 출처

> __김영한의 실전! 스프링 데이터 JPA__