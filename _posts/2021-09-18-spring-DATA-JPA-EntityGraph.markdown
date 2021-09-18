---
layout: post
title:  "spring Data JPA Entity Graph"
subtitle:   "spring Data JPA Entity Graph"
date:   2021-09-18 20:45:27 +0900
categories: spring
tags: spring JPA ORM Mapping String-Data-JPA Entity Graph
comments: true
---


<br>

- 목차
	- [지연 로딩](#지연-로딩)
	- [JPQL 페치 조인](#jpql-페치-조인)
	- [EntityGraph](#entitygraph)
    
<br>

# 지연 로딩

member -> team은 지연로딩 관계이다. 따라서 다음과 같이 team의 데이터를 조회할 때 마다 쿼리가 실행된다. (N+1 문제 발생)

```java
@SpringBootTest
@Transactional
@Rollback(false)
public class MemberRepositoryTest {

    @Autowired
    MemberRepository memberRepository;

    @Autowired
    TeamRepository teamRepository;

    @Test
    public void findMemberLazy() {
        //given
        //member1 -> teamA
        //member2 -> teamB

        Team teamA = new Team("teamA");
        Team teamB = new Team("teamB");
        teamRepository.save(teamA);
        teamRepository.save(teamB);

        Member member1 = new Member("member1", 10, teamA);
        Member member2 = new Member("member2", 10, teamB);
        memberRepository.save(member1);
        memberRepository.save(member2);

        em.flush();
        em.clear();

        //when
        //select Member N + 1 문제 발생
        List<Member> members = memberRepository.findAll();

        for (Member member : members) {
            System.out.println("member = " + member);
            System.out.println("    teamClass = " + member.getTeam().getClass());
            //select Team
            System.out.println("    teamName = " + member.getTeam().getName());
        }
    }
}
```

- 결과

```
select
    member0_.member_id as member_i1_0_,
    member0_.age as age2_0_,
    member0_.team_id as team_id4_0_,
    member0_.username as username3_0_ 
from
    member member0_

member = Member(id=3, username=member1, age=10)
    teamClass = class study.datajpa.entity.Team$HibernateProxy$QQqWPX5n

select
    team0_.team_id as team_id1_1_0_,
    team0_.name as name2_1_0_ 
from
    team team0_ 
where
    team0_.team_id=?

    teamName = teamA
member = Member(id=4, username=member2, age=10)

select
    team0_.team_id as team_id1_1_0_,
    team0_.name as name2_1_0_ 
from
    team team0_ 
where
    team0_.team_id=?

    teamName = teamB
```

위 결과와 같이 모든 Member를 가져오고 member.getTeam().getName() 호출에 의해, team 데이터를 가져오는 쿼리가 또 실행 됬음을 알수 있다. 이는 N + 1문제를 발생시킨다. 그리고 teamClass = class study.datajpa.entity.Team$HibernateProxy$QQqWPX5n로 이상하게 생성되어 있다. team class가 proxy 객체로 생성된 이유는 memberRepository.findAll()로 데이터를 가져왔을 때 위 쿼리에서 확인할수 있는 것처럼(제일 처음 쿼리) Member데이터만 가져오고, team 데이터는 가져오지 않은 것이다. LAZY 로딩으로 설정 되어 있기 때문이다. 그렇기 때문에 team은 proxy객체로 생성 되게 된것이다. 이러한 문제를 해결하기 위해서는 Fetch join이 필요하다.

<br><br>

# JPQL 페치 조인

```java
public interface MemberRepository extends JpaRepository<Member, Long> {

    @Query("select m from Member m join fetch m.team")
    List<Member> findMemberFetchJoin();

}

@SpringBootTest
@Transactional
@Rollback(false)
public class MemberRepositoryTest {

    @Autowired
    MemberRepository memberRepository;

    @Autowired
    TeamRepository teamRepository;

    @Test
    public void findMemberFetch() {
        //given
        //member1 -> teamA
        //member2 -> teamB

        Team teamA = new Team("teamA");
        Team teamB = new Team("teamB");
        teamRepository.save(teamA);
        teamRepository.save(teamB);

        Member member1 = new Member("member1", 10, teamA);
        Member member2 = new Member("member2", 10, teamB);
        memberRepository.save(member1);
        memberRepository.save(member2);

        em.flush();
        em.clear();

        //when
        //select Member
        List<Member> members = memberRepository.findMemberFetchJoin();

        for (Member member : members) {
            System.out.println("member = " + member);
            System.out.println("    teamClass = " + member.getTeam().getClass());
            System.out.println("    teamName = " + member.getTeam().getName());
        }
    }
}
```

- 결과

```
select
    member0_.member_id as member_i1_0_0_,
    team1_.team_id as team_id1_1_1_,
    member0_.age as age2_0_0_,
    member0_.team_id as team_id4_0_0_,
    member0_.username as username3_0_0_,
    team1_.name as name2_1_1_ 
from
    member member0_ 
inner join
    team team1_ 
        on member0_.team_id=team1_.team_id

member = Member(id=3, username=member1, age=10)
    teamClass = class study.datajpa.entity.Team
    teamName = teamA
member = Member(id=4, username=member2, age=10)
    teamClass = class study.datajpa.entity.Team
    teamName = teamB
```

위 결과와 같이 Member와 Team을 inner join으로 한번에 쿼리해온다. 그리고 teamClass도 proxy객체가 아닌 실제 Team Entity객체이다.

<br><br>

# EntityGraph

연관된 엔티티들을 SQL 한번에 조회하는 방법으로 Fetch join과 같다. 스프링 데이터 JPA는 JPA가 제공하는 엔티티 그래프 기능을 편리하게 사용하게 도와준다. 이 기능을
사용하면 JPQL 없이 페치 조인을 사용할 수 있다. (JPQL + 엔티티 그래프도 가능)

<br>

```java
public interface MemberRepository extends JpaRepository<Member, Long> {

    //공통 메서드 오버라이드
    @Override
    @EntityGraph(attributePaths = {"team"})
    List<Member> findAll();

    //JPQL + 엔티티 그래프
    @EntityGraph(attributePaths = {"team"})
    @Query("select m from Member m")
    List<Member> findMemberEntityGraph();

    //메서드 이름으로 쿼리에서 특히 편리하다.
    @EntityGraph(attributePaths = {"team"})
    List<Member> findByUsername(String username)
}

@SpringBootTest
@Transactional
@Rollback(false)
public class MemberRepositoryTest {

    @Autowired
    MemberRepository memberRepository;

    @Autowired
    TeamRepository teamRepository;

    @Test
    public void entityGraphTest() {
        //given
        //member1 -> teamA
        //member2 -> teamB

        Team teamA = new Team("teamA");
        Team teamB = new Team("teamB");
        teamRepository.save(teamA);
        teamRepository.save(teamB);

        Member member1 = new Member("member1", 10, teamA);
        Member member2 = new Member("member2", 10, teamB);
        memberRepository.save(member1);
        memberRepository.save(member2);

        em.flush();
        em.clear();

        //when
        //select Member
        List<Member> members = memberRepository.findAll();

        for (Member member : members) {
            System.out.println("member = " + member);
            System.out.println("    teamClass = " + member.getTeam().getClass());
            System.out.println("    teamName = " + member.getTeam().getName());
        }
    }
}
```

- 결과

```
select
    member0_.member_id as member_i1_0_0_,
    team1_.team_id as team_id1_1_1_,
    member0_.age as age2_0_0_,
    member0_.team_id as team_id4_0_0_,
    member0_.username as username3_0_0_,
    team1_.name as name2_1_1_ 
from
    member member0_ 
left outer join
    team team1_ 
        on member0_.team_id=team1_.team_id

member = Member(id=3, username=member1, age=10)
    teamClass = class study.datajpa.entity.Team
    teamName = teamA
member = Member(id=4, username=member2, age=10)
    teamClass = class study.datajpa.entity.Team
    teamName = teamB

```

위 결과와 같이 Member와 Team을 left outer join으로 한번에 쿼리해온다. 그리고 teamClass도 proxy객체가 아닌 실제 Team Entity객체이다. @EntityGraph 어노테이션 attributePaths 속성에 어떤 엔티티를 같이 가져올 것인지 설정해주면 된다.

<br><br><br>
## References 및 사진 출처

> __김영한의 실전! 스프링 데이터 JPA__