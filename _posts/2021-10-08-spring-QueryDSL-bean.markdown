---
layout: post
title:  "spring Querydsl JPAQueryFactory 스프링 빈 등록"
subtitle:   "spring Querydsl JPAQueryFactory 스프링 빈 등록"
date:   2021-10-08 02:40:27 +0900
categories: spring
tags: spring JPA ORM Mapping QueryDSL JPAQueryFactory 스프링 빈 등록
comments: true
---


<br>

- 목차
	- [JPAQueryFactory 스프링 빈 등록](#jpaqueryfactory-스프링-빈-등록)
	
<br>

# JPAQueryFactory 스프링 빈 등록

```java
@SpringBootApplication
public class QuerydslApplication {

	public static void main(String[] args) {
		SpringApplication.run(QuerydslApplication.class, args);
	}

    // jpaQueryFactory Bean 생성
	@Bean
	JPAQueryFactory jpaQueryFactory(EntityManager em) {
		return new JPAQueryFactory(em);
	}
}

// jpaQueryFactory Bean 생성전
@Repository
public class MemberJpaRepository {

    private final EntityManager em;
    private final JPAQueryFactory queryFactory;

    public MemberJpaRepository(EntityManager em) {
        this.em = em;
        queryFactory = new JPAQueryFactory(em);
    }

    ...
}

// jpaQueryFactory Bean 생성후
@Repository
@RequiredArgsConstructor
public class MemberJpaRepository {

    private final EntityManager em;
    private final JPAQueryFactory queryFactory;

    ...
}
```

위 코드와 같이 jpaQueryFactory Bean을 생성하고 JPAQueryFactory를 사용하는 곳에서 @RequiredArgsConstructor 어노테이션을 같이 사용하게 된다면 더욱더 깔끔하게 코드를 구현할 수 있다. <br><br>

> 참고! <br>
동시성 문제는 걱정하지 않아도 된다. 왜냐하면 여기서 스프링이 주입해주는 엔티티 매니저는 실제 동작 시점에 진짜 엔티티 매니저를 찾아주는 프록시용 가짜 엔티티 매니저이다. 이 가짜 엔티티 매니저는 실제 사용 시점에 트랜잭션 단위로 실제 엔티티 매니저(영속성 컨텍스트)를 할당해준다.

<br><br><br>
## References 및 사진 출처

> __김영한의 실전! Querydsl__