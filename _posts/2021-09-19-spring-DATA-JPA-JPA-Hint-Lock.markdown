---
layout: post
title:  "spring Data JPA Hint Lock"
subtitle:   "spring Data JPA Hint Lock"
date:   2021-09-19 00:01:27 +0900
categories: spring
tags: spring JPA ORM Mapping String-Data-JPA Hint Lock
comments: true
---


<br>

- 목차
	- [JPA Hint](#JPA Hint)
	- [JPA Lock](#JPA Lock)
    
<br>

# JPA Hint

JPA에게 쿼리 힌트를 제공한다.(SQL 힌트가 아니라 JPA 구현체에게 제공하는 힌트)

```java
public interface MemberRepository extends JpaRepository<Member, Long> {

    @QueryHints(value = @QueryHint(name = "org.hibernate.readOnly", value = "true"))
    Member findReadOnlyByUsername(String username);
}

@SpringBootTest
@Transactional
@Rollback(false)
public class MemberRepositoryTest {

    @Autowired
    MemberRepository memberRepository;

    @Autowired
    TeamRepository teamRepository;

    @PersistenceContext
    EntityManager em;

    @Test
    public void queryHint() {

        //given
        Member member1 = memberRepository.save(new Member("member1", 10));
        em.flush();
        em.clear();

        //when
        Member findMember = memberRepository.findReadOnlyByUsername("member1");
        findMember.setUsername("member2");

        em.flush(); //  update 쿼리 실행되지 않음
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
where
    member0_.username=?
```

위 결과 처럼 member1을 가지고 온후 member2로 데이터를 변경 하면 update 쿼리가 실행되어야 하지만 @QueryHints(value = @QueryHint(name = "org.hibernate.readOnly", value = "true")) 설정으로 ReadOnly가 되었기 때문에 Update 쿼리가 실행되지 않았다. 이렇게 ReadOnly로 설정하게 되면 JPA 내부적으로 스냅샷을 찍은후 Dirty Check(변경감지)를 통해 이전 스냅샷과 현재 스냅샷을 비교하는 메커니즘이 있는데, 이과정을 생략하므로 약간의 메모리적인 최적화를 할수 있다.

<br>

> 참고 : @QueryHint의 readOnly는 스냅샷을 만들지 않기 때문에, 메모리가 절약된다. <br>
그리고 @Transaction(readOnly=true)는 트랜잭션 커밋 시점에 flush를 하지 않기 때문에 이로 인한 dirty checking 비용이 들지 않는다. 따라서 cpu가 절약된다.<br> 스프링 5.1 버전 이후, @Transaction(readOnly=true)로 설정하면, @QueryHint의 readOnly까지 모두 동작한다고 한다.

<br><br>

# JPA Lock

```java
public interface MemberRepository extends JpaRepository<Member, Long> {

    @Lock(LockModeType.PESSIMISTIC_WRITE)
    List<Member> findLockByUsername(String username);
}

@SpringBootTest
@Transactional
@Rollback(false)
public class MemberRepositoryTest {

    @Autowired
    MemberRepository memberRepository;

    @Autowired
    TeamRepository teamRepository;

    @PersistenceContext
    EntityManager em;

    @Test
    public void queryLock() {

        //given
        Member member1 = memberRepository.save(new Member("member1", 10));
        em.flush();
        em.clear();

        //when
        List<Member> findMember = memberRepository.findLockByUsername("member1");
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
where
    member0_.username=? for update
```

위 결과를 보면 DB Lock을 위한 for update를 확인 할수 있다. @Lock 어노테이션을 통해 설정 가능하다.

<br><br><br>
## References 및 사진 출처

> __김영한의 실전! 스프링 데이터 JPA__