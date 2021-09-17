---
layout: post
title:  "spring Data JPA Query"
subtitle:   "spring Data JPA Query"
date:   2021-09-18 04:30:27 +0900
categories: spring
tags: spring JPA ORM Mapping String-Data-JPA Query
comments: true
---


<br>

- 목차
	- [@Query, 리포지토리 메소드에 쿼리 정의하기](#query-리포지토리-메소드에-쿼리-정의하기)
	- [메서드에 JPQL 쿼리 작성](#메서드에-jpql-쿼리-작성)
	- [단순히 값 하나를 조회](#단순히-값-하나를-조회)
	- [DTO로 직접 조회](#dto로-직접-조회)
	- [파라미터 바인딩](#파라미터-바인딩)
	- [컬렉션 파라미터 바인딩](#컬렉션-파라미터-바인딩)
	- [반환 타입](#반환-타입)
	    - [조회 결과가 많거나 없으면?](#조회-결과가-많거나-없으면)
    
<br>

# @Query, 리포지토리 메소드에 쿼리 정의하기

<br>

# 메서드에 JPQL 쿼리 작성

```java

public interface MemberRepository extends JpaRepository<Member, Long> {

    @Query("select m from Member m where m.username = :username and m.age = :age")
    List<Member> findUser(@Param("username") String username, @Param("age") int age);
}

@SpringBootTest
@Transactional
@Rollback(false)
public class MemberRepositoryTest {

    @Autowired
    MemberRepository memberRepository;

    @Test
    public void testQuery() {

        Member m1 = new Member("AAA", 10);
        Member m2 = new Member("AAA", 20);

        memberRepository.save(m1);
        memberRepository.save(m2);

        List<Member> aaa = memberRepository.findUser("AAA",10);
        Member findMember = aaa.get(0);
        Assertions.assertThat(findMember).isEqualTo(m1);
    }
}
```

- @org.springframework.data.jpa.repository.Query 어노테이션을 사용
- 실행할 메서드에 정적 쿼리를 직접 작성하므로 이름 없는 Named 쿼리라 할 수 있음
- JPA Named 쿼리처럼 애플리케이션 실행 시점에 문법 오류를 발견할 수 있음(매우 큰 장점!)

```java
public interface MemberRepository extends JpaRepository<Member, Long> {
    
    @Query("select m from Member m where m.usefwefrname = :username and m.age = :age")
    List<Member> findUser(@Param("username") String username, @Param("age") int age);
}
```

만일 위와 같이 Member Entity에 username가 아닌 다른 값을 넣게 되면 실행 시점에 아래와 같이 찾을수 없다는 에러가 발생된다.

```
Caused by: java.lang.IllegalArgumentException: org.hibernate.QueryException: could not resolve property: usernaasdfasdfme of: study.datajpa.entity.Member [select m from study.datajpa.entity.Member m where m.usernaasdfasdfme = :username and m.age = :age]
```


> 참고: 실무에서는 메소드 이름으로 쿼리 생성 기능은 파라미터가 증가하면 메서드 이름이 매우 지저분해진다. 따라서 @Query 기능을 자주 사용하게 된다.

<br><br>

# 단순히 값 하나를 조회

```java
public interface MemberRepository extends JpaRepository<Member, Long> {

    @Query("select m.username from Member m")
    List<String> findUsernameList();
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
    public void findUsernameList() {

        Member m1 = new Member("AAA", 10);
        Member m2 = new Member("BBB", 20);

        memberRepository.save(m1);
        memberRepository.save(m2);

        List<String> usernameList = memberRepository.findUsernameList();
        for (String s : usernameList) {
            System.out.println("s = " + s);
        }
    }
}
```

- 결과

```
select
    member0_.username as col_0_0_ 
from
    member member0_

s = AAA
s = BBB
```

<br>

# DTO로 직접 조회

```java
public interface MemberRepository extends JpaRepository<Member, Long> {

    @Query("select new study.datajpa.dto.MemberDto(m.id, m.username, t.name) from Member m join m.team t")
    List<MemberDto> findMemberDto();
}

@Data
public class MemberDto {

    private Long id;
    private String username;
    private String teamName;

    public MemberDto(Long id, String username, String teamName) {
        this.id = id;
        this.username = username;
        this.teamName = teamName;
    }
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
    public void findMemberDto() {

        Team team = new Team("teamA");
        teamRepository.save(team);

        Member m1 = new Member("AAA", 10);
        m1.changeTeam(team);

        memberRepository.save(m1);

        List<MemberDto> memberDto = memberRepository.findMemberDto();
        for (MemberDto dto : memberDto) {
            System.out.println("dto = " + dto);
        }
    }
}
```

- 결과

```
select
    member0_.member_id as col_0_0_,
    member0_.username as col_1_0_,
    team1_.name as col_2_0_ 
from
    member member0_ 
inner join
    team team1_ 
        on member0_.team_id=team1_.team_id

dto = MemberDto(id=2, username=AAA, teamName=teamA)
```

> 주의! DTO로 직접 조회 하려면 JPA의 new 명령어를 사용해야 한다. 그리고 다음과 같이 생성자가 맞는 DTO가 필요하다. (JPA와 사용방식이 동일하다.)

<br><br>

# 파라미터 바인딩

위치 기반, 이름 기반 두가지로 바인딩 할 수 있다.

```sql
select m from Member m where m.username = ?0 //위치 기반

select m from Member m where m.username = :name //이름 기반
```

- 예제

```java

public interface MemberRepository extends JpaRepository<Member, Long> {

    @Query("select m from Member m where m.username = :username and m.age = :age")
    List<Member> findUser(@Param("username") String username, @Param("age") int age);
}

@SpringBootTest
@Transactional
@Rollback(false)
public class MemberRepositoryTest {

    @Autowired
    MemberRepository memberRepository;

    @Test
    public void testQuery() {

        Member m1 = new Member("AAA", 10);
        Member m2 = new Member("AAA", 20);

        memberRepository.save(m1);
        memberRepository.save(m2);

        List<Member> aaa = memberRepository.findUser("AAA",10);
        Member findMember = aaa.get(0);
        Assertions.assertThat(findMember).isEqualTo(m1);
    }
}
```

> 참고: 코드 가독성과 유지보수를 위해 이름 기반 파라미터 바인딩을 사용하자 (위치기반은 순서 실수를 하게 되면 장애로 이어질수 있다.)

<br><br>

# 컬렉션 파라미터 바인딩

Collection 타입으로 in절 지원한다.

```java
public interface MemberRepository extends JpaRepository<Member, Long> {

    @Query("select m from Member m where m.username in :names")
    List<Member> findByNames(@Param("names") Collection<String> names);
}


@SpringBootTest
@Transactional
@Rollback(false)
public class MemberRepositoryTest {

    @Autowired
    MemberRepository memberRepository;

    @Test
    public void findByNames() {

        Member m1 = new Member("AAA", 10);
        Member m2 = new Member("BBB", 20);

        memberRepository.save(m1);
        memberRepository.save(m2);

        
        List<String> usernameList = memberRepository.findUsernameList();
        List<Member> result = memberRepository.findByNames(usernameList);

        // 위의 방법, 아래방법 둘다 사용 가능하다.
        //List<Member> result = memberRepository.findByNames(Arrays.asList("AAA", "BBB"));
        for (Member member : result) {
            System.out.println("member = " + member);
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
where
    member0_.username in (
        ? , ?
    )
```

위 결과와 같이 컬렉션 타입으로 파라미터값을 넘겨도 정상적으로 쿼리가 실행된다.

<br><br>

# 반환 타입

스프링 데이터 JPA는 유연한 반환 타입 지원한다.

```java
public interface MemberRepository extends JpaRepository<Member, Long> {

    List<Member> findListByUsername(String username);   //  컬렉션
    Member findMemberByUsername(String username);       //  단건
    Optional<Member> findOptionalByUsername(String username);       //  단건
}

@SpringBootTest
@Transactional
@Rollback(false)
public class MemberRepositoryTest {

    @Autowired
    MemberRepository memberRepository;

    @Test
    public void returnType() {

        Member m1 = new Member("AAA", 10);
        Member m2 = new Member("BBB", 20);

        memberRepository.save(m1);
        memberRepository.save(m2);

        // 주의
        // 결과가 없으면 EmptyCollection을 반환하기 때문에 findMember 는 null이 아니다..
        List<Member> findMember = memberRepository.findListByUsername("AAA");
        for (Member member : findMember) {
            System.out.println("member = " + member);
        }

        // 주의
        // 결과가 없으면 null을 반환한다.
        Member findMember2 = memberRepository.findMemberByUsername("AAA");
        System.out.println("findMember2 = " + findMember2);

        Optional<Member> findMember3 = memberRepository.findOptionalByUsername("AAA");
        System.out.println("findMember3 = " + findMember3.get());
    }

}
```

- 반환타입은 아래의 공식문서에서 확인 할 수 있다.

> [https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#repository-query-return-types](https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#repository-query-return-types)

<br>

## 조회 결과가 많거나 없으면?
- 컬렉션
    - 결과 없음: 빈 컬렉션 반환

```java
public interface MemberRepository extends JpaRepository<Member, Long> {

    List<Member> findListByUsername(String username);   //  컬렉션
}

@SpringBootTest
@Transactional
@Rollback(false)
public class MemberRepositoryTest {

    @Autowired
    MemberRepository memberRepository;

    @Test
    public void returnType() {

        Member m1 = new Member("AAA", 10);
        Member m2 = new Member("BBB", 20);

        memberRepository.save(m1);
        memberRepository.save(m2);

        List<Member> findMember = memberRepository.findListByUsername("AAA");
        System.out.println("findMember = " + findMember.getClass());
        System.out.println("size = " + findMember.size());
    }

}
```

- 결과

```
findMember = class java.util.ArrayList
size = 0
```

<br>

- 단건 조회
    - 결과 없음: null 반환
    - 결과가 2건 이상: javax.persistence.NonUniqueResultException 예외 발생

```java
public interface MemberRepository extends JpaRepository<Member, Long> {

    Optional<Member> findOptionalByUsername(String username);       //  단건
}

@SpringBootTest
@Transactional
@Rollback(false)
public class MemberRepositoryTest {

    @Autowired
    MemberRepository memberRepository;

    @Test
    public void returnType() {

        Member m1 = new Member("AAA", 10);
        Member m2 = new Member("BBB", 20);

        memberRepository.save(m1);
        memberRepository.save(m2);

        Optional<Member> findMember3 = memberRepository.findOptionalByUsername("AAA");
        System.out.println("findMember3 = " + findMember3.get());
    }

}
```

- 결과

```
org.springframework.dao.IncorrectResultSizeDataAccessException: query did not return a unique result: 2; nested exception is javax.persistence.NonUniqueResultException: query did not return a unique result: 2
...
Caused by: javax.persistence.NonUniqueResultException: query did not return a unique result: 2
...
```

위와같이 IncorrectResultSizeDataAccessException를 반환한다. 사실 반환값이 두개 이상일경우 JPA에서는 NonUniqueResultException이 발생하도록 되어있는데 JPA에서만 사용하는 Exception 이기 때문에 Spring 공통의 Exception인 IncorrectResultSizeDataAccessException로 변환되어 반환하게 된다.

<br>

> 참고: 단건으로 지정한 메서드를 호출하면 스프링 데이터 JPA는 내부에서 JPQL의
Query.getSingleResult() 메서드를 호출한다. 이 메서드를 호출했을 때 조회 결과가 없으면 javax.persistence.NoResultException 예외가 발생하는데 개발자 입장에서 다루기가 상당히 불편하다. 스프링 데이터 JPA는 단건을 조회할 때 이 예외가 발생하면 예외를 무시하고 대신에 null 을 반환한다

<br><br><br>
## References 및 사진 출처

> __김영한의 실전! 스프링 데이터 JPA__