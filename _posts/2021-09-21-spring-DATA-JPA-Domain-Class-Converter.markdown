---
layout: post
title:  "spring Data JPA 도메인 클래스 컨버터"
subtitle:   "spring Data JPA 도메인 클래스 컨버터"
date:   2021-09-21 22:29:27 +0900
categories: spring
tags: spring JPA ORM Mapping String-Data-JPA Domain Class Converter
comments: true
---


<br>

- 목차
	- [도메인 클래스 컨버터](#도메인-클래스-컨버터)
	- [도메인 클래스 컨버터 사용 전](#도메인-클래스-컨버터-사용-전)
	- [도메인 클래스 컨버터 사용 후](#도메인-클래스-컨버터-사용-후)
    
<br>

# 도메인 클래스 컨버터

HTTP 파라미터로 넘어온 엔티티의 아이디로 엔티티 객체를 찾아서 바인딩한다.

<br><br>

# 도메인 클래스 컨버터 사용 전

```java
@RestController
@RequiredArgsConstructor
public class MemberController {

    private final MemberRepository memberRepository;

    @GetMapping("/members/{id}")
    public String findMember(@PathVariable("id") Long id) {
        Member member = memberRepository.findById(id).get();
        return member.getUsername();
    }

    @PostConstruct
    public void init() {
        Member member = new Member("user1");
        Member member2 = new Member("user2");
        Member member3 = new Member("user3");
        memberRepository.save(member);
        memberRepository.save(member2);
        memberRepository.save(member3);
    }
}
```

-결과

![그림1](https://sehwan-choi.github.io/assets/img/spring/SPRING-DATA-JPA/jpa2.jpg)

<br><br>

# 도메인 클래스 컨버터 사용 후

```java
@RestController
@RequiredArgsConstructor
public class MemberController {

    private final MemberRepository memberRepository;

    @GetMapping("/members/{id}")
    public String findMember(@PathVariable("id") Member member) {
        return member.getUsername();
    }

    @PostConstruct
    public void init() {
        Member member = new Member("user1");
        Member member2 = new Member("user2");
        Member member3 = new Member("user3");
        memberRepository.save(member);
        memberRepository.save(member2);
        memberRepository.save(member3);
    }
}
```

도메인 클래스 컨버터 사용전과 후의 결과는 같다.

- HTTP 요청은 회원 id 를 받지만 도메인 클래스 컨버터가 중간에 동작해서 회원 엔티티 객체를 반환한다.
- 도메인 클래스 컨버터도 리파지토리를 사용해서 엔티티를 찾음

> 주의: 도메인 클래스 컨버터로 엔티티를 파라미터로 받으면, ___이 엔티티는 단순 조회용으로만 사용해야 한다.___ (트랜잭션이 없는 범위에서 엔티티를 조회했으므로, 엔티티를 변경해도 DB에 반영되지 않는다.)

<br><br><br>
## References 및 사진 출처

> __김영한의 실전! 스프링 데이터 JPA__