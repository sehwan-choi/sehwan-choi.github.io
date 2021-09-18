---
layout: post
title:  "spring Data JPA 사용자 정의 리포지토리 구현"
subtitle:   "spring Data JPA 사용자 정의 리포지토리 구현"
date:   2021-09-19 00:01:27 +0900
categories: spring
tags: spring JPA ORM Mapping String-Data-JPA 사용자 정의 리포지토리 구현
comments: true
---


<br>

- 목차
	- [사용자 정의 리포지토리 구현](#사용자-정의-리포지토리-구현)
    
<br>

# 사용자 정의 리포지토리 구현

- 스프링 데이터 JPA 리포지토리는 인터페이스만 정의하고 구현체는 스프링이 자동 생성한다.
- 스프링 데이터 JPA가 제공하는 인터페이스를 직접 구현하면 구현해야 하는 기능이 너무 많다.
- 다양한 이유로 인터페이스의 메서드를 직접 구현하고 싶다면?
    - JPA 직접 사용( EntityManager )
    - 스프링 JDBC Template 사용
    - MyBatis 사용
    - 데이터베이스 커넥션 직접 사용 등등...
    - Querydsl 사용

```java
// 사용자 정의 인터페이스
public interface MemberRepositoryCustom {

    List<Member> findMemberCustom();
}

// 사용자 정의 인터페이스 상속
public interface MemberRepository extends JpaRepository<Member, Long>, MemberRepositoryCustom {

    ...

}

// 사용자 정의 인터페이스 구현 클래스
@RequiredArgsConstructor
public class MemberRepositoryImpl implements MemberRepositoryCustom{

    private final EntityManager em;

    @Override
    public List<Member> findMemberCustom() {
        return em.createQuery("select m from Member m")
                .getResultList();
    }
}

// 사용자 정의 메서드 호출 코드
@SpringBootTest
@Transactional
@Rollback(false)
public class MemberRepositoryTest {

    @Autowired
    MemberRepository memberRepository;

    @Test
    public void callCustom() {
        List<Member> result = memberRepository.findMemberCustom();
    }
}
```

위와 같이 memberRepository.findMemberCustom(); 호출하게 되면 JPA에서 MemberRepositoryImpl에 findMemberCustom()를 호출해준다. 대신 여기에는 규칙이 있다. 리포지토리 인터페이스 이름 + Impl(MemberRepositoryImpl = MemberRepository + Impl) 또는 사용자 정의 인터페이스 명 + Impl(MemberRepositoryCustomImpl = MemberRepositoryCustom + Impl) 으로 생성해야한다. 그래야 스프링 데이터 JPA가 인식해서 스프링 빈으로 등록한다.

<br><br><br>
## References 및 사진 출처

> __김영한의 실전! 스프링 데이터 JPA__