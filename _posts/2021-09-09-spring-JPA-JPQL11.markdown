---
layout: post
title:  "spring JPA JQPL Named Query"
subtitle:   "spring JPA JPQL Named Query"
date:   2021-09-09 15:16:27 +0900
categories: spring
tags: spring JPA ORM Mapping JPQL Named Query
comments: true
---


<br>

- 목차
	- [Named 쿼리](#Named 쿼리)
    
<br>

# Named 쿼리

- 미리 정의해서 이름을 부여해두고 사용하는 JPQL 
- 정적 쿼리
- 어노테이션, XML에 정의
- 애플리케이션 로딩 시점에 초기화 후 재사용
- 애플리케이션 로딩 시점에 쿼리를 검증

```java
@Entity
@NamedQuery(
        name = "Member.findByUsername", <- NamedQuery 이름
        query = "select m from Member m where m.username = :username" <- NamedQuery 실제 Query
)
@NamedQuery(
        name = "Member.count", <- NamedQuery 이름
        query = "select count(m) from Member m" <- NamedQuery 실제 Query
)
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

public class NamedQueryMain {

    public static void main(String[] args) {
        ...
        try {
            /**
             * Member.findByUsername 사용
            */
            List<Member> resultList = em.createNamedQuery("Member.findByUsername", Member.class)
                    .setParameter("username", "회원1")
                    .getResultList();

            for (Member member : resultList) {
                System.out.println("member = " + member);
            }

            /**
             * Member.count 사용
            */
            Long singleResult = em.createNamedQuery("Member.count", Long.class)
                    .getSingleResult();

            System.out.println("singleResult = " + singleResult);

        }
    }
}

```

<br>

# 애플리케이션 로딩 시점에 쿼리를 검증

```java
@Entity
@NamedQuery(
        name = "Member.findByUsername",
        query = "select m from Memberqqq m where m.username = :username"    <-- 쿼리문에서 Member가 아닌 Memberqqq로 변경했다
)
@NamedQuery(
        name = "Member.count",
        query = "select count(m) from Member m"
)
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
```

- 결과

```
Member.findByUsername failed because of: org.hibernate.hql.internal.ast.QuerySyntaxException: Memberqqq is not mapped [select m from Memberqqq m where m.username = :username]
```

우의 Exception이 발생한다.

<br><br><br>
## References 및 사진 출처

> __김영한의 자바 ORM 표준 JPA 프로그래밍 - 기본편__