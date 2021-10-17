---
layout: post
title:  "spring Querydsl 와 spring Data JPA 페이징"
subtitle:   "spring Querydsl 와 spring Data JPA 페이징"
date:   2021-10-17 21:20:27 +0900
categories: spring
tags: spring JPA ORM Mapping QueryDSL SpringDataJPA Paging
comments: true
---


<br>

- 목차
	- [Querydsl 페이징 연동](#querydsl-페이징-연동)
	- [전체 카운트를 한번에 조회하는 단순한 방법](#전체-카운트를-한번에-조회하는-단순한-방법)
	- [데이터 내용과 전체 카운트를 별도로 조회하는 방법](#데이터-내용과-전체-카운트를-별도로-조회하는-방법)
	- [CountQuery 최적화](#countquery-최적화)
	- [스프링 데이터 정렬(Sort)](#스프링-데이터-정렬sort)
	
<br>

# Querydsl 페이징 연동

- 스프링 데이터의 Page, Pageable을 활용해보자.
- 전체 카운트를 한번에 조회하는 단순한 방법
- 데이터 내용과 전체 카운트를 별도로 조회하는 방법

<br><br>

# 전체 카운트를 한번에 조회하는 단순한 방법

```java
public interface MemberRepositoryCustom {

    Page<MemberTeamDto> searchPageSimple(MemberSearchCondition condition, Pageable pageable);
}

public class MemberRepositoryImpl implements MemberRepositoryCustom{

    private final JPAQueryFactory queryFactory;

    public MemberRepositoryImpl(EntityManager em) {
        this.queryFactory = new JPAQueryFactory(em);
    }

    @Override
    public Page<MemberTeamDto> searchPageSimple(MemberSearchCondition condition, Pageable pageable) {
        QueryResults<MemberTeamDto> results = queryFactory
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
                .offset(pageable.getOffset())
                .limit(pageable.getPageSize())
                .fetchResults();

        List<MemberTeamDto> contents = results.getResults();
        long total = results.getTotal();

        return new PageImpl<>(contents, pageable, total);
    }
}

@SpringBootTest
@Transactional
class MemberRepositoryTest {


    @Autowired
    EntityManager em;

    @Autowired
    MemberRepository memberRepository;

    @Test
    public void searchPageSimple() {

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

        MemberSearchCondition condition = new MemberSearchCondition();
        PageRequest request = PageRequest.of(0, 3);

        Page<MemberTeamDto> result = memberRepository.searchPageSimple(condition, request);

        Assertions.assertThat(result.getSize()).isEqualTo(3);
        Assertions.assertThat(result.getContent()).extracting("username").containsExactly("member1","member2","member3");

        System.out.println("result.getSize() = " + result.getSize());
        
        for (MemberTeamDto memberTeamDto : result.getContent()) {
            System.out.println("memberTeamDto = " + memberTeamDto);
        }
    }
}
```

- 결과

```sql
// 카운트 조회 쿼리
JPQL :
select
    count(member1) 
from
    Member member1   
left join
    member1.team as team 

SQL :    
select
    count(member0_.member_id) as col_0_0_ 
from
    member member0_ 
left outer join
    team team1_ 
        on member0_.team_id=team1_.team_id

// 데이터 조회 쿼리
JPQL :
select
    member1.id,
    member1.username,
    member1.age,
    team.id,
    team.name 
from
    Member member1   
left join
    member1.team as team 

SQL :   
select
    member0_.member_id as col_0_0_,
    member0_.username as col_1_0_,
    member0_.age as col_2_0_,
    team1_.team_id as col_3_0_,
    team1_.name as col_4_0_ 
from
    member member0_ 
left outer join
    team team1_ 
        on member0_.team_id=team1_.team_id limit ?

result.getSize() = 3
memberTeamDto = MemberTeamDto(memberId=3, username=member1, age=10, teamId=1, teamName=teamA)
memberTeamDto = MemberTeamDto(memberId=4, username=member2, age=20, teamId=1, teamName=teamA)
memberTeamDto = MemberTeamDto(memberId=5, username=member3, age=30, teamId=2, teamName=teamB)
```

위 결과와 같이 fetchResults()를 사용하면 데이터 내용과 전체 카운트를 별도로 조회할수 있다. 카운트 쿼리 + 데이터 쿼리 두개의 쿼리가 발생한다.

<br><br>

# 데이터 내용과 전체 카운트를 별도로 조회하는 방법

```java

// 사용자 정의 인터페이스에 페이징 2가지 추가
public interface MemberRepositoryCustom {

    Page<MemberTeamDto> searchPageComplex(MemberSearchCondition condition, Pageable pageable);
}

public class MemberRepositoryImpl implements MemberRepositoryCustom{

    private final JPAQueryFactory queryFactory;

    public MemberRepositoryImpl(EntityManager em) {
        this.queryFactory = new JPAQueryFactory(em);
    }

    @Override
    public Page<MemberTeamDto> searchPageComplex(MemberSearchCondition condition, Pageable pageable) {
        List<MemberTeamDto> contents = queryFactory
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
                .offset(pageable.getOffset())   //
                .limit(pageable.getPageSize())
                .fetch();

        // count 쿼리를 최적화 하기 위해 따로 쿼리를 실행한다.
        long total = queryFactory
                .select(member)
                .from(member)
                .leftJoin(member.team, team)
                .where(
                        usernameEq(condition.getUsername()),
                        teamNameEq(condition.getTeamName()),
                        ageGoe(condition.getAgeGoe()),
                        ageLog(condition.getAgeLoe())
                )
                .fetchCount();

        return new PageImpl<>(contents, pageable, total);
    }
}

@SpringBootTest
@Transactional
class MemberRepositoryTest {


    @Autowired
    EntityManager em;

    @Autowired
    MemberRepository memberRepository;

    @Test
    public void searchPageComplect() {

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

        MemberSearchCondition condition = new MemberSearchCondition();
        PageRequest request = PageRequest.of(0, 3);

        Page<MemberTeamDto> result = memberRepository.searchPageComplex(condition, request);

        Assertions.assertThat(result.getSize()).isEqualTo(3);
        Assertions.assertThat(result.getContent()).extracting("username").containsExactly("member1","member2","member3");

        System.out.println("result.getSize() = " + result.getSize());

        for (MemberTeamDto memberTeamDto : result.getContent()) {
            System.out.println("memberTeamDto = " + memberTeamDto);
        }
    }
}
```

- 결과

```sql
// 데이터 조회 쿼리
JPQL :
select
    member1.id,
    member1.username,
    member1.age,
    team.id,
    team.name 
from
    Member member1   
left join
    member1.team as team 

SQL :        
select
    member0_.member_id as col_0_0_,
    member0_.username as col_1_0_,
    member0_.age as col_2_0_,
    team1_.team_id as col_3_0_,
    team1_.name as col_4_0_ 
from
    member member0_ 
left outer join
    team team1_ 
        on member0_.team_id=team1_.team_id limit ?

// 데이터 조회 쿼리
JPQL :
select
    count(member1) 
from
    Member member1   
left join
    member1.team as team 

SQL :  
select
    count(member0_.member_id) as col_0_0_ 
from
    member member0_ 
left outer join
    team team1_ 
        on member0_.team_id=team1_.team_id

result.getSize() = 3
memberTeamDto = MemberTeamDto(memberId=3, username=member1, age=10, teamId=1, teamName=teamA)
memberTeamDto = MemberTeamDto(memberId=4, username=member2, age=20, teamId=1, teamName=teamA)
memberTeamDto = MemberTeamDto(memberId=5, username=member3, age=30, teamId=2, teamName=teamB)
```

위 코드와 같이 전체 카운트를 조회 하는 방법을 최적화 할 수 있으면 이렇게 분리하면 된다. (예를 들어서 전체 카운트를 조회할 때 조인 쿼리를 줄일 수 있다면 상당한 효과가 있다.)

<br><br>

# CountQuery 최적화

```java
public interface MemberRepositoryCustom {

    Page<MemberTeamDto> searchPageComplexOptimization(MemberSearchCondition condition, Pageable pageable);
}

public class MemberRepositoryImpl implements MemberRepositoryCustom{

    private final JPAQueryFactory queryFactory;

    public MemberRepositoryImpl(EntityManager em) {
        this.queryFactory = new JPAQueryFactory(em);
    }
    @Override
    public Page<MemberTeamDto> searchPageComplexOptimization(MemberSearchCondition condition, Pageable pageable) {
        List<MemberTeamDto> contents = queryFactory
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
                .offset(pageable.getOffset())
                .limit(pageable.getPageSize())
                .fetch();

        // count 쿼리를 최적화 하기 위해 따로 쿼리를 실행한다.
        JPAQuery<Member> countQuery = queryFactory
                .select(member)
                .from(member)
                .leftJoin(member.team, team)
                .where(
                        usernameEq(condition.getUsername()),
                        teamNameEq(condition.getTeamName()),
                        ageGoe(condition.getAgeGoe()),
                        ageLog(condition.getAgeLoe())
                );

        // count 쿼리가 생략 가능한 경우 생략해서 처리
        // 페이지 시작이면서 컨텐츠 사이즈가 페이지 사이즈보다 작을 때
        // 마지막 페이지 일 때 (offset + 컨텐츠 사이즈를 더해서 전체 사이즈 구함)
        // 위 상황일때 countQuery.fetchCount() 쿼리를 실행하지 않는다.
        return PageableExecutionUtils.getPage(contents, pageable, () -> countQuery.fetchCount());
    }
}


@SpringBootTest
@Transactional
class MemberRepositoryTest {


    @Autowired
    EntityManager em;

    @Autowired
    MemberRepository memberRepository;

    @Test
    public void searchPageComplexOptimization() {

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

        MemberSearchCondition condition = new MemberSearchCondition();
        PageRequest request = PageRequest.of(0, 5);

        Page<MemberTeamDto> result = memberRepository.searchPageComplexOptimization(condition, request);

        Assertions.assertThat(result.getSize()).isEqualTo(5);
        Assertions.assertThat(result.getContent()).extracting("username").containsExactly("member1","member2","member3","member4");

        System.out.println("result.getSize() = " + result.getSize());

        for (MemberTeamDto memberTeamDto : result.getContent()) {
            System.out.println("memberTeamDto = " + memberTeamDto);
        }
    }
}
```

- 결과

```sql
JPQL :
select
    member1.id,
    member1.username,
    member1.age,
    team.id,
    team.name 
from
    Member member1   
left join
    member1.team as team 

SQL :   
select
    member0_.member_id as col_0_0_,
    member0_.username as col_1_0_,
    member0_.age as col_2_0_,
    team1_.team_id as col_3_0_,
    team1_.name as col_4_0_ 
from
    member member0_ 
left outer join
    team team1_ 
        on member0_.team_id=team1_.team_id limit ?

result.getSize() = 5
memberTeamDto = MemberTeamDto(memberId=3, username=member1, age=10, teamId=1, teamName=teamA)
memberTeamDto = MemberTeamDto(memberId=4, username=member2, age=20, teamId=1, teamName=teamA)
memberTeamDto = MemberTeamDto(memberId=5, username=member3, age=30, teamId=2, teamName=teamB)
memberTeamDto = MemberTeamDto(memberId=6, username=member4, age=40, teamId=2, teamName=teamB)
```

위 코드와 같이 PageableExecutionUtils.getPage()를 사용하면 count 쿼리가 생략 가능한 경우 생략해서 처리한다. <br>
ex ). 페이지 시작이면서 컨텐츠 사이즈가 페이지 사이즈보다 작을 때, <br>
마지막 페이지 일 때 (offset + 컨텐츠 사이즈를 더해서 전체 사이즈 구함) <br>

위 결과와 같이 count쿼리는 발생하지 않는다.

<br><br>

# 스프링 데이터 정렬(Sort)

스프링 데이터 JPA는 자신의 정렬(Sort)을 Querydsl의 정렬(OrderSpecifier)로 편리하게 변경하는 기능을 제공한다. 스프링 데이터의 정렬을 Querydsl의 정렬로 직접 전환하는 방법은 다음 코드를 참고하자. <br>

```java
JPAQuery<Member> query = queryFactory.selectFrom(member);

for (Sort.Order o : pageable.getSort()) {
     PathBuilder pathBuilder = new PathBuilder(member.getType(),member.getMetadata());

    query.orderBy(new OrderSpecifier(o.isAscending() ? Order.ASC : Order.DESC,  pathBuilder.get(o.getProperty())));
}
List<Member> result = query.fetch()
```

정렬(Sort)은 조건이 조금만 복잡해져도 Pageable 의 Sort 기능을 사용하기 어렵다. 루트 엔티티 범위를 넘어가는 동적 정렬 기능이 필요하면 스프링 데이터 페이징이 제공하는 Sort 를 사용하기 보다는 파라미터를 받아서 직접 처리하는 것을 권장한다.

<br><br><br>
## References 및 사진 출처

> __김영한의 실전! Querydsl__