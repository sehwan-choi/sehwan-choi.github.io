---
layout: post
title:  "spring JPA JQPL 벌크연산"
subtitle:   "spring JPA JPQL 벌크연산"
date:   2021-09-09 17:16:27 +0900
categories: spring
tags: spring JPA ORM Mapping JPQL 벌크연산
comments: true
---


<br>

- 목차
	- [벌크연산](#벌크연산)
	- [벌크 연산 예제](#벌크-연산-예제)
	- [벌크 연산 주의](#벌크-연산-주의)
    
<br>

# 벌크연산

- 재고가 10개 미만인 모든 상품의 가격을 10% 상승하려면? 
- JPA 변경 감지 기능으로 실행하려면 너무 많은 SQL 실행
    1. 재고가 10개 미만인 상품을 리스트로 조회한다. 
    2. 상품 엔티티의 가격을 10% 증가한다. 
    3. 트랜잭션 커밋 시점에 변경감지가 동작한다. 
- 변경된 데이터가 100건이라면 100번의 UPDATE SQL 실행

<br>

# 벌크 연산 예제

- 쿼리 한 번으로 여러 테이블 로우 변경(엔티티) 
- executeUpdate()의 결과는 영향받은 엔티티 수 반환
- UPDATE, DELETE 지원
- INSERT(insert into .. select, 하이버네이트 지원

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

public class BulkMain {

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

            /**
             * 벌크 연산
             */

            em.flush();
            em.clear();

            int resultCount = em.createQuery("update Member m set m.age = 20").executeUpdate();

            System.out.println("resultCount = " + resultCount);

            List<Member> findMember = em.createQuery("select m from Member m", Member.class).getResultList();

            for (Member member1 : findMember) {
                System.out.println("member1 = " + member1);
            }
        }
    }
}
```

- 결과

```
resultCount = 3
...
member1 = Member{id=3, type=null, username='회원1', age=20}
member1 = Member{id=4, type=null, username='회원2', age=20}
member1 = Member{id=5, type=null, username='회원3', age=20}
```

<br>

# 벌크 연산 주의

- 벌크 연산은 영속성 컨텍스트를 무시하고 데이터베이스에 직접 쿼리
- 장애를 피하고 싶다면 아래와 같이 사용
    - 벌크 연산을 먼저 실행할 것
    - 벌크 연산 수행 후 영속성 컨텍스트 초기화할 것

```java
public class BulkMain {

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

            // createQuery 하는 시점에 flush()가 발생한다.
            int resultCount = em.createQuery("update Member m set m.age = 20").executeUpdate();

            System.out.println("resultCount = " + resultCount);

            List<Member> findMember = em.createQuery("select m from Member m", Member.class).getResultList();

            for (Member member1 : findMember) {
                System.out.println("member1 = " + member1);
            }

            ...
        }
    }
}
```

- 결과

```
member1 = Member{id=3, type=null, username='회원1', age=0}
member1 = Member{id=4, type=null, username='회원2', age=0}
member1 = Member{id=5, type=null, username='회원3', age=0}
```

위 결과를 보면 age를 분명히 20으로 변경 하였지만 원래 값인 0이 나오게 된다. 왜냐하면 벌크 연산은 영속성 컨텍스트를 무시하고 데이터베이스에 직접 쿼리하기 때문이다. 이렇기 때문에 영속성 컨텐스트에는 update문이 반영 되지 않은 상태이다. 이 상태에서 잘못 사용할 경우 정합성이 맞지 않기때문에 장애로 이어질수 있다. <br>
반드시 clear()를 호출하여 영속성 컨텍스트를 비워준후에 사용하여야 한다.


```java
public class BulkMain {

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

            // createQuery 하는 시점에 flush()가 발생한다.
            int resultCount = em.createQuery("update Member m set m.age = 20").executeUpdate();

            System.out.println("resultCount = " + resultCount);

            em.clear(); //  영속성 컨텍스트를 초기화 해야한다.

            List<Member> findMember = em.createQuery("select m from Member m", Member.class).getResultList();

            for (Member member1 : findMember) {
                System.out.println("member1 = " + member1);
            }

            ...
        }
    }
}
```

- 결과

```
member1 = Member{id=3, type=null, username='회원1', age=20}
member1 = Member{id=4, type=null, username='회원2', age=20}
member1 = Member{id=5, type=null, username='회원3', age=20}
```

<br><br><br>
## References 및 사진 출처

> __김영한의 자바 ORM 표준 JPA 프로그래밍 - 기본편__