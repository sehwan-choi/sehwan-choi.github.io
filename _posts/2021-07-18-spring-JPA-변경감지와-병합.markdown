---
layout: post
title:  "JPA 변경 감지(Dirty Checking)와 병합(Merge)"
subtitle:   "JPA 변경 감지(Dirty Checking)와 병합(Merge)"
date:   2021-07-18 23:50:27 +0900
categories: spring
tags: spring dirtychecking merge update modify
comments: true
---

# JPA 변경 감지(Dirty Checking)와 병합(Merge)

<br>

- 목차
	- [변경 감지와 병합(Merge)](#변경-감지와-병합merge)
	    - [준영속 엔티티](#준영속-엔티티)
	- [준영속 엔티티를 수정하는 2가지 방법](#준영속-엔티티를-수정하는-2가지-방법)
	    - [변경 감지 기능 사용(Dirty Checking)](#변경-감지-기능-사용dirty-checking)
	    - [병합 사용(Merge)](#병합-사용merge)
	    - [병합 동작 방식](#병합-동작-방식)
	    - [병합 동작 주의점](#병합-동작-주의점)
	    - [가장 좋은 해결 방법](#가장-좋은-해결-방법)
<br>

<br>

# 변경 감지와 병합(Merge)

>참고: 정말 중요한 내용이니 꼭! 완벽하게 이해하셔야 합니다.

<br>
<br>

## 준영속 엔티티
영속성 컨텍스트가 더는 관리하지 않는 엔티티를 말한다.

ex). 영속 엔티티
```java
@Service
@Transactional(readOnly = true)
@RequiredArgsConstructor
public class MemberService {

    private final MemberRepository memberRepository;

    @Transactional
    public void test (Long memberId) {
        Member member = memberRepository.findOne(memberId); 
        member.setName("test2");
    }
}

@Repository
@RequiredArgsConstructor
public class MemberRepository {
    
    private final EntityManager em;

    public void save(Member member) { em.persist(member); }
    public Member findOne(Long id) { return em.find(Member.class, id); }
}
```

영속성 컨텍스트가 관리하는 엔티티를 memberRepository.findOne으로 찾아왔기 때문에 영속성 엔티티이다.
위와같이 이름이 변경된경우 @Transactional 으로 인해 memberRepository.save를 하지 않아도 자동으로 데이터베이스에 반영이 된다.
<br>
<br>

ex). 준영속 엔티티
```java

@Service
@Transactional(readOnly = true)
@RequiredArgsConstructor
public class MemberService {

    private final MemberRepository memberRepository;

    @Transactional
    public void test2 (Long memberId) {
    Member member = memberRepository.findOne(memberId); 

    Member member2 = new Member();
    member2.setName(member.getName());
    member2.setId(member.getId());
    memberRepository.save(member2);
    }
}

@Repository
@RequiredArgsConstructor
public class MemberRepository {
    
    private final EntityManager em;

    public void save(Member member) { em.persist(member); }
    public Member findOne(Long id) { return em.find(Member.class, id); }
}
```

하지만 위와 같이 새로 생성한 member2 객체에 영속 엔티티인 member의 값을 set한다고 해도 member2 객체는 영속성 컨테스트가 관리하지 않는 준영속 엔티티상태이다. 왜냐하면 당연하게도 new 로 만들었기 때문이다. 그렇기 때문에 member2의 값을 변경한후 memberRepository.save를 호출하지 않으면 데이터는 변경되지 않을 것이다.
<br><br>

# 준영속 엔티티를 수정하는 2가지 방법
<br>

## 변경 감지 기능 사용(Dirty Checking)

```java
@Transactional
void update(Item itemParam) { //itemParam: 파리미터로 넘어온 준영속 상태의 엔티티
    Item findItem = em.find(Item.class, itemParam.getId()); //같은 엔티티를 조회한
다.
    findItem.setPrice(itemParam.getPrice()); //데이터를 수정한다.
}
```
영속성 컨텍스트에서 엔티티를 다시 조회한 후에 데이터를 수정하는 방법<br>
트랜잭션 안에서 엔티티를 다시 조회, 변경할 값 선택 트랜잭션 커밋 시점에 변경 감지(Dirty Checking)이 동작해서 데이터베이스에 UPDATE SQL 실행한다.
<br><br>

## 병합 사용(Merge)

```java
@Transactional
void update(Item itemParam) { //itemParam: 파리미터로 넘어온 준영속 상태의 엔티티
 Item mergeItem = em.merge(item);
}
```
병합은 준영속 상태의 엔티티를 영속 상태로 변경할 때 사용하는 기능이다.

![그림1](https://sehwan-choi.github.io/assets/img/spring/JPA1/jpa1.jpg)

<br>

## 병합 동작 방식

1. merge() 를 실행한다.
2. 파라미터로 넘어온 준영속 엔티티의 식별자 값으로 1차 캐시에서 엔티티를 조회한다.
2-1. 만약 1차 캐시에 엔티티가 없으면 데이터베이스에서 엔티티를 조회하고, 1차 캐시에 저장한다.
3. 조회한 영속 엔티티( mergeMember )에 member 엔티티의 값을 채워 넣는다. (member 엔티티의 모든 값
을 mergeMember에 밀어 넣는다. 이때 mergeMember의 “회원1”이라는 이름이 “회원명변경”으로 바
뀐다.)
4. 영속 상태인 mergeMember를 반환한다
<br><br>

## 병합 동작 주의점
변경 감지 기능을 사용하면 원하는 속성만 선택해서 변경할 수 있지만, 병합을 사용하면 모든 속성이 변경된다.<br>
 __병합시 값이 없으면 null 로 업데이트 할 위험도 있다. (병합은 모든 필드를 교체한다.)__


```java

@Entity
@Getter
@Setter
public class Member {

    @Id
    @GeneratedValue
    @Column(name = "member_id")
    private  Long id;
    private String name;
    private String passwd;
    private String email;
    private String phone;
}

@Service
@Transactional(readOnly = true)
@RequiredArgsConstructor
public class MemberService {

    private final MemberRepository memberRepository;

    @Transactional
    public void test (Long memberId) {
    Member member = memberRepository.findOne(memberId); 

    Member member2 = new Member();
    member2.setName(member.getName());
    member2.setId(member.getId());
    // passwd = null
    // email = null
    // phone = null
    memberRepository.merge(member2);
    }
}

@Repository
@RequiredArgsConstructor
public class MemberRepository {
    
    private final EntityManager em;

    public void merge(Member member) { em.merge(item); }
}
```
위와 같이 기존 member객체에는 모든 필드가 채워져있다고 가정했을때, member 데이터를 member2 데이터에 set 한 후 merge를 하게 되면 passwd, email, phone은 모두 null 값이 들어가게 될 것이다.
<br><br>

## 가장 좋은 해결 방법

엔티티를 변경할 때는 항상 __변경 감지__ 를 사용해야 한다.

- 컨트롤러에서 어설프게 엔티티를 생성하지 않는다.
- 트랜잭션이 있는 서비스 계층에 식별자( id )와 변경할 데이터를 명확하게 전달한다.(파라미터 or dto)
- 트랜잭션이 있는 서비스 계층에서 영속 상태의 엔티티를 조회하고, 엔티티의 데이터를 직접 변경한다.
- 트랜잭션 커밋 시점에 변경 감지가 실행된다.

<br><br><br>
## References 및 사진 출처

> __김영한의 실전! 스프링 부트와 JPA 활용1__

> __책 자바 ORM 표준 JPA 프로그래밍 3.6.5__