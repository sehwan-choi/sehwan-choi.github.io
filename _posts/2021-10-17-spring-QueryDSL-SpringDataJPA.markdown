---
layout: post
title:  "spring Querydsl 와 spring Data JPA 사용"
subtitle:   "spring Querydsl 와 spring Data JPA 사용"
date:   2021-10-17 19:20:27 +0900
categories: spring
tags: spring JPA ORM Mapping QueryDSL SpringDataJPA
comments: true
---


<br>

- 목차
	- [SpringDataJPA와 SpringQueryDSL 사용하기](#springdatajpa와-springquerydsl-사용하기)
	- [1. 사용자 정의 인터페이스 작성](#1-사용자-정의-인터페이스-작성)
	- [2. 사용자 정의 인터페이스 구현](#2-사용자-정의-인터페이스-구현)
	- [3. 스프링 데이터 리포지토리에 사용자 정의 인터페이스 상속](#3-스프링-데이터-리포지토리에-사용자-정의-인터페이스-상속)
	
<br>

# SpringDataJPA와 SpringQueryDSL 사용하기

[Spring Data JPA 사용자정의 레포지토리 구현](https://sehwan-choi.github.io/spring/2021/09/18/spring-DATA-JPA-Extends/) 의 글을 참고해도 좋다.

사용자 정의 리포지토리 사용법
1. 사용자 정의 인터페이스 작성
2. 사용자 정의 인터페이스 구현
3. 스프링 데이터 리포지토리에 사용자 정의 인터페이스 상속


![그림1](https://sehwan-choi.github.io/assets/img/spring/QueryDSL/jpa6.jpg)

<br><br>

# 1. 사용자 정의 인터페이스 작성

```java
public interface MemberRepositoryCustom {

    List<MemberTeamDto> search(MemberSearchCondition condition);
}
```

<br><br>

# 2. 사용자 정의 인터페이스 구현

```java
public class MemberRepositoryImpl implements MemberRepositoryCustom{

    private final JPAQueryFactory queryFactory;

    public MemberRepositoryImpl(EntityManager em) {
        this.queryFactory = new JPAQueryFactory(em);
    }

    @Override
    public List<MemberTeamDto> search(MemberSearchCondition condition) {
        return queryFactory
                .select(new QMemberTeamDto(
                        member.id,
                        member.username,
                        member.age,
                        team.id,
                        team.name
                ))
                .from(member)
                .leftJoin(member.team, team)
                .where(
                        usernameEq(condition.getUsername()),
                        teamNameEq(condition.getTeamName()),
                        ageGoe(condition.getAgeGoe()),
                        ageLog(condition.getAgeLoe())
                )
                .fetch();
    }

    private BooleanExpression usernameEq(String username) {
        return StringUtils.hasText(username) ? member.username.eq(username) : null;
    }

    private BooleanExpression teamNameEq(String teamName) {
        return StringUtils.hasText(teamName) ? team.name.eq(teamName) : null;
    }

    private BooleanExpression ageGoe(Integer ageGoe) {
        return ageGoe != null ? member.age.goe(ageGoe) : null;
    }

    private BooleanExpression ageLog(Integer ageLoe) {
        return ageLoe != null ? member.age.loe(ageLoe) : null;
    }
}
```

MemberRepositoryCustom을 상속받아 메소드를 QueryDSL로 구현한다.

<br><br>

# 3. 스프링 데이터 리포지토리에 사용자 정의 인터페이스 상속

```java
public interface MemberRepository extends JpaRepository<Member, Long>, MemberRepositoryCustom {

    List<Member> findByUsername(String username);
}
```

JpaRepository와 MemberRepositoryCustom을 상속 받게 되면 MemberRepositoryCustom을 상속 하고있는 MemberRepositoryImpl의 search 메소드를 사용할 수 있다.

<br>

> 주의! <br>
사용자 정의 인터페이스를 구현한 클래스이름은 반드시 JpaRepository를 상속받는 인터페이스 명(MemberRepository) + Impl로 만들어야 한다. 


<br><br><br>
## References 및 사진 출처

> __김영한의 실전! Querydsl__