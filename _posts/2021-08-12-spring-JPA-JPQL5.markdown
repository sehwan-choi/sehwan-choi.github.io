---
layout: post
title:  "spring JPA JPQL 조건식 Case"
subtitle:   "spring JPA JPQL 조건식 Case"
date:   2021-09-01 10:30:27 +0900
categories: spring
tags: spring JPA ORM Mapping JPQL Case
comments: true
---


<br>

- 목차
	- [조건식 - CASE 식](#조건식---case-식)
	- [기본 CASE 식](#기본-case-식)
	- [단순 CASE 식](#단순-case-식)
	- [조건식 - COALESCE](#조건식---coalesce)
	- [조건식 - NULLIF](#조건식---nullif)
    
<br>

# 조건식 - CASE 식

<br>

# 기본 CASE 식

```
select
    case when m.age <= 10 then '학생요금'
         when m.age >= 60 then '경로요금'
         else '일반요금'
    end
from Member m
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

    public void addTeam(Team team){
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

public class CaseMain {
    public static void main(String[] args) {
        ...

        Member member = new Member();
        member.setAge(10);
        member.setUsername("Member");
        member.setType(MemberType.ADMIN);

        Team team = new Team();
        team.setName("Team");
        member.addteam(team);

        em.persist(team);
        em.persist(member);
        
        em.flush();
        em.clear();

        // 기본 Case 식
        String query = "select " +
                            "case when m.age <= 10 then '학생요금' " +
                                    "when m.age >= 60 then '경로요금' " +
                                    "else '일반요금' " +
                                    "end " +
                " from Member m";

        List<String> resultList = em.createQuery(query, String.class).getResultList();
        for (String s : resultList) {
            System.out.println("s = " + s);
        }
        ...
    }
}
```

- 결과

```
s = 학생요금
```

<br>

# 단순 CASE 식

```
select
    case t.name 
         when 'Team' then '110%'
         when 'TeamB' then '120%'
         else '105%'
    end
from Team t
```

```java

public class CaseMain {
    public static void main(String[] args) {
        ...
        Member member = new Member();
        member.setAge(10);
        member.setUsername("Member");
        member.setType(MemberType.ADMIN);

        Team team = new Team();
        team.setName("Team");
        member.addteam(team);

        em.persist(team);
        em.persist(member);
        
        em.flush();
        em.clear();

        // 단순 Case 식
        String query = "select " +
                            "case t.name " +
                                    "when 'Team' then '110%' " +
                                    "when 'TeamB' then '120%' " +
                                    "else '105%' " +
                                    "end " +
                " from Team t";

        List<String> resultList = em.createQuery(query, String.class).getResultList();
        for (String s : resultList) {
            System.out.println("s = " + s);
        }
        ...
    }
}
```

- 결과

```
s = 110%
```

<br>

# 조건식 - COALESCE

coalesce(조건 A, 조건 A 가 null인경우 표출할 내용) 으로 사용되며 조건 A가 null이게 된다면 두번째 인자로 넘긴 내용이 표출된다.

```java
public class CaseMain {
    public static void main(String[] args) {
        ...
        // 조건 Case 식(coalesce)
        Member nullMember = new Member();
        nullMember.setAge(10);
        nullMember.setUsername(null);
        nullMember.setType(MemberType.ADMIN);
        em.persist(nullMember);

        String query = "select coalesce(m.username, '이름 없는 회원') from Member m";

        List<String> resultList = em.createQuery(query, String.class).getResultList();
        for (String s : resultList) {
            System.out.println("s = " + s);
        }
        ...
    }
}
```

- 결과

```
s = 이름 없는 회원
```

위 결과와 같이 사용자 이름이 없으면 coalesce 두번째 인자인 '이름 없는 회원'을 반환한다.

<br>

# 조건식 - NULLIF

nullif(조건 A, 조건 B) 으로 사용되며 조건 A, 조건 B 두 값이 같으면 null 반환, 다르면 첫번째 값(조건 A)을 반환한다.

```java
public class CaseMain {
    public static void main(String[] args) {
        ...
        // 조건 Case 식(nullif)
        Member nullMember = new Member();
        nullMember.setAge(10);
        nullMember.setUsername("관리자");
        nullMember.setType(MemberType.ADMIN);
        em.persist(nullMember);

        String query = "select nullif(m.username, '관리자') from Member m";

        List<String> resultList = em.createQuery(query, String.class).getResultList();
        for (String s : resultList) {
            System.out.println("s = " + s);
        }
        ...
    }
}
```

- 결과

```
s = null
```

위 결과와 같이 nullif의 m.username과 '관리자'가 같기 때문에 null이 표출되는 것을 볼 수 있다.

<br>

<br><br><br>
## References 및 사진 출처

> __김영한의 자바 ORM 표준 JPA 프로그래밍 - 기본편__