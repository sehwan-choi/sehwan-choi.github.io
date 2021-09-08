---
layout: post
title:  "spring JPA JQPL Fetch join"
subtitle:   "spring JPA JPQL Fetch join"
date:   2021-09-08 23:50:27 +0900
categories: spring
tags: spring JPA ORM Mapping JPQL Fetch join
comments: true
---


<br>

- 목차
	- [Fetch Join](#fetch-join)
	- [엔티티 fetch join](#엔티티-fetch-join)
	- [컬렉션 fetch join](#컬렉션-fetch-join)
	- [fetch join과 DISTINCT](#fetch-join과-distinct)
	- [fetch join과 일반 join의 차이](#fetch-join과-일반-join의-차이)
	- [fetch join의 특징과 한계](#fetch-join의-특징과-한계)
	- [정리](#정리)
    
<br>

# Fetch Join

- SQL 조인 종류가 아님
- JPQL에서 성능 최적화를 위해 제공하는 기능
- 연관된 엔티티나 컬렉션을 SQL 한 번에 함께 조회하는 기능
- join fetch 명령어 사용
- 페치 조인 ::= [ LEFT [OUTER] | INNER ] JOIN FETCH 조인경로
    - select m from Member join fetch m.team

<br>

# 엔티티 fetch join

- 회원을 조회하면서 연관된 팀도 함께 조회(SQL 한 번에) 
- SQL을 보면 회원 뿐만 아니라 팀(T.*)도 함께 SELECT

```
[JPQL]
    - select m from Member m join fetch m.team 

[SQL]
    - SELECT M.*, T.* FROM MEMBER M INNER JOIN TEAM T ON M.TEAM_ID=T.ID
```

```java
@Entity
public class Member {

    @Id
    @GeneratedValue
    @Column(name = "MEMBER_ID")
    private Long id;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "TEAM_ID")
    private Team team;

    public void changeTeam(Team team){
        team.getMembers().add(this);
        this.setTeam(team);
    }

    @Enumerated(EnumType.STRING)
    private MemberType type;

    private String username;

    private int age;
    ...
}

@Entity
public class Team {

    @Id
    @GeneratedValue
    @Column(name = "TEAM_ID")
    private Long id;

    private String name;

    @OneToMany(mappedBy = "team")
    private List<Member> members = new ArrayList<>();
    ...
}

public class FetchJoinMain {

    public static void main(String[] args) {
        ...
        try {
            Team team = new Team();
            team.setName("TeamA");
            em.persist(team);

            Team teamB = new Team();
            teamB.setName("TeamB");
            em.persist(teamB);

            Member member = new Member();
            member.setUsername("회원1");

            member.addteam(team);
            em.persist(member);

            Member member2 = new Member();
            member2.setUsername("회원2");
            member2.addteam(team);
            em.persist(member2);

            Member member3 = new Member();
            member3.setUsername("회원3");
            member3.addteam(teamB);
            em.persist(member3);

            em.flush();
            em.clear();

            /**
             * 엔티티 Fetch Join
             * Fetch Join을 사용하게 되면 지연로딩이 발생하지 않는다.
             */
            String query = "select m from Member m join fetch m.team";
            List<Member> resultList = em.createQuery(query, Member.class).getResultList();

            for (Member member1 : resultList) {
                System.out.println("username = " + member1.getUsername() + " # teamname = " + member1.getTeam().getName());
            }
        }
        ...
    }
}
```

- 결과

```java
username = 회원1 # teamname = TeamA
username = 회원2 # teamname = TeamA
username = 회원3 # teamname = TeamB
```

![그림1](https://sehwan-choi.github.io/assets/img/spring/JPQL/jpa3.jpg)

<br>

# 컬렉션 fetch join

- 일대다 관계, 컬렉션 페치 조인

```
[JPQL]
select t
    from Team t join fetch t.members
    where t.name = ‘팀A' 

[SQL]
SELECT T.*, M.*
    FROM TEAM T
    INNER JOIN MEMBER M ON T.ID=M.TEAM_ID
    WHERE T.NAME = '팀A'
```

```java
public class FetchJoinMain {

    public static void main(String[] args) {
        ...
        try {
            Team team = new Team();
            team.setName("TeamA");
            em.persist(team);

            Team teamB = new Team();
            teamB.setName("TeamB");
            em.persist(teamB);

            Member member = new Member();
            member.setUsername("회원1");

            member.addteam(team);
            em.persist(member);

            Member member2 = new Member();
            member2.setUsername("회원2");
            member2.addteam(team);
            em.persist(member2);

            Member member3 = new Member();
            member3.setUsername("회원3");
            member3.addteam(teamB);
            em.persist(member3);

            em.flush();
            em.clear();

            /**
             * 컬렉션 Fetch Join(중복 제거 전)
             */
            String query = "select t from Team t join fetch t.members";
            List<Team> resultList = em.createQuery(query, Team.class).getResultList();
            System.out.println("size = " + resultList.size());

            for (Team team1 : resultList) {
                System.out.println("teamname = " + team1.getName());
                for(Member mem : team1.getMembers())
                {
                    System.out.println("    memberUsername = " + mem.getUsername());
                }
            }
        }
        ...
    }
}
```

- 결과

```
size = 3
teamname = TeamA
    memberUsername = 회원1
    memberUsername = 회원2
teamname = TeamA
    memberUsername = 회원1
    memberUsername = 회원2
teamname = TeamB
    memberUsername = 회원3
```

![그림1](https://sehwan-choi.github.io/assets/img/spring/JPQL/jpa4.jpg)

중복 제거가 되지않아 size가 3개가 나오게 된다.

<br>

# fetch join과 DISTINCT

- SQL의 DISTINCT는 중복된 결과를 제거하는 명령
- JPQL의 DISTINCT 2가지 기능 제공
1. SQL에 DISTINCT를 추가하여 중복 제거
2. 애플리케이션에서 엔티티 중복 제거

만일 1번 방법에서 중복제거가 되지 않는경우

```
select distinct t
from Team t join fetch t.members
where t.name = ‘팀A’
```

![그림1](https://sehwan-choi.github.io/assets/img/spring/JPQL/jpa5.jpg)

위 쿼리 결과로 나온 데이터가 정확히 일치하지 않는다면 중복제거가 되지 않을 것이다. <br>

이럴 경우를 대비하여 JPQL에서는 같은 식별자를 가진 엔티티를 제거한다.

![그림1](https://sehwan-choi.github.io/assets/img/spring/JPQL/jpa6.jpg)


```java
public class FetchJoinMain {

    public static void main(String[] args) {
        ...
        try {
            Team team = new Team();
            team.setName("TeamA");
            em.persist(team);

            Team teamB = new Team();
            teamB.setName("TeamB");
            em.persist(teamB);

            Member member = new Member();
            member.setUsername("회원1");

            member.addteam(team);
            em.persist(member);

            Member member2 = new Member();
            member2.setUsername("회원2");
            member2.addteam(team);
            em.persist(member2);

            Member member3 = new Member();
            member3.setUsername("회원3");
            member3.addteam(teamB);
            em.persist(member3);

            em.flush();
            em.clear();

            /**
             * 컬렉션 Fetch Join(중복 제거 후)
             */
            String query = "select distinct t from Team t join fetch t.members";
            List<Team> resultList = em.createQuery(query, Team.class).getResultList();
            System.out.println("size = " + resultList.size());

            for (Team team1 : resultList) {
                System.out.println("teamname = " + team1.getName());
                for(Member mem : team1.getMembers())
                {
                    System.out.println("    memberUsername = " + mem.getUsername());
                }
            }
        }
        ...
    }
}
```

- 결과

```
size = 2
teamname = TeamA
    memberUsername = 회원1
    memberUsername = 회원2
teamname = TeamB
    memberUsername = 회원3
```

<br>

# fetch join과 일반 join의 차이

- 일반 조인 실행시 연관된 엔티티를 함께 조회하지 않음(지연 로딩)

```
[JPQL]
select t
    from Team t join t.members m

[SQL]
SELECT T.*
    FROM TEAM T
    INNER JOIN MEMBER M ON T.ID=M.TEAM_ID 
```

<br>

- 페치 조인 실행시 연관된 엔티티도 함께 조회(즉시 로딩)

```
[JPQL]
select m 
    from Member m join fetch m.team 

[SQL]
SELECT M.*, T.* 
    FROM MEMBER M 
    INNER JOIN TEAM T ON M.TEAM_ID=T.ID
```

JPQL은 결과를 반환할 때 연관관계는 고려하지 않는다. 단지 SELECT 절에 지정한 엔티티만 조회를 한다. fetch join은 객체 그래프를 SQL 한번에 조회하는 개념이다.

<br>

# fetch join의 특징과 한계

- 페치 조인 대상에는 별칭을 줄 수 없다. 
- 하이버네이트는 가능, 가급적 사용하면 안된다.
- 둘 이상의 컬렉션은 페치 조인 할 수 없다. 
- 컬렉션을 페치 조인하면 페이징 API(setFirstResult, setMaxResults)를 사용할 수 없다. 
- 일대일, 다대일 같은 단일 값 연관 필드들은 페치 조인해도 페이징 가능
- 하이버네이트는 경고 로그를 남기고 메모리에서 페이징(매우 위험)

```java
public class FetchJoinMain {

    public static void main(String[] args) {
        ...
        try {
            Team team = new Team();
            team.setName("TeamA");
            em.persist(team);

            Team teamB = new Team();
            teamB.setName("TeamB");
            em.persist(teamB);

            Member member = new Member();
            member.setUsername("회원1");

            member.addteam(team);
            em.persist(member);

            Member member2 = new Member();
            member2.setUsername("회원2");
            member2.addteam(team);
            em.persist(member2);

            Member member3 = new Member();
            member3.setUsername("회원3");
            member3.addteam(teamB);
            em.persist(member3);

            em.flush();
            em.clear();

            /**
             * 컬렉션 Fetch Join 페이징 사용(위험)
             * 일대다인 경우
             * 위험한 이유 : 쿼리문을 보면 페이징 쿼리가 나가지않음, 모든 데이터를 가져오게됨
             */
            String query = "select t from Team t join fetch t.members";
            List<Team> resultList = em.createQuery(query, Team.class)
                    .setFirstResult(0)
                    .setMaxResults(1)
                    .getResultList();
            System.out.println("size = " + resultList.size());

            for (Team team1 : resultList) {
                System.out.println("teamname = " + team1.getName());
                for(Member mem : team1.getMembers())
                {
                    System.out.println("    memberUsername = " + mem.getUsername());
                }
            }
        }
        ...
    }
}
```

- 결과

```
...
WARN org.hibernate.hql.internal.ast.QueryTranslatorImpl - HHH000104: firstResult/maxResults specified with collection fetch; applying in memory! <- 경고 문구가 출력된다.
...
select
    team0_.TEAM_ID as team_id1_4_0_,
    members1_.MEMBER_ID as member_i1_1_1_,
    team0_.name as name2_4_0_,
    members1_.age as age2_1_1_,
    members1_.TEAM_ID as team_id5_1_1_,
    members1_.type as type3_1_1_,
    members1_.username as username4_1_1_,
    members1_.TEAM_ID as team_id5_1_0__,
    members1_.MEMBER_ID as member_i1_1_0__ 
from
    Team team0_ 
inner join
    Member members1_ 
        on team0_.TEAM_ID=members1_.TEAM_ID
```

결과에 쿼리문과 같이 페이징이 되지않고 모든 데이터를 가져온다. 데이터가 몇개 없다면 상관없지만 양이 많다면 장애로 이어지기 때문에 반드시 조심해야 한다.

<br>

- 연관된 엔티티들을 SQL 한 번으로 조회 - 성능 최적화
- 엔티티에 직접 적용하는 글로벌 로딩 전략보다 우선한다.
    - @OneToMany(fetch = FetchType.LAZY) 지연로딩으로 설정 되어있어도 fetch join을 사용하는 경우 즉시로딩으로 데이터를 가져온다.
- 실무에서 글로벌 로딩 전략은 모두 지연 로딩으로 설정 한다.
- 최적화가 필요한 곳은 페치 조인 적용한다.

<br>

# 정리

- 모든 것을 페치 조인으로 해결할 수 는 없음
- 페치 조인은 객체 그래프를 유지할 때 사용하면 효과적
- 여러 테이블을 조인해서 엔티티가 가진 모양이 아닌 전혀 다른 결과를 내야 하면, 페치 조인 보다는 일반 조인을 사용하고 필요한 데이터들만 조회해서 DTO로 반환하는 것이 효과적

<br><br><br>
## References 및 사진 출처

> __김영한의 자바 ORM 표준 JPA 프로그래밍 - 기본편__