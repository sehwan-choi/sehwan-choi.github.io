---
layout: post
title:  "spring JPA 연관관계 맵핑"
subtitle:   "spring JPA 연관관계 맵핑"
date:   2021-08-04 01:00:27 +0900
categories: spring
tags: spring JPA ORM Mapping
comments: true
---

# 연관관계 매핑

<br>

- 목차
	- [연관관계 매핑](#연관관계-매핑)
	- [연관관계가 필요한 이유](#연관관계가-필요한-이유)
	    - [예제 시나리오](#예제-시나리오)
	    - [객체를 테이블에 맞추어 모델링](#객체를-테이블에-맞추어-모델링)
	    - [결론](#결론)
	- [단방향 연관관계](#단방향-연관관계)
	    - [객체 지향 모델링](#객체-지향-모델링)
	- [양방향 연관관계](#양방향-연관관계)
	    - [양방향 매핑](#양방향-매핑)
	- [연관관계의 주인과 mappedBy](#연관관계의-주인과-mappedby)
	    - [객체와 테이블이 관계를 맺는 차이](#객체와-테이블이-관계를-맺는-차이)
	    - [객체의 양방향 관계](#객체의-양방향-관계)
	    - [테이블의 양방향 연관관계](#테이블의-양방향-연관관계)
	    - [연관관계의 주인(Owner)](#연관관계의-주인owner)
	    - [누구를 주인으로?](#누구를-주인으로)
	    - [양방향 매핑시 가장 많이 하는 실수](#양방향-매핑시-가장-많이-하는-실수)
	    - [양방향 연관관계 주의사항](#양방향-연관관계-주의사항)
	    - [정리](#정리)
	    - [연관관계의 주인을 정하는 기준](#연관관계의-주인을-정하는-기준)
<br>

<br>

# 연관관계가 필요한 이유

<br>

## 예제 시나리오
- 회원과 팀이 있다. 
- 회원은 하나의 팀에만 소속될 수 있다. 
- 회원과 팀은 다대일 관계다.
<br>

![그림1](https://sehwan-choi.github.io/assets/img/spring/JPA-MAPPING/jpa1.jpg)

<br>

## 객체를 테이블에 맞추어 모델링
참조 대신에 외래 키를 그대로 사용
```java
@Entity
 public class Member { 
    @Id 
    @GeneratedValue
    private Long id;

    @Column(name = "USERNAME")
    private String name;

    @Column(name = "TEAM_ID")
    private Long teamId; 
    … 
 } 

 @Entity
 public class Team {
    @Id 
    @GeneratedValue
    private Long id;

    private String name; 
    … 
 }

```
```java
//팀 저장
 Team team = new Team();
 team.setName("TeamA");
 em.persist(team);
 //회원 저장
 Member member = new Member();
 member.setName("member1");
 member.setTeamId(team.getId());
 em.persist(member);
```
```java
//조회
 Member findMember = em.find(Member.class, member.getId()); 
 //연관관계가 없음
 Team findTeam = em.find(Team.class, team.getId());
```
위와같이 외래 키를 직접 다루게 되며, Team을 조회 하기 위해서는 Member를 조회한 후 team의 PK를 가지고 또 조회를 해와야하는 번거로움이 있다. 그리고 이 방법은 객체 지향적인 방법이 아니다.

<br>

## 결론
- 객체를 테이블에 맞추어 데이터 중심으로 모델링하면, 협력 관계를 만들 수 없다.
- 테이블은 외래 키로 조인을 사용해서 연관된 테이블을 찾는다. 
- 객체는 참조를 사용해서 연관된 객체를 찾는다. 
- 테이블과 객체 사이에는 이런 큰 간격이 있다

<br>

# 단방향 연관관계

<br>

![그림1](https://sehwan-choi.github.io/assets/img/spring/JPA-MAPPING/jpa2.jpg)

<br>

## 객체 지향 모델링
객체의 참조와 테이블의 외래 키를 매핑

```java
 @Entity
 public class Member { 
    @Id @GeneratedValue
    private Long id;

    @Column(name = "USERNAME")
    private String name;

    private int age;

    // @Column(name = "TEAM_ID")
    // private Long teamId;

    @ManyToOne
    @JoinColumn(name = "TEAM_ID")
    private Team team;
    … 
 ```

<br>

![그림1](https://sehwan-choi.github.io/assets/img/spring/JPA-MAPPING/jpa3.jpg)

```java
//팀 저장
 Team team = new Team();
 team.setName("TeamA");
 em.persist(team);

 //회원 저장
 Member member = new Member();
 member.setName("member1");
 member.setTeam(team); //단방향 연관관계 설정, 참조 저장
 em.persist(member);
```

```java
//조회
 Member findMember = em.find(Member.class, member.getId()); 
//참조를 사용해서 연관관계 조회
 Team findTeam = findMember.getTeam();
 ```
 member를 조회한후 team 정보를 확인하고 싶다면 참조값을 통해서 조회할 수 있다.
 <br>

 ```java
  // 새로운 팀B
 Team teamB = new Team();
 teamB.setName("TeamB");
 em.persist(teamB);
 // 회원1에 새로운 팀B 설정
 member.setTeam(teamB);
 ```
 기존 member의 team정보를 수정하는 것도 가능하다.

<br>

![그림1](https://sehwan-choi.github.io/assets/img/spring/JPA-MAPPING/jpa4.jpg)
 <br><br>


 # 양방향 연관관계


<br>

## 양방향 매핑

Member 엔티티는 단방향과 동일하며, Team 엔티티는 컬렉션이 추가된다.
```java
@Entity
 public class Member { 
    @Id 
    @GeneratedValue
    private Long id;
    
    @Column(name = "USERNAME")
    private String name;
    
    private int age;
    
    @ManyToOne
    @JoinColumn(name = "TEAM_ID")
    private Team team;
    …
 ```
 ```java
 @Entity
 public class Team {
    @Id 
    @GeneratedValue
    private Long id;
    
    private String name;
    @OneToMany(mappedBy = "team")
    List<Member> members = new ArrayList<Member>();
    … 
 }
```
```java
//조회
 Team findTeam = em.find(Team.class, team.getId()); 
 int memberSize = findTeam.getMembers().size(); //역방향 조회
```
위와 같이 team을 조회한후 getMembers를 통해서 해당 team에 속한 member를 조회할 수 있다.

<br>

# 연관관계의 주인과 mappedBy

<br>

## 객체와 테이블이 관계를 맺는 차이
- 객체 연관관계 = 2개
    - 회원 -> 팀 연관관계 1개(단방향) 
    - 팀 -> 회원 연관관계 1개(단방향) 
- 테이블 연관관계 = 1개
    - 회원 <-> 팀의 연관관계 1개(양방향)

<br>

![그림1](https://sehwan-choi.github.io/assets/img/spring/JPA-MAPPING/jpa5.jpg)

<br>

## 객체의 양방향 관계

- 객체의 양방향 관계는 사실 양방향 관계가 아니라 서로 다른 단뱡향 관계 2개다.
- 객체를 양방향으로 참조하려면 단방향 연관관계를 2개 만들어야 한다.

```java
A -> B (a.getB())
B -> A (b.getA())

class A {
    B b;
}

class B {
    A a;
}
```

<br>

## 테이블의 양방향 연관관계

- 테이블은 외래 키 하나로 두 테이블의 연관관계를 관리
- MEMBER.TEAM_ID 외래 키 하나로 양방향 연관관계 가짐 (양쪽으로 조인할 수 있다.)

```SQL
SELECT * 
FROM MEMBER M
JOIN TEAM T ON M.TEAM_ID = T.TEAM_ID 

SELECT * 
FROM TEAM T
JOIN MEMBER M ON T.TEAM_ID = M.TEAM_ID
```

<br>

- 둘 중 하나로 외래 키를 관리해야 한다.

![그림1](https://sehwan-choi.github.io/assets/img/spring/JPA-MAPPING/jpa6.jpg)

<br>

## 연관관계의 주인(Owner)

양방향 매핑 규칙
- 객체의 두 관계중 하나를 연관관계의 주인으로 지정
- __연관관계의 주인만이 외래 키를 관리(등록, 수정)__
- __주인이 아닌쪽은 읽기만 가능__ ( 매우 중요 )
- 주인은 mappedBy 속성 사용X
- 주인이 아니면 mappedBy 속성으로 주인 지정

<br>

## 누구를 주인으로?

- 외래 키가 있는 있는 곳을 주인으로 정해라
- 여기서는 Member.team이 연관관계의 주인

![그림1](https://sehwan-choi.github.io/assets/img/spring/JPA-MAPPING/jpa7.jpg)

<br>

## 양방향 매핑시 가장 많이 하는 실수

<br>
- 연관관계의 주인에 값을 입력하지 않음

```java
Team team = new Team();
 team.setName("TeamA");
 em.persist(team);
 Member member = new Member();
 member.setName("member1");
 //역방향(주인이 아닌 방향)만 연관관계 설정
 team.getMembers().add(member);
 em.persist(member);
 ```

![그림1](https://sehwan-choi.github.io/assets/img/spring/JPA-MAPPING/jpa8.jpg)

 위와같이 연관관계 주인은 Member인데 team의 member list에 add를 한다. team의 member list는 __읽기 전용__ 임을 꼭 기억해야 한다. 실행 결과 외래키가 null로 들어가게 된다. (버그 발생할 가능성이 매우 높다.)

 <br>

 ```java
 Team team = new Team();
 team.setName("TeamA");
 em.persist(team);
 Member member = new Member();
 member.setName("member1");
 team.getMembers().add(member); 
 //연관관계의 주인에 값 설정
 member.setTeam(team); //**
 em.persist(member);
 ```

위와같이 연관관계 주인인 Member에 반드시 세팅해야한다.

<br>

## 양방향 연관관계 주의사항

- 순수 객체 상태를 고려해서 항상 양쪽에 값을 설정하자
- 연관관계 편의 메소드를 생성하자
- 양방향 매핑시에 무한 루프를 조심하자
    - 예: toString(), lombok, JSON 생성 라이브러리

<br>

## 정리

- 단방향 매핑만으로도 이미 연관관계 매핑은 완료
- 양방향 매핑은 반대 방향으로 조회(객체 그래프 탐색) 기능이 추
가된 것 뿐
- JPQL에서 역방향으로 탐색할 일이 많음
- 단방향 매핑을 잘 하고 양방향은 필요할 때 추가해도 됨(테이블에 영향을 주지 않음)

<br><br>

## 연관관계의 주인을 정하는 기준

- 비즈니스 로직을 기준으로 연관관계의 주인을 선택하면 안됨
- 연관관계의 주인은 외래 키의 위치를 기준으로 정해야함

<br><br><br>
## References 및 사진 출처

> __김영한의 자바 ORM 표준 JPA 프로그래밍 - 기본편__
https://gmlwjd9405.github.io/2019/08/03/reason-why-use-jpa.html