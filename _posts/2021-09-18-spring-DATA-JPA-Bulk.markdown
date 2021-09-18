---
layout: post
title:  "spring Data JPA Bulk"
subtitle:   "spring Data JPA Bulk"
date:   2021-09-18 20:25:27 +0900
categories: spring
tags: spring JPA ORM Mapping String-Data-JPA Bulk
comments: true
---


<br>

- 목차
	- [JPA를 사용한 벌크성 수정 쿼리](#jpa를-사용한-벌크성-수정-쿼리)
	- [스프링 데이터 JPA를 사용한 벌크성 수정 쿼리](#스프링-데이터-jpa를-사용한-벌크성-수정-쿼리)
	- [벌크성 수정, 삭제 쿼리시 주의할점](#벌크성-수정-삭제-쿼리시-주의할점)
	- [정리](#정리)
    
<br>

# JPA를 사용한 벌크성 수정 쿼리

<br>

```java
@Repository
@RequiredArgsConstructor
public class MemberJpaRepository {

    private final EntityManager em;

    public int bulkAgePlus(int age) {
        return em.createQuery("update Member m set m.age = m.age + 1 where m.age >= :age")
                .setParameter("age", age)
                .executeUpdate();
    }
}

@SpringBootTest
@Transactional
@Rollback(false)
public class MemberJpaRepositoryTest {

    @Autowired
    MemberJpaRepository memberJpaRepository;

    @Test
    public void bulkUpdate() {
        //given
        memberJpaRepository.save(new Member("member1", 10));
        memberJpaRepository.save(new Member("member2", 19));
        memberJpaRepository.save(new Member("member3", 20));
        memberJpaRepository.save(new Member("member4", 21));
        memberJpaRepository.save(new Member("member5", 40));

        //when
        int resultCount = memberJpaRepository.bulkAgePlus(20);

        //then
        Assertions.assertThat(resultCount).isEqualTo(3);
    }
}
```

- 결과

```
update
    member 
set
    age=age+1 
where
    age>=20
```

<br><br>

# 스프링 데이터 JPA를 사용한 벌크성 수정 쿼리

<br>

```java
public interface MemberRepository extends JpaRepository<Member, Long> {

    @Modifying(clearAutomatically = true)
    @Query("update Member m set m.age = m.age + 1 where m.age >= :age")
    int bulkAgePlus(@Param("age") int age);

}

@SpringBootTest
@Transactional
@Rollback(false)
public class MemberRepositoryTest {

    @Autowired
    MemberRepository memberRepository;

    @Test
    public void bulkUpdate() {
        //given
        memberRepository.save(new Member("member1", 10));
        memberRepository.save(new Member("member2", 19));
        memberRepository.save(new Member("member3", 20));
        memberRepository.save(new Member("member4", 21));
        memberRepository.save(new Member("member5", 40));

        //when
        int resultCount = memberRepository.bulkAgePlus(20);

        //then
        Assertions.assertThat(resultCount).isEqualTo(3);
    }
}
```

- 결과

```
update
    member 
set
    age=age+1 
where
    age>=20
```

MemberRepository에서 @Modiffing 어노테이션 꼭 설정해야 update 문이라는 것을 명시해야한다. 하지 않으면 아래 예외 발생한다.

```
[org.springframework.dao.InvalidDataAccessApiUsageException: org.hibernate.hql.internal.QueryExecutionRequestException: Not supported for DML operations]
```

<br>

# 벌크성 수정, 삭제 쿼리시 주의할점

<br>

만일 벌크성 수정후 데이터를 조회한다면?

```java
public interface MemberRepository extends JpaRepository<Member, Long> {

    @Modifying(clearAutomatically = true)
    @Query("update Member m set m.age = m.age + 1 where m.age >= :age")
    int bulkAgePlus(@Param("age") int age);

}

@SpringBootTest
@Transactional
@Rollback(false)
public class MemberRepositoryTest {

    @Autowired
    MemberRepository memberRepository;

    @Test
    public void bulkUpdateWarn() {
        //given
        memberRepository.save(new Member("member1", 10));
        memberRepository.save(new Member("member2", 19));
        memberRepository.save(new Member("member3", 20));
        memberRepository.save(new Member("member4", 21));
        memberRepository.save(new Member("member5", 40));

        //when
        int resultCount = memberRepository.bulkAgePlus(20);

        List<Member> member5 = memberRepository.findByUsername("member5");
        for (Member member : member5) {
            System.out.println("member = " + member);
        }

        //then
        Assertions.assertThat(resultCount).isEqualTo(3);
    }
}
```

- 결과

```
update
    member 
set
    age=age+1 
where
    age>=20

member = Member(id=5, username=member5, age=40)
```

위 결과를 보면 이상한점이 있다. 분형이 member5의 age는 +1이 되어 41로 나와야 하는데, 여전히 40으로 나오는 것을 볼수 있다. 왜냐하면 bulk성 수정 쿼리는 영속성 컨텍스트를 거치지 않고 바로 DB로 쿼리가 실행되기 때문에 영속성 컨텍스트에는 이전 값이 남아 있는것 이다. 이런 상황을 피하기 위해서는 벌크성 수정쿼리 이후 영속성 컨텍스트를 초기화 해줘야 한다.

```java
public interface MemberRepository extends JpaRepository<Member, Long> {

    @Modifying(clearAutomatically = true)
    @Query("update Member m set m.age = m.age + 1 where m.age >= :age")
    int bulkAgePlus(@Param("age") int age);
}

    /**
     * 벌크연산후 영속성 컨텍스트를 비우지 않으면 기존데이터를 가져오는 문제가 발생한다.
     *
     * @Modifying(clearAutomatically = true) 를 했을때와 안 했을때 비교를 하면 알 수있다.
     */
    @Test
    public void bulkUpdateWarn() {
        //given
        memberRepository.save(new Member("member1", 10));
        memberRepository.save(new Member("member2", 19));
        memberRepository.save(new Member("member3", 20));
        memberRepository.save(new Member("member4", 21));
        memberRepository.save(new Member("member5", 40));

        //when
        int resultCount = memberRepository.bulkAgePlus(20);

        List<Member> member5 = memberRepository.findByUsername("member5");
        for (Member member : member5) {
            System.out.println("member = " + member);
        }

        //then
        Assertions.assertThat(resultCount).isEqualTo(3);
    }
}
```

- 결과

```
update
    member 
set
    age=age+1 
where
    age>=?

member = Member(id=5, username=member5, age=41)  
```

@Modifying(clearAutomatically = true)에서 clearAutomatically = true 옵션은 벌크성 수정 쿼리 실행후 영속성 컨텍스트에 남아있던 데이터를 DB에 반영후 영속성 컨텍스트를 clear한다. 즉, em.flush() 후 em.clear()하는 것 과 같다.

<br>

# 정리

- 벌크성 수정, 삭제 쿼리는 @Modifying 어노테이션을 사용
- 사용하지 않으면 다음 예외 발생

```
org.hibernate.hql.internal.QueryExecutionRequestException: Not supported for 
DML operations
```

- 벌크성 쿼리를 실행하고 나서 영속성 컨텍스트 초기화: @Modifying(clearAutomatically = true) (이 옵션의 기본값은 false )
- 이 옵션 없이 회원을 findById 로 다시 조회하면 영속성 컨텍스트에 과거 값이 남아서 문제가 될 수 있다. 만약 다시 조회해야 하면 꼭 영속성 컨텍스트를 초기화 하자.

<br>

> 참고: 벌크 연산은 영속성 컨텍스트를 무시하고 실행하기 때문에, 영속성 컨텍스트에 있는 엔티티의 상태와 DB에 엔티티 상태가 달라질 수 있다. <br><br>
> 권장하는 방안
> 1. 영속성 컨텍스트에 엔티티가 없는 상태에서 벌크 연산을 먼저 실행한다.
> 2. 부득이하게 영속성 컨텍스트에 엔티티가 있으면 벌크 연산 직후 영속성 컨텍스트를 초기화 한다.

<br><br><br>
## References 및 사진 출처

> __김영한의 실전! 스프링 데이터 JPA__