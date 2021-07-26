---
layout: post
title:  "JPA 엔티티 맵핑"
subtitle:   "JPA 엔티티 맵핑"
date:   2021-07-27 03:42:27 +0900
categories: spring
tags: spring JPA Entity EntityMapping
comments: true
---

# JPA 엔티티 맵핑

<br>

- 목차
	- [JPA 엔티티 맵핑](#jpa-엔티티-맵핑)
	- [데이터베이스 스키마 자동 생성](#데이터베이스-스키마-자동-생성)
	    - [데이터베이스 스키마 자동 생성 - 속성](#데이터베이스-스키마-자동-생성---속성)
	    - [데이터베이스 스키마 자동 생성 - 주의](#데이터베이스-스키마-자동-생성---주의)
	    - [DDL 생성 기능](#ddl-생성-기능)
	- [객체와 테이블 매핑](#객체와-테이블-매핑)
	    - [@Entity](#entity)
	    - [@Entity 속성 정리](#entity-속성-정리)
	- [필드와 컬럼 매핑](#필드와-컬럼-매핑)
	    - [매핑 어노테이션 정리](#매핑-어노테이션-정리)
	- [기본 키 매핑](#기본-키-매핑)
        - [기본 키 매핑 어노테이션](#기본-키-매핑-어노테이션)
        - [기본 키 매핑 방법](#기본-키-매핑-방법)
<br>

<br>

# 데이터베이스 스키마 자동 생성

- DDL을 애플리케이션 실행 시점에 자동 생성
- 테이블 중심 -> 객체 중심
- 데이터베이스 방언을 활용해서 데이터베이스에 맞는 적절한
- DDL 생성
- 이렇게 생성된 DDL은 개발 장비에서만 사용
- 생성된 DDL은 운영서버에서는 사용하지 않거나, 적절히 다듬은 후 사용

<br>

## 데이터베이스 스키마 자동 생성 - 속성

```xml
<property name="hibernate.hbm2ddl.auto" value="create" />
<property name="hibernate.hbm2ddl.auto" value="create-drop" />
<property name="hibernate.hbm2ddl.auto" value="update" />
<property name="hibernate.hbm2ddl.auto" value="validate" />
<property name="hibernate.hbm2ddl.auto" value="none" />
```

![그림1](https://sehwan-choi.github.io/assets/img/spring/JPA-BASIC2/jpa11.jpg)

<br>

## 데이터베이스 스키마 자동 생성 - 주의

- __운영 장비에는 절대 create, create-drop, update 사용하면 안된다.__
- 개발 초기 단계는 create 또는 update 을 사용한다.
- 테스트 서버는 update 또는 validate 을 사용한다.
- 스테이징과 운영 서버는 validate 또는 none 을 사용한다.

<br>

## DDL 생성 기능

- 제약조건 추가: 회원 이름은 필수, 10자 초과X 
    - @Column(nullable = false, length = 10) 
- 유니크 제약조건 추가
    - @Table(uniqueConstraints = {@UniqueConstraint( name = "NAME_AGE_UNIQUE",
 columnNames = {"NAME", "AGE"} )}) 
- DDL 생성 기능은 DDL을 자동 생성할 때만 사용되고
JPA의 실행 로직에는 영향을 주지 않는다.

<br><br>

# 객체와 테이블 매핑

<br>

## @Entity

- @Entity가 붙은 클래스는 JPA가 관리, 엔티티라 한다. 
- JPA를 사용해서 테이블과 매핑할 클래스는 @Entity 필수
- __주의__
    - 기본 생성자 필수(파라미터가 없는 public 또는 protected 생성자) 
    - final 클래스, enum, interface, inner 클래스 사용X 
    - 저장할 필드에 final 사용 X

<br>

## @Entity 속성 정리

- @ name 
    - JPA에서 사용할 엔티티 이름을 지정한다. 
    - 기본값: 클래스 이름을 그대로 사용(예: Member) 
    - 같은 클래스 이름이 없으면 가급적 기본값을 사용한다.
```java
@Entity(name = "TEST")
public class Member {
    ...
}
```

- @Table
    - @Table은 엔티티와 매핑할 테이블 지정
```java
@Entity
@Table(name = "tbl_Member")
public class Member {
    ...
}
```

![그림1](https://sehwan-choi.github.io/assets/img/spring/JPA-BASIC2/jpa12.jpg)

<br>

# 필드와 컬럼 매핑

<br>

## 매핑 어노테이션 정리

```java
@Entity 
public class Member { 
    @Id 
    private Long id; 
 
    @Column(name = "name") 
    private String username; 
 
    private Integer age; 
 
    @Enumerated(EnumType.STRING) 
    private RoleType roleType; 
 
    @Temporal(TemporalType.TIMESTAMP)   //  과거 버전에서 사용
    private Date createdDate; 
 
    private LocalDateTime lastModifiedDate; //  최신 Hibernate에서는 LocalDateTime사용

    @Lob 
    private String description; 
 //Getter, Setter… 
} 
```

- @Column
    - 컬럼 맵핑

![그림1](https://sehwan-choi.github.io/assets/img/spring/JPA-BASIC2/jpa13.jpg)

<br>

- @Temporal
    - 날짜 타입(java.util.Date, java.util.Calendar)을 매핑할 때 사용
    - 참고: LocalDate, LocalDateTime을 사용할 때는 생략 가능(최신 하이버네이트 지원)

![그림1](https://sehwan-choi.github.io/assets/img/spring/JPA-BASIC2/jpa15.jpg)

<br>

- @Enumerated
    - 자바 enum 타입을 매핑할 때 사용
    - __주의! EnumType.ORDINAL 사용 X__

![그림1](https://sehwan-choi.github.io/assets/img/spring/JPA-BASIC2/jpa14.jpg)

<br>

- @Lob
    - 데이터베이스 BLOB, CLOB 타입과 매핑 
    - @Lob에는 지정할 수 있는 속성이 없다. 
    - 매핑하는 필드 타입이 문자면 CLOB 매핑, 나머지는 BLOB 매핑
    - CLOB: String, char[], java.sql.CLOB 
    - BLOB: byte[], java.sql. BLOB

<br>

- @Transient
    - 필드 매핑X 
    - 데이터베이스에 저장X, 조회X 
    - 주로 메모리상에서만 임시로 어떤 값을 보관하고 싶을 때 사용
```java
@Transient
private Integer temp; 
```

<br><br>

# 기본 키 매핑

<br>

## 기본 키 매핑 어노테이션

- @Id
- @GeneratedValue
```java
@Id @GeneratedValue(strategy = GenerationType.AUTO)
private Long id;
```

<br>

## 기본 키 매핑 방법

- 직접 할당
    - @Id를 사용하여 기본 키를 직접 set한다
```java
Member member = new Member();
member.setId(1L);
```

<br>

- IDENTITY
    - 기본 키 생성을 데이터베이스에 위임
    - 주로 MySQL, PostgreSQL, SQL Server, DB2에서 사용
        (예: MySQL의 AUTO_ INCREMENT) 
    - JPA는 보통 트랜잭션 커밋 시점에 INSERT SQL 실행
    - AUTO_ INCREMENT는 데이터베이스에 INSERT SQL을 실행한 이후에 ID 값을 알 수 있음
    - __IDENTITY 전략은 em.persist() 시점에 즉시 INSERT SQL 실행하고 DB에서 식별자를 조회__
```java
@Entity 
public class Member { 
 @Id 
 @GeneratedValue(strategy = GenerationType.IDENTITY) 
 private Long id;
 ```

 <br>

 - SEQUENCE
    - 데이터베이스 시퀀스는 유일한 값을 순서대로 생성하는 특별한 데이터베이스 오브젝트(예: 오라클 시퀀스) 
    - 오라클, PostgreSQL, DB2, H2 데이터베이스에서 사용

```java
@Entity 
@SequenceGenerator( 
    name = “MEMBER_SEQ_GENERATOR", 
    sequenceName = “MEMBER_SEQ", //매핑할 데이터베이스 시퀀스 이름
    initialValue = 1, allocationSize = 1) 
public class Member { 

    @Id 
    @GeneratedValue(strategy = GenerationType.SEQUENCE, 
                    generator = "MEMBER_SEQ_GENERATOR") 
    private Long id;
 ```

 - SEQUENCE - @SequenceGenerator

![그림1](https://sehwan-choi.github.io/assets/img/spring/JPA-BASIC2/jpa16.jpg)

<br>

- TABLE
    - 키 생성 전용 테이블을 하나 만들어서 데이터베이스 시퀀스를 흉내내는 전략
    - 장점: 모든 데이터베이스에 적용 가능
    - 단점: 성능
```SQL
create table MY_SEQUENCES ( 
 sequence_name varchar(255) not null, 
 next_val bigint, 
 primary key ( sequence_name ) 
)
```

```java
@Entity 
@TableGenerator( 
     name = "MEMBER_SEQ_GENERATOR", 
    table = "MY_SEQUENCES", 
    pkColumnValue = "MEMBER_SEQ", allocationSize = 1) 
public class Member { 
 
    @Id 
    @GeneratedValue(strategy = GenerationType.TABLE, 
                    generator = "MEMBER_SEQ_GENERATOR") 
    private Long id;
```

- @TableGenerator - 속성

![그림1](https://sehwan-choi.github.io/assets/img/spring/JPA-BASIC2/jpa17.jpg)

<br>

<br><br><br>
## References 및 사진 출처

> __김영한의 자바 ORM 표준 JPA 프로그래밍 - 기본편__