---
layout: post
title:  "spring Data JPA Query Method"
subtitle:   "spring Data JPA Query Method"
date:   2021-09-17 23:45:27 +0900
categories: spring
tags: spring JPA ORM Mapping String-Data-JPA Query Method
comments: true
---


<br>

- 목차
	- [Query Method](#query-method)
	- [쿼리 메소드 필터 조건](#쿼리-메소드-필터-조건)
	- [스프링 데이터 JPA가 제공하는 쿼리 메소드 기능](#스프링-데이터-jpa가-제공하는-쿼리-메소드-기능)
    
<br>

# Query Method

<br>

이름과 나이를 기준으로 회원을 조회하려면?

- 순수 JPA 리포지토리 사용할 경우

```java
public List<Member> findByUsernameAndAgeGreaterThan(String username, int age) {
      return em.createQuery("select m from Member m where m.username = :username and m.age > :age")
            .setParameter("username", username)
            .setParameter("age",age)
            .getResultList();
}

@Test
public void findByUsernameAndAgeGreaterThen() {
      Member m1 = new Member("AAA", 10);
      Member m2 = new Member("AAA", 20);

      memberJpaRepository.save(m1);
      memberJpaRepository.save(m2);

      List<Member> result = memberJpaRepository.findByUsernameAndAgeGreaterThan("AAA", 15);

      Assertions.assertThat(result.get(0).getUsername()).isEqualTo("AAA");
      Assertions.assertThat(result.get(0).getAge()).isEqualTo(20);
      Assertions.assertThat(result.size()).isEqualTo(1);
}
```

위와같이 JPQL을 직접 작성해야한다.

<br>

- Spring Data JPA 사용할 경우

```java
public interface MemberRepository extends JpaRepository<Member, Long> {

    List<Member> findByUsernameAndAgeGreaterThan(String username, int age);
}

@Test
public void findByUsernameAndAgeGreaterThen() {
      Member m1 = new Member("AAA", 10);
      Member m2 = new Member("AAA", 20);

      memberRepository.save(m1);
      memberRepository.save(m2);

      List<Member> result = memberRepository.findByUsernameAndAgeGreaterThan("AAA", 15);

      Assertions.assertThat(result.get(0).getUsername()).isEqualTo("AAA");
      Assertions.assertThat(result.get(0).getAge()).isEqualTo(20);
      Assertions.assertThat(result.size()).isEqualTo(1);
}
```

 위와같이 Spring Dta JPA에서 제공하는 이름에 맞춰서 method를 생성하면 알아서 만들어 준다!

 <br>

 # 쿼리 메소드 필터 조건

|Keyword	|Sample	|JPQL snippet|
|:---|---|---:|
|Distinct|findDistinctByLastnameAndFirstname|select distinct …​ where x.lastname = ?1 and x.firstname = ?2|
|And|findByLastnameAndFirstname|… where x.lastname = ?1 and x.firstname = ?2|
|Or|findByLastnameOrFirstname|… where x.lastname = ?1 or x.firstname = ?2
|Is, Equals|findByFirstname,findByFirstnameIs,findByFirstnameEquals|… where x.firstname = ?1|
|Between|findByStartDateBetween|… where x.startDate between ?1 and ?2|
|LessThan|findByAgeLessThan|… where x.age < ?1|
|LessThanEqual|findByAgeLessThanEqual|… where x.age <= ?1|
|GreaterThan|findByAgeGreaterThan|… where x.age > ?1|
|GreaterThanEqual|findByAgeGreaterThanEqual|… where x.age >= ?1|
|After|findByStartDateAfter|… where x.startDate > ?1|
|Before|findByStartDateBefore|… where x.startDate < ?1|
|IsNull, Null|findByAge(Is)Null|… where x.age is null|
|IsNotNull, NotNull|findByAge(Is)NotNull|… where x.age not null|
|Like|findByFirstnameLike|… where x.firstname like ?1|
|NotLike|findByFirstnameNotLike|… where x.firstname not like ?1|
|StartingWith|findByFirstnameStartingWith|… where x.firstname like ?1 (parameter bound with appended %)|
|EndingWith|findByFirstnameEndingWith|… where x.firstname like ?1 (parameter bound with prepended %)|
|Containing|findByFirstnameContaining|… where x.firstname like ?1 (parameter bound wrapped in %)|
|OrderBy|findByAgeOrderByLastnameDesc|… where x.age = ?1 order by x.lastname desc|
|Not|findByLastnameNot|… where x.lastname <> ?1|
|In|findByAgeIn(Collection<Age> ages)|… where x.age in ?1|
|NotIn|findByAgeNotIn(Collection<Age> ages)|… where x.age not in ?1|
|True|findByActiveTrue()|… where x.active = true|
|False|findByActiveFalse()|… where x.active = false|
|IgnoreCase|findByFirstnameIgnoreCase|… where UPPER(x.firstname) = UPPER(?1)|


- 스프링 데이터 JPA 공식 문서 참고

> [https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#jpa.query-methods.query-creation](https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#jpa.query-methods.query-creation)

<br><br>

# 스프링 데이터 JPA가 제공하는 쿼리 메소드 기능

- 조회: find…By ,read…By ,query…By get…By,
  - findHelloBy 처럼 ...에 식별하기 위한 내용(설명)이 들어가도 된다.
  - 스프링 데이터 JPA 공식 문서 참고

> [https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#repositories.query-methods.query-creation](https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#repositories.query-methods.query-creation)

```java
public interface MemberRepository extends JpaRepository<Member, Long> {

    List<Member> findMemberAllSelectBy();
}
```

```
select
      member0_.member_id as member_i1_0_,
      member0_.age as age2_0_,
      member0_.team_id as team_id4_0_,
      member0_.username as username3_0_ 
from
      member member0_
```
위와 같이 findMemberAllSelectBy결과로 모든 Member를 조회하는 쿼리가 발생한다.

<br>

- COUNT: count…By 반환타입 long

```java
public interface MemberRepository extends JpaRepository<Member, Long> {

    long countMemberAllSelectBy();
}
```

```
select
      count(member0_.member_id) as col_0_0_ 
from
      member member0_
```
위와 같이 countMemberAllSelectBy 모든 Member의 count를 조회하는 쿼리가 발생한다.

<br>

- EXISTS: exists…By 반환타입 boolean

```java
public interface MemberRepository extends JpaRepository<Member, Long> {

    boolean existsMemberById(Long id);
}
```

```
select
      member0_.member_id as col_0_0_ 
from
      member member0_ 
where
      member0_.member_id=? limit ?
```
위와 같이 MemberId가 있는지 조회하는 쿼리가 발생한다.

<br>

- 삭제: delete…By, remove…By 반환타입 long

```java
public interface MemberRepository extends JpaRepository<Member, Long> {

    long deleteMemberAllBy();
    long deleteMemberById(Long id);
}
```

```
// deleteMemberAllBy 실행 결과
delete 
from
      member 
where
      member_id=?

// deleteMemberById 실행 결과
delete 
from
      member 
where
      member_id=?
```
위와 같이 deleteMemberAllBy와 deleteMemberById는 같아 보이지만 deleteMemberAllBy는 데이터가 10건이 있다면 각각 delete문이 10번이 나가게 되고 deleteMemberById는 특정 id값만 지운다.

<br>

- DISTINCT: findDistinct, findMemberDistinctBy

```java
public interface MemberRepository extends JpaRepository<Member, Long> {
    List<Member> findMemberDistinctBy();
}
```

```
select
      distinct member0_.member_id as member_i1_0_,
      member0_.age as age2_0_,
      member0_.team_id as team_id4_0_,
      member0_.username as username3_0_ 
from
      member member0_
```
위와 같이 distinct가 포함된 쿼리가 실행된다.

<br>

- LIMIT: findFirst3By, findFirstBy, findTopBy, findTop3By

```java
public interface MemberRepository extends JpaRepository<Member, Long> {
    List<Member> findFirst3By();
    List<Member> findTop3By();
    List<Member> findFirstBy();
}
```

```
// findFirst3By
select
      member0_.member_id as member_i1_0_,
      member0_.age as age2_0_,
      member0_.team_id as team_id4_0_,
      member0_.username as username3_0_ 
from
      member member0_ limit 3

// findTop3By
select
      member0_.member_id as member_i1_0_,
      member0_.age as age2_0_,
      member0_.team_id as team_id4_0_,
      member0_.username as username3_0_ 
from
      member member0_ limit 3

// findFirstBy
select
      member0_.member_id as member_i1_0_,
      member0_.age as age2_0_,
      member0_.team_id as team_id4_0_,
      member0_.username as username3_0_ 
from
      member member0_ limit 1
```
위와 같이 h2 Database에서, findFirst3By와 findTop3By 는 같은 쿼리가 실행되며 findFirstBy처럼 숫자가 없는 경우 limit 1로 데이터를 가져오게 된다.

<br>

- 스프링 데이터 JPA 공식 문서 참고
> [https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#repositories.limit-query-result](https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#repositories.limit-query-result)

<br>

> 참고: 이 기능은 엔티티의 필드명이 변경되면 인터페이스에 정의한 메서드 이름도 꼭 함께 변경해야 한다.  그렇지 않으면 애플리케이션을 시작하는 시점에 오류가 발생한다.
> 이렇게 애플리케이션 로딩 시점에 오류를 인지할 수 있는 것이 스프링 데이터 JPA의 매우 큰 장점이다



<br><br><br>
## References 및 사진 출처

> __김영한의 실전! 스프링 데이터 JPA__