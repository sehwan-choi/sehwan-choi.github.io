---
layout: post
title:  "spring JPA 다양한 연관관계 맵핑"
subtitle:   "spring JPA 다양한 연관관계 맵핑"
date:   2021-08-12 01:00:27 +0900
categories: spring
tags: spring JPA ORM Mapping 1:N N:1 1:1 N:M
comments: true
---

# 다양한 연관관계 매핑

<br>

- 목차
	- [다양한 연관관계 매핑](#다양한-연관관계-매핑)
	- [연관관계 매핑시 고려사항 3가지](#연관관계-매핑시-고려사항-3가지)
	- [다대일 (N:1)](#다대일-n1)
	    - [다대일 단방향](#다대일-단방향)
	    - [다대일 양방향](#다대일-양방향)
	- [일대다 (1:N)](#일대다-1n)
	    - [일대다 단방향](#일대다-단방향)
	    - [일대다 양방향](#일대다-양방향)
	- [일대일 (1:1)](#일대일-11)
	    - [일대일 관계](#일대일-관계)
	    - [일대일 정리](#일대일-정리)
	- [다대다 (N:M)](#다대다-nm)
	    - [다대다](#다대다)
	    - [다대다 매핑의 한계](#다대다-매핑의-한계)
    	- [다대다 한계 극복](#다대다-한계-극복)
<br>

<br>

# 연관관계 매핑시 고려사항 3가지

- 다중성
    - 다대일: @ManyToOne 
    - 일대다: @OneToMany 
    - 일대일: @OneToOne 
    - 다대다: @ManyToMany

<br>

- 단방향, 양방향
    - 테이블
        - 외래 키 하나로 양쪽 조인 가능
        - 사실 방향이라는 개념이 없음
    - 객체
        - 참조용 필드가 있는 쪽으로만 참조 가능
        - 한쪽만 참조하면 단방향
        - 양쪽이 서로 참조하면 양방향

<br>

- 연관관계의 주인
    - 테이블은 외래 키 하나로 두 테이블이 연관관계를 맺음
    - 객체 양방향 관계는 A->B, B->A 처럼 참조가 2군데
    - 객체 양방향 관계는 참조가 2군데 있음. 둘중 테이블의 외래 키를 관리할 곳을 지정해야함
    - 연관관계의 주인: 외래 키를 관리하는 참조
    - 주인의 반대편: 외래 키에 영향을 주지 않음, 단순 조회만 가능

<br><br>

# 다대일 (N:1)

<br>

## 다대일 단방향

![그림1](https://sehwan-choi.github.io/assets/img/spring/JPA-MAPPING2/jpa1.jpg)

- 가장 많이 사용하는 연관관계
- 다대일의 반대는 일대다

```java
@Entity
public class Members {

    @Id
    @GeneratedValue
    @Column(name = "MEMBERS_ID")
    private Long id;

    @ManyToOne
    @JoinColumn(name = "TEAM_ID")
    private Team team;

    private String username;
}

@Entity
public class Team {

    @Id
    @GeneratedValue
    @Column(name = "TEAM_ID")
    private Long id;

    private String name;
}

```

<br>

## 다대일 양방향

![그림1](https://sehwan-choi.github.io/assets/img/spring/JPA-MAPPING2/jpa2.jpg)

- 외래 키가 있는 쪽이 연관관계의 주인
- 양쪽을 서로 참조하도록 개발

```java
@Entity
public class Members {

    @Id
    @GeneratedValue
    @Column(name = "MEMBERS_ID")
    private Long id;

    //연관관계 주인이므로 JoinColumn사용
    @ManyToOne
    @JoinColumn(name = "TEAM_ID")
    private Team team;

    private String username;
}

@Entity
public class Team {

    @Id
    @GeneratedValue
    @Column(name = "TEAM_ID")
    private Long id;

    //members를 읽기전용으로 사용하게 된다.(수정불가능)
    @OneToMany(mappedBy = "team")
    private List<Members> membersList = new ArrayList<>();

    private String name;
}
```

<br>

## 다대일 주요속성

![그림1](https://sehwan-choi.github.io/assets/img/spring/JPA-MAPPING2/jpa13.jpg)


<br><br>

# 일대다 (1:N)

<br>

## 일대다 단방향

![그림1](https://sehwan-choi.github.io/assets/img/spring/JPA-MAPPING2/jpa3.jpg)

- 일대다 단방향은 일대다(1:N)에서 일(1)이 연관관계의 주인
- 테이블 일대다 관계는 항상 다(N) 쪽에 외래 키가 있음
- 객체와 테이블의 차이 때문에 반대편 테이블의 외래 키를 관리하는 특이한 구조
- @JoinColumn을 꼭 사용해야 함. 그렇지 않으면 조인 테이블방식을 사용함(중간에 테이블을 하나 추가함)
- 일대다 단방향 매핑의 단점
    - 엔티티가 관리하는 외래 키가 다른 테이블에 있음
    - 연관관계 관리를 위해 추가로 UPDATE SQL 실행
- 일대다 단방향 매핑보다는 다대일 양방향 매핑을 사용하자

```java
@Entity
public class Members {

    @Id
    @GeneratedValue
    @Column(name = "MEMBERS_ID")
    private Long id;

    private String username;
}

@Entity
public class Team {

    @Id
    @GeneratedValue
    @Column(name = "TEAM_ID")
    private Long id;

    //Members테이블의 TEAM_ID(FK)를 관리해야 하기떄문에 TEAM_ID를 JoinColumn으로 설정
    @OneToMany
    @JoinColumn(name = "TEAM_ID")
    private List<Members> membersList = new ArrayList<>();

    private String name;
}
```

<br>

## 일대다 양방향

![그림1](https://sehwan-choi.github.io/assets/img/spring/JPA-MAPPING2/jpa4.jpg)

- 이런 매핑은 공식적으로 존재X (꼼수를 사용해서 일대다 양방향 처럼 보이게 함)
- @JoinColumn(insertable=false, updatable=false) 
- 읽기 전용 필드를 사용해서 양방향 처럼 사용하는 방법
- 다대일 양방향을 사용하자

```java
@Entity
public class Members {

    @Id
    @GeneratedValue
    @Column(name = "MEMBERS_ID")
    private Long id;

    private String username;

    @ManyToOne
    @JoinColumn(name = "TEAM_ID" ,insertable = false, updatable = false)
    private Team team;
}

@Entity
public class Team {

    @Id
    @GeneratedValue
    @Column(name = "TEAM_ID")
    private Long id;

    @OneToMany
    @JoinColumn(name = "TEAM_ID")
    private List<Members> membersList = new ArrayList<>();

    private String name;
}
```
위 코드에서 inserable = false, updatable = false를 사용하는 이유는, 지금 Team과 Members가 양방향 처럼 보이지만 사실은 단방향으로 서로를 참조하는 형태이다. 만약 inserable = false, updatable = false이 없다면 Members입장에서는 자신이 연관관계 주인이고, Team입장에서는 자신이 연관관계 주인이기 때문에 말도 안되는 현상이 발생하기 때문에 Members에 inserable = false, updatable = false를 추가함으로써 자신은 연관관계의 주인이 아니라는 것을 설정하게 된다(읽기전용). 위 코드에서는 Team이 연관관계의 주인이 된다. 사실 이 방법은 꼼수에 해당한다. 이런 맵핑은 공식적으로 존재하지 않기때문이다.

<br>

## 일대다 주요속성

![그림1](https://sehwan-choi.github.io/assets/img/spring/JPA-MAPPING2/jpa14.jpg)


<br><br>

# 일대일 (1:1)

<br>

## 일대일 관계

- 주 테이블이나 대상 테이블 중에 외래 키 선택 가능
- 주 테이블에 외래 키
- 대상 테이블에 외래 키
- 외래 키에 데이터베이스 유니크(UNI) 제약조건 추가

<br>

## 일대일: 주 테이블에 외래 키 단방향

![그림1](https://sehwan-choi.github.io/assets/img/spring/JPA-MAPPING2/jpa5.jpg)

- 다대일(@ManyToOne) 단방향 매핑과 유사

```java
@Entity
public class Members {

    @Id
    @GeneratedValue
    @Column(name = "MEMBERS_ID")
    private Long id;

    private String username;

    @OneToOne
    @JoinColumn(name = "LOCKER_ID")
    private Locker locker;
}

@Entity
public class Locker {

    @Id
    @GeneratedValue
    @Column(name = "LOCKER_ID")
    private Long id;

    private String name;
}
```

<br>

> 주 테이블이란 기본적으로 많이 사용하고 많이 쿼리하는 테이블을 말함

<br>

## 일대일: 주 테이블에 외래 키 양방향

![그림1](https://sehwan-choi.github.io/assets/img/spring/JPA-MAPPING2/jpa6.jpg)

- 다대일 양방향 매핑 처럼 외래 키가 있는 곳이 연관관계의 주인
- 반대편은 mappedBy 적용

```java
@Entity
public class Members {

    @Id
    @GeneratedValue
    @Column(name = "MEMBERS_ID")
    private Long id;

    private String username;

    //연관관계 주인이기 때문에 JoinColumn을 사용
    @OneToOne
    @JoinColumn(name = "LOCKER_ID")
    private Locker locker;
}

@Entity
public class Locker {

    @Id
    @GeneratedValue
    @Column(name = "LOCKER_ID")
    private Long id;

    private String name;

    @OneToOne(mappedBy = "locker")
    private Members members;
}
```

<br>

## 일대일: 대상 테이블에 외래 키 단방향

![그림1](https://sehwan-choi.github.io/assets/img/spring/JPA-MAPPING2/jpa7.jpg)

- 단방향 관계는 JPA 지원X
- 양방향 관계는 지원

<br>

## 일대일: 대상 테이블에 외래 키 양방향

![그림1](https://sehwan-choi.github.io/assets/img/spring/JPA-MAPPING2/jpa8.jpg)

- 사실 일대일 주 테이블에 외래 키 양방향과 매핑 방법은 같음

<br>

## 일대일 정리
- 주 테이블에 외래 키
    - 주 객체가 대상 객체의 참조를 가지는 것 처럼 주 테이블에 외래 키를 두고 대상 테이블을 찾음
    - 객체지향 개발자 선호
    - JPA 매핑 편리
    - 장점: 주 테이블만 조회해도 대상 테이블에 데이터가 있는지 확인 가능
    - 단점: 값이 없으면 외래 키에 null 허용

<br>
    
- 대상 테이블에 외래 키
    - 대상 테이블에 외래 키가 존재
    - 전통적인 데이터베이스 개발자 선호
    - 장점: 주 테이블과 대상 테이블을 일대일에서 일대다 관계로 변경할 때 테이블 구조 유지
    - 단점: 프록시 기능의 한계로 지연 로딩으로 설정해도 항상 즉시 로딩됨

<br><br>

# 다대다 (N:M)

<br>

## 다대다

![그림1](https://sehwan-choi.github.io/assets/img/spring/JPA-MAPPING2/jpa9.jpg)

- 관계형 데이터베이스는 정규화된 테이블 2개로 다대다 관계를 표현할 수 없음
- 연결 테이블을 추가해서 일대다, 다대일 관계로 풀어내야함


<br>

![그림1](https://sehwan-choi.github.io/assets/img/spring/JPA-MAPPING2/jpa10.jpg)

- 객체는 컬렉션을 사용해서 객체 2개로 다대다 관계 가능

```java
@Entity
public class Members {

    @Id
    @GeneratedValue
    @Column(name = "MEMBERS_ID")
    private Long id;

    private String username;

    @ManyToMany
    @JoinTable(name = "MEMBERS_PRODUCT")
    private List<Product> products = new ArrayList<>();
}

@Entity
public class Product {

    @Id
    @GeneratedValue
    @Column(name = "PRODUCT_ID")
    private Long id;

}
```

<br>

- @ManyToMany 사용
- @JoinTable로 연결 테이블 지정
- 다대다 매핑: 단방향, 양방향 가능

<br>

## 다대다 매핑의 한계

![그림1](https://sehwan-choi.github.io/assets/img/spring/JPA-MAPPING2/jpa11.jpg)

- 편리해 보이지만 실무에서 사용X 
- 연결 테이블이 단순히 연결만 하고 끝나지 않음
- 자동으로 만들어주는 연결 테이블에 원하는 컬럼을 추가하기 힘듬

<br>

## 다대다 한계 극복

![그림1](https://sehwan-choi.github.io/assets/img/spring/JPA-MAPPING2/jpa12.jpg)

- 연결 테이블용 엔티티 추가(연결 테이블을 엔티티로 승격) 
- 1:N, N:1의 연결관계로 만듬
- @ManyToMany -> @OneToMany, @ManyToOne

```java
@Entity
public class Members {

    @Id
    @GeneratedValue
    @Column(name = "MEMBERS_ID")
    private Long id;

    private String username;

    @OneToMany(mappedBy = "member")
    private List<Product> products = new ArrayList<>();
}

@Entity
public class Product {

    @Id
    @GeneratedValue
    @Column(name = "PRODUCT_ID")
    private Long id;

    @OneToMany(mappedBy = "product")
    private List<Orders> orders = new ArrayList<>();
}

@Entity
public class Orders {

    @Id
    @GeneratedValue
    @Column(name = "ORDER_ID")
    private Long id;

    @ManyToOne
    @JoinColumn(name = "MEMBERS_ID")
    private Members member;

    @ManyToOne
    @JoinColumn(name = "PRODUCT_ID")
    private Product product;

    private int orderAmount;

    private LocalDateTime orderDate;
}
```

<br><br><br>
## References 및 사진 출처

> __김영한의 자바 ORM 표준 JPA 프로그래밍 - 기본편__