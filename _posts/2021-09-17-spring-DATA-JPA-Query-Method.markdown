---
layout: post
title:  "spring Data JPA Named Query"
subtitle:   "spring Data JPA Named Query"
date:   2021-09-18 12:57:27 +0900
categories: spring
tags: spring JPA ORM Mapping String-Data-JPA Named Query
comments: true
---


<br>

- 목차
	- [JPA NamedQuery](#JPA NamedQuery)
    
<br>

# JPA NamedQuery

JPA의 NamedQuery를 호출할 수 있다.

```java
@Entity
@Getter @Setter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
@ToString(of = {"id","username","age"})
@NamedQuery(
        name="Member.findByUsername",
        query ="select m from Member m where m.username = :username"
)
public class Member {

    @Id
    @GeneratedValue
    @Column(name = "MEMBER_ID")
    private Long id;

    private String username;

    private int age;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "TEAM_ID")
    private Team team;
    ...
}
```

위와 같이 @NamedQuery 어노테이션으로 Named 쿼리 정의한다.

<br>

- JPA를 직접 사용해서 Named 쿼리 호출

```java
@Repository
@RequiredArgsConstructor
public class MemberJpaRepository {

    private final EntityManager em;
    public List<Member> findByUsername(String username) {
        return em.createNamedQuery("Member.findByUsername")
                .setParameter("username",username)
                .getResultList();
    }
}

@SpringBootTest
@Transactional
@Rollback(false)
class MemberJpaRepositoryTest {

    @Autowired
    MemberJpaRepository memberJpaRepository;
    
    @Test
    public void testNamedQuery() {
        Member m1 = new Member("AAA", 10);
        Member m2 = new Member("AAA", 20);

        memberJpaRepository.save(m1);
        memberJpaRepository.save(m2);

        List<Member> aaa = memberJpaRepository.findByUsername("AAA");
        Member findMember = aaa.get(0);
        Assertions.assertThat(findMember).isEqualTo(m1);
    }
}
```

<br>

- 스프링 데이터 JPA로 NamedQuery 사용

```java
public interface MemberRepository extends JpaRepository<Member, Long> {

      @Query(name = "Member.findByUsername")
      List<Member> findByUsername(@Param("username") String username);
}

@SpringBootTest
@Transactional
@Rollback(false)
public class MemberRepositoryTest {

    @Autowired
    MemberRepository memberRepository;

    @Test
    public void testNamedQuery() {

        Member m1 = new Member("AAA", 10);
        Member m2 = new Member("AAA", 20);

        memberRepository.save(m1);
        memberRepository.save(m2);

        List<Member> aaa = memberRepository.findByUsername("AAA");
        Member findMember = aaa.get(0);
        Assertions.assertThat(findMember).isEqualTo(m1);
    }
}
```

- 스프링 데이터 JPA는 선언한 "도메인 클래스 + .(점) + 메서드 이름"으로 Named 쿼리를 찾아서 실행한다. JpaRepository<Member, Long> 에 선언한 도메인 클래스 Member와 메소드 이름으로 제일먼저 NamedQuery가 있는지 확인한다. 위 메소드 실행시 Member.findByUsername 가 된다. 이런경우 @Query 어노테이션이 없어도 정상 실행된다. <br>
 만일 Member.class에 NamedQuery에 설정된 name이 다른경우는 @Query로 name을 잡아 줘야 정상 실행이 된다.
- 만약 실행할 Named 쿼리가 없으면 메서드 이름으로 쿼리 생성 전략을 사용한다. 
- 필요하면 아래 링크를 참고하여 전략을 변경할 수 있지만 권장하지 않는다.

> [https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#repositories.query-methods.query-lookup-strategies](https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#repositories.query-methods.query-lookup-strategies)


<br><br><br>
## References 및 사진 출처

> __김영한의 실전! 스프링 데이터 JPA__