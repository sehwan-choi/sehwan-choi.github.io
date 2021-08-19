---
layout: post
title:  "spring JPA 값타입 - 임베디드 타입"
subtitle:   "spring JPA ValueType Embedded"
date:   2021-08-19 17:00:27 +0900
categories: spring
tags: spring JPA ORM Mapping ValueType Embedded
comments: true
---


<br>

- 목차
	- [임베디드 타입(복합 값 타입)](#임베디드-타입복합-값-타입)
	- [임베디드 타입 사용법](#임베디드-타입-사용법)
	- [임베디드 타입 설정](#임베디드-타입-설정)
	- [임베디드 타입의 장점](#임베디드-타입의-장점)
	- [임베디드 타입과 테이블 매핑](#임베디드-타입과-테이블-매핑)
	- [임베디드 타입과 연관관계](#임베디드-타입과-연관관계)
	- [중복된 임베디드 값 사용](#중복된-임베디드-값-사용)
	- [임베디드 타입과 null](#임베디드-타입과-null)
    
<br>

# 임베디드 타입(복합 값 타입)

- 새로운 값 타입을 직접 정의할 수 있음
- JPA는 임베디드 타입(embedded type)이라 함
- 주로 기본 값 타입을 모아서 만들어서 복합 값 타입이라고도 함
- int, String과 같은 값 타입

<br>

## 임베디드 타입 사용법

- @Embeddable: 값 타입을 정의하는 곳에 표시
- @Embedded: 값 타입을 사용하는 곳에 표시
- 기본 생성자 필수

<br>

## 임베디드 타입 설정

- 회원 엔티티는 이름, 근무 시작일, 근무 종료일, 주소 도시, 주소 번지, 주소 우편번호를 가진다.

![그림1](https://sehwan-choi.github.io/assets/img/spring/JPA-MAPPING4/jpa13.jpg)

- 회원 엔티티는 이름, 근무 기간, 집 주소를 가진다.

![그림1](https://sehwan-choi.github.io/assets/img/spring/JPA-MAPPING4/jpa14.jpg)

![그림1](https://sehwan-choi.github.io/assets/img/spring/JPA-MAPPING4/jpa15.jpg)

<br>

```java
@Entity
public class Member{

    @Id
    @GeneratedValue
    @Column(name = "MEMBER_ID")
    private Long id;

    private String username;

    @Embedded
    private Period period;

    @Embedded
    private Address homeAddress;

    ...
}

@Embeddable
public class Address {

    private String city;
    private String street;
    private String zipcode;

    public Address() {  // 기본생성자 필수!
    }

    public Address(String city, String street, String zipcode) {
        this.city = city;
        this.street = street;
        this.zipcode = zipcode;
    }

    ...
}

@Embeddable
public class Period {

    private LocalDateTime startDate;

    private LocalDateTime endDate;

    public Period() {   // 기본생성자 필수!
    }

    public Period(LocalDateTime startDate, LocalDateTime endDate) {
        this.startDate = startDate;
        this.endDate = endDate;
    }

    ...

}

public class jpaMain {

    public static void main(String[] args) {

        ...

        try {

            Member member = new Member();

            member.setUsername("NAME");
            member.setPeriod(new Period(LocalDateTime.now(),LocalDateTime.now()));
            member.setHomeAddress(new Address("CITY","STREET","ZIPCODE"));

            em.persist(member);

            em.flush();
            em.clear();

            Member member1 = em.find(Member.class, member.getId());

            System.out.println("member city = " + member1.getHomeAddress().getCity());
            System.out.println("member street = " + member1.getHomeAddress().getStreet());
            System.out.println("member zipcode = " + member1.getHomeAddress().getZipcode());


            System.out.println("member StartDate = " + member1.getPeriod().getStartDate());
            System.out.println("member EndDate = " + member1.getPeriod().getEndDate());
        }

        ...
```

- 실행결과

![그림1](https://sehwan-choi.github.io/assets/img/spring/JPA-MAPPING4/jpa16.jpg)

위 사진과 같이 DB에 정상적으로 값이 들어가있다.

<br>

## 임베디드 타입의 장점
- 재사용
- 높은 응집도
- Period.isWork()처럼 해당 값 타입만 사용하는 의미 있는 메소드를 만들 수 있음
- 임베디드 타입을 포함한 모든 값 타입은, 값 타입을 소유한 엔티티에 생명주기를 의존함

<br>

## 임베디드 타입과 테이블 매핑

![그림1](https://sehwan-choi.github.io/assets/img/spring/JPA-MAPPING4/jpa20.jpg)

위 사진과 같이 Member객체 안에는 Period, Address 객체로 Embedded형식으로 값이 들어가있지만 실제 Table에는 따로 분산되지 않고 Member 테이블안에 모두 들어있게 된다.

- 임베디드 타입은 엔티티의 값일 뿐이다. 
- 임베디드 타입을 사용하기 전과 후에 매핑하는 테이블은 같다. 
- 객체와 테이블을 아주 세밀하게(find-grained) 매핑하는 것이 가능
- 잘 설계한 ORM 애플리케이션은 매핑한 테이블의 수보다 클래스의 수가 더 많음

<br>

## 임베디드 타입과 연관관계

![그림1](https://sehwan-choi.github.io/assets/img/spring/JPA-MAPPING4/jpa17.jpg)

위 사진과 같이 Member안에 Address Embedded객체는 또 다른 Embedded객체(Zipcode)를 가질수 있다. <br>
또한, PhoneNumber Embedded객체는 다른 Entity 객체를 가질수 있다.

<br>

## 중복된 임베디드 값 사용

- 한 엔티티에서 같은 값 타입을 사용하면? 
- 컬럼 명이 중복됨
- @AttributeOverrides, @AttributeOverride를 사용해서 컬럼명 속성을 재정의

```java

@Entity
public class Member{

    @Id
    @GeneratedValue
    @Column(name = "MEMBER_ID")
    private Long id;

    private String username;

    @Embedded
    private Period period;

    @Embedded
    private Address homeAddress;

    @Embedded   //  homeAddress와 컬렁명 중복!!!
    private Address workAddress;

    ...
}
```

위 코드 실행시 'Repeated column in mapping for entity' Exception을 보게 될것이다. 왜냐하면 Member 테이블에는 이미 homeAdress의 city, street, zipcode 컬럼명이 있는데, 또 workAddress의 city, street, zipcode를 추가하려고 했기 때문에 컬럼 중복이 발생한 것 이다. <br>

이럴 때는 @AttributeOverrides, @AttributeOverride를 사용해서 컬럼명 속성을 재정의 한다.

```java
@Entity
public class Member{

    @Id
    @GeneratedValue
    @Column(name = "MEMBER_ID")
    private Long id;

    private String username;

    @Embedded
    private Period period;

    @Embedded
    private Address homeAddress;

    @Embedded
    @AttributeOverrides({
            @AttributeOverride(name="city",                 // Address의 city컬럼명을
                    column=@Column(name = "work_city")),    // work_city로 변경
            @AttributeOverride(name="street",               // Address의 street 컬럼명을
                    column=@Column(name = "work_street")),  // work_street으로 변경
            @AttributeOverride(name="zipcode",              // Address의 zipcode 컬럼명을
                    column=@Column(name = "work_zipcode"))  // work_zipcode로 변경 
    })
    private Address workAddress;

    ...
}
```

- 실행결과

![그림1](https://sehwan-choi.github.io/assets/img/spring/JPA-MAPPING4/jpa18.jpg)

위 사진과 같이 workAddress에 @AttributeOverrides, @AttributeOverride속성을 적용한것 처럼 work_city, work_street, work_zipcode 가 DB에 생성되었음을 알 수 있다.

<br>

## 임베디드 타입과 null

- 임베디드 타입의 값이 null이면 매핑한 컬럼 값은 모두 null

```java
public class jpaMain {

    public static void main(String[] args) {

        ...

        try {

            Member member = new Member();

            member.setUsername("NAME");
            member.setPeriod(new Period(LocalDateTime.now(),LocalDateTime.now()));
            member.setHomeAddress(new Address("CITY","STREET","ZIPCODE"));
            //member.setWorkAddress(new Address("WORK_CITY","WORK_STREET","WORK_ZIPCODE"));
            member.setWorkAddress(null);

            em.persist(member);

            ...
        }
    }
}
```

- 실행결과

![그림1](https://sehwan-choi.github.io/assets/img/spring/JPA-MAPPING4/jpa19.jpg)

위 사진과 같이 workAddress에 null을 set한다면 workAddress 하위의 work_city, work_street, work_zipcode는 모두 null로 들어가게 된다.

<br><br><br>
## References 및 사진 출처

> __김영한의 자바 ORM 표준 JPA 프로그래밍 - 기본편__