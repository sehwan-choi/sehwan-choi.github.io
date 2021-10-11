---
layout: post
title:  "spring Querydsl 동적 쿼리 DTO 조회"
subtitle:   "spring Querydsl 동적 쿼리 DTO 조회"
date:   2021-10-12 02:20:27 +0900
categories: spring
tags: spring JPA ORM Mapping QueryDSL DinamicQuery Dto
comments: true
---


<br>

- 목차
	- [동적 쿼리와 성능 최적화 조회 - Builder 사용](#동적-쿼리와-성능-최적화-조회---builder-사용)
	- [동적 쿼리와 성능 최적화 조회 - Where절 파라미터 사용](#동적-쿼리와-성능-최적화-조회---where절-파라미터-사용)
	
<br>

> 참고 ! <br>
@QueryProjection 을 추가하면 Q 파일을 생성하기 위해 compileQuerydsl 을 한번 실행하자. 아래 블로그 참고 <br>
[https://sehwan-choi.github.io/spring/2021/10/06/spring-QueryDSL-Projection/#queryprojection](https://sehwan-choi.github.io/spring/2021/10/06/spring-QueryDSL-Projection/#queryprojection)

<br><br>

# 동적 쿼리와 성능 최적화 조회 - Builder 사용

```java
@Data
public class MemberTeamDto {

    private Long memberId;
    private String username;
    private int age;
    private Long teamId;
    private String teamName;

    @QueryProjection
    public MemberTeamDto(Long memberId, String username, int age, Long teamId, String teamName) {
        this.memberId = memberId;
        this.username = username;
        this.age = age;
        this.teamId = teamId;
        this.teamName = teamName;
    }
}


@Repository
public class MemberJpaRepository {

    private EntityManager em;
    private JPAQueryFactory queryFactory;

    public MemberDslRepository(EntityManager em) {
        this.em = em;
        queryFactory = new JPAQueryFactory(em);
    }

    public List<MemberTeamDto> searchByBuilder(MemberSearchCondition condition) {

        BooleanBuilder builder = new BooleanBuilder();
        if(StringUtils.hasText(condition.getUsername())) {
            builder.and(member.username.eq(condition.getUsername()));
        }
        if(StringUtils.hasText(condition.getTeamName())) {
            builder.and(team.name.eq(condition.getTeamName()));
        }
        if(condition.getAgeGoe() != null) {
            builder.and(member.age.goe(condition.getAgeGoe()));
        }
        if(condition.getAgeLoe() != null) {
            builder.and(member.age.loe(condition.getAgeLoe()));
        }

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
                .where(builder)
                .fetch();
    }
}


@SpringBootTest
@Transactional
class MemberJpaRepositoryTest {

    @Autowired
    EntityManager em;

    @Autowired
    MemberJpaRepository memberJpaRepository;

    @Test
    public void searchTest() {

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
        condition.setAgeGoe(35);
        condition.setAgeLoe(40);
        condition.setTeamName("teamB");

        List<MemberTeamDto> result = memberJpaRepository.searchByBuilder(condition);

        Assertions.assertThat(result).extracting("username").containsExactly("member4");
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
where
    team.name = ?1 
    and member1.age >= ?2 
    and member1.age <= ?3

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
        on member0_.team_id=team1_.team_id 
where
    team1_.name=? 
    and member0_.age>=? 
    and member0_.age<=?

memberTeamDto = MemberTeamDto(memberId=6, username=member4, age=40, teamId=2, teamName=teamB)
```

위 코드와 같이 where 절을 BooleanBuilder builder = new BooleanBuilder(); 로 하나하나 붙여서 만든다.

<br><br>

# 동적 쿼리와 성능 최적화 조회 - Where절 파라미터 사용

```java
@Data
public class MemberTeamDto {

    private Long memberId;
    private String username;
    private int age;
    private Long teamId;
    private String teamName;

    @QueryProjection
    public MemberTeamDto(Long memberId, String username, int age, Long teamId, String teamName) {
        this.memberId = memberId;
        this.username = username;
        this.age = age;
        this.teamId = teamId;
        this.teamName = teamName;
    }
}


@Repository
public class MemberJpaRepository {

    private EntityManager em;
    private JPAQueryFactory queryFactory;

    public MemberDslRepository(EntityManager em) {
        this.em = em;
        queryFactory = new JPAQueryFactory(em);
    }

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


@SpringBootTest
@Transactional
class MemberJpaRepositoryTest {

    @Autowired
    EntityManager em;

    @Autowired
    MemberJpaRepository memberJpaRepository;

    @Test
    public void searchTest2() {

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
        condition.setAgeGoe(35);
        condition.setAgeLoe(40);
        condition.setTeamName("teamB");

        List<MemberTeamDto> result = memberJpaRepository.search(condition);

        Assertions.assertThat(result).extracting("username").containsExactly("member4");

        for (MemberTeamDto memberTeamDto : result) {
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
where
    team.name = ?1 
    and member1.age >= ?2 
    and member1.age >= ?2

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
        on member0_.team_id=team1_.team_id 
where
    team1_.name=? 
    and member0_.age>=? 
    and member0_.age>=?

memberTeamDto = MemberTeamDto(memberId=6, username=member4, age=40, teamId=2, teamName=teamB)
```

위 코드와 같이 각각 메서드를 만들어서 where절에 파라미터로 사용했다. where절에 null이 들어오게 되면 무시가 되며 usernameEq(), ageEq() 와같은 메서드를 다른 쿼리에서도 재활용 할 수 있고 쿼리 자체의 가독성이 높아진다.
아래와 같이 조합해서 사용할 수 있다.

```java
    private BooleanExpression ageGoe(Integer ageGoe) {
        return ageGoe != null ? member.age.goe(ageGoe) : null;
    }

    private BooleanExpression ageLog(Integer ageLoe) {
        return ageLoe != null ? member.age.loe(ageLoe) : null;
    }

    private BooleanExpression ageBetweeen(int ageLoe, int ageGoe) {
        return ageGoe(ageGoe).and(ageGoe(ageGoe));
    }
```

> 주의! <br>
null 체크는 주의해서 처리해야한다.

<br>

> 참고 ! <br>
@QueryProjection 을 사용하면 해당 DTO가 Querydsl을 의존하게 된다. 이런 의존이 싫으면, 해당 에노테이션을 제거하고, Projection.bean(), fields(), constructor() 을 사용하면 된다.

<br>


<br><br><br>
## References 및 사진 출처

> __김영한의 실전! Querydsl__