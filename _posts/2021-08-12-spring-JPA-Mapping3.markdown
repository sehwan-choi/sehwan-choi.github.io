---
layout: post
title:  "spring JPA 상속관계 맵핑"
subtitle:   "spring JPA 상속관계 맵핑"
date:   2021-08-15 14:00:27 +0900
categories: spring
tags: spring JPA ORM Mapping MappedSuperclass
comments: true
---


<br>

- 목차
	- [](#)
<br>

# 상속관계 맵핑

![그림1](https://sehwan-choi.github.io/assets/img/spring/JPA-MAPPING3/jpa1.jpg)

- 관계형 데이터베이스는 상속 관계X 
- 슈퍼타입 서브타입 관계라는 모델링 기법이 객체 상속과 유사
- 상속관계 매핑: 객체의 상속과 구조와 DB의 슈퍼타입 서브타입 관계를 매핑

- 슈퍼타입 서브타입 논리 모델을 실제 물리 모델로 구현하는 방법
    - 각각 테이블로 변환 -> 조인 전략
    - 통합 테이블로 변환 -> 단일 테이블 전략
    - 서브타입 테이블로 변환 -> 구현 클래스마다 테이블 전략

<br>

## 주요 어노테이션 

- @Inheritance(strategy=InheritanceType.XXX) 
    - JOINED: 조인 전략
    - SINGLE_TABLE: 단일 테이블 전략
    - TABLE_PER_CLASS: 구현 클래스마다 테이블 전략
- @DiscriminatorColumn(name=“DTYPE”) 
- @DiscriminatorValue(“XXX”)

<br>

# 조인전략

![그림1](https://sehwan-choi.github.io/assets/img/spring/JPA-MAPPING3/jpa2.jpg)

- 장점
    - 테이블 정규화
    - 외래 키 참조 무결성 제약조건 활용가능
    - 저장공간 효율화
- 단점
    - 조회시 조인을 많이 사용, 성능 저하
    - 조회 쿼리가 복잡함
    - 데이터 저장시 INSERT SQL 2번 호출

<br>

```java
@Entity
@Inheritance(strategy = InheritanceType.JOINED)
@DiscriminatorColumn
public class Item {

    @Id
    @GeneratedValue
    @Column(name = "ITEM_ID")
    private Long id;

    private String name;

    private int price;
}

@Entity
@DiscriminatorValue("A")
public class Album extends Item{

    private String artist;

    public String getArtist() {
        return artist;
    }

    public void setArtist(String artist) {
        this.artist = artist;
    }
}

@Entity
@DiscriminatorValue("B")
public class Book extends Item{

    private String author;
    private String isbn;
}

@Entity
@DiscriminatorValue("M")
public class Movie extends Item{

    private String director;
    private String actor;
}
```
@Inheritance(strategy = InheritanceType.JOINED) 로 하면 조인 테이블 전략을 사용할수 있다.

<br>

- @DiscriminatorColumn 사용하지 않았을 경우 DB의 Item 컬럼

![그림1](https://sehwan-choi.github.io/assets/img/spring/JPA-MAPPING3/jpa3.jpg)

<br>

- @DiscriminatorColumn 사용했을 경우 DB의 Item 컬럼

![그림1](https://sehwan-choi.github.io/assets/img/spring/JPA-MAPPING3/jpa4.jpg)

Item이 Album, Book, Movie중 어떤것으로 만들어 졌는지 확인할 수 있으며 DB상에서는 DTYPE으로 생성된다. @DiscriminatorColumn(name = "DATA_TYPE") 로 name 속성을 추가해준다면 다른 이름으로 변경이 가능하다. <br>
또한, DTYPE에 movie, album, book이 아닌 다른값으로 설정하고 싶다면 Item을 상속 받은 클래스에 @DiscriminatorValue 어노테이션을 사용하면 된다. 위의 코드에서 Book은 @DiscriminatorValue("B") Album은 @DiscriminatorValue("A") Movie는 @DiscriminatorValue("M") 로 설정했기 때문에 Item 컬럼은 DTYPE에는 아래와 같이 "M", "A", "B" 형태로 들어가게 된다.

<br>

![그림1](https://sehwan-choi.github.io/assets/img/spring/JPA-MAPPING3/jpa5.jpg)

<br>

# 단일 테이블 전략

![그림1](https://sehwan-choi.github.io/assets/img/spring/JPA-MAPPING3/jpa6.jpg)

- 장점
    - 조인이 필요 없으므로 일반적으로 조회 성능이 빠름
    - 조회 쿼리가 단순함
- 단점
    - 자식 엔티티가 매핑한 컬럼은 모두 null 허용
    - 단일 테이블에 모든 것을 저장하므로 테이블이 커질 수 있다. 상황에 따라서 조회 성능이 오히려 느려질 수 있다.

<br>

```java
@Entity
@Inheritance(strategy = InheritanceType.SINGLE_TABLE)
//@DiscriminatorColumn <- 없어도 DTYPE이 자동생성됨
public class Item {

    @Id
    @GeneratedValue
    @Column(name = "ITEM_ID")
    private Long id;

    private String name;

    private int price;
}

@Entity
@DiscriminatorValue("A")
public class Album extends Item{

    private String artist;

    public String getArtist() {
        return artist;
    }

    public void setArtist(String artist) {
        this.artist = artist;
    }
}

@Entity
@DiscriminatorValue("B")
public class Book extends Item{

    private String author;
    private String isbn;
}

@Entity
@DiscriminatorValue("M")
public class Movie extends Item{

    private String director;
    private String actor;
}
```
@Inheritance(strategy = InheritanceType.SINGLE_TABLE) 로 변경하면 싱글테이블 전략을 사용할수 있다.

<br>

# 구현 클래스마다 테이블 전략

![그림1](https://sehwan-choi.github.io/assets/img/spring/JPA-MAPPING3/jpa7.jpg)

- 이 전략은 데이터베이스 설계자와 ORM 전문가 둘 다 추천X 
- 장점
    - 서브 타입을 명확하게 구분해서 처리할 때 효과적
    - not null 제약조건 사용 가능
- 단점
    - 여러 자식 테이블을 함께 조회할 때 성능이 느림(UNION SQL 필요, 아래에서 설명) 
    - 자식 테이블을 통합해서 쿼리하기 어려움

<br>

```java
@Entity
@Inheritance(strategy = InheritanceType.TABLE_PER_CLASS)
public abstract class Item {

    @Id
    @GeneratedValue
    @Column(name = "ITEM_ID")
    private Long id;

    private String name;

    private int price;
}

@Entity
public class Album extends Item{

    private String artist;

    public String getArtist() {
        return artist;
    }

    public void setArtist(String artist) {
        this.artist = artist;
    }
}

@Entity
public class Book extends Item{

    private String author;
    private String isbn;
}

@Entity
public class Movie extends Item{

    private String director;
    private String actor;
}
```
@Inheritance(strategy = InheritanceType.TABLE_PER_CLASS) 로 변경하면 구현 클래스마다 테이블 전략을 사용할수 있다. <br>
위 코드 실행시 Album, Book, Movie 컬럼이 생성된다.(Item 컬럼은 생성되지 않음)

<br>

- 단점
    - 여러 자식 테이블을 함께 조회할 때 성능이 느림(UNION SQL 필요) 

```java
    Movie movie = new Movie();
    movie.setDirector("Director");
    movie.setActor("Actor");
    movie.setName("movie");
    movie.setPrice(10000);
    em.persist(movie);

    em.flush();
    em.clear();

    Item item = em.find(Item.class, movie.getId());
```
위코드 실행시 아래와 같이 쿼리문이 생성되며 Album, Movie, Book 테이블을 Union으로 가져오는 것을 알수있다.

```
Hibernate: 
    select
        item0_.ITEM_ID as item_id1_5_0_,
        item0_.name as name2_5_0_,
        item0_.price as price3_5_0_,
        item0_.actor as actor1_7_0_,
        item0_.director as director2_7_0_,
        item0_.author as author1_1_0_,
        item0_.isbn as isbn2_1_0_,
        item0_.artist as artist1_0_0_,
        item0_.clazz_ as clazz_0_ 
    from
        ( select
            ITEM_ID,
            name,
            price,
            actor,
            director,
            null as author,
            null as isbn,
            null as artist,
            1 as clazz_ 
        from
            Movie 
        union
        all select
            ITEM_ID,
            name,
            price,
            null as actor,
            null as director,
            author,
            isbn,
            null as artist,
            2 as clazz_ 
        from
            Book 
        union
        all select
            ITEM_ID,
            name,
            price,
            null as actor,
            null as director,
            null as author,
            null as isbn,
            artist,
            3 as clazz_ 
        from
            Album 
    ) item0_ 
where
    item0_.ITEM_ID=?
```

<br>

# @MappedSuperclass

![그림1](https://sehwan-choi.github.io/assets/img/spring/JPA-MAPPING3/jpa8.jpg)

- 공통 매핑 정보가 필요할 때 사용(id, name)
- 상속관계 매핑X 
- 엔티티X, 테이블과 매핑X 
- 부모 클래스를 상속 받는 자식 클래스에 매핑 정보만 제공
- 조회, 검색 불가(em.find(BaseEntity) 불가) 
- 직접 생성해서 사용할 일이 없으므로 추상 클래스 권장
- 테이블과 관계 없고, 단순히 엔티티가 공통으로 사용하는 매핑
정보를 모으는 역할
- 주로 등록일, 수정일, 등록자, 수정자 같은 전체 엔티티에서 공통
으로 적용하는 정보를 모을 때 사용
- 참고: @Entity 클래스는 엔티티나 @MappedSuperclass로 지
정한 클래스만 상속 가능

<br>

```java
@MappedSuperclass
public abstract class BaseEntity {

    private String createBy;
    private LocalDateTime createdDate;
    private String lastModifiedBy;
    private LocalDateTime lastModifiedDate;
}

@Entity
public class Member extends BaseEntity{

    @Id
    @GeneratedValue
    @Column(name = "MEMBER_ID")
    private Long id;

    private String name;

    private String city;

    private String street;

    private String zipcode;
}

@Entity
@Inheritance(strategy = InheritanceType.JOINED)
public class Item extends BaseEntity{

    @Id
    @GeneratedValue
    @Column(name = "ITEM_ID")
    private Long id;

    private String name;

    private int price;
}

```
Member, Item클래스는 BaseEntity클래스의 createBy, createdDate, lastModifiedBy, lastModifiedDate 속성만 받아서 사용하게 된다(Entity 맵핑X)

![그림1](https://sehwan-choi.github.io/assets/img/spring/JPA-MAPPING3/jpa9.jpg)

![그림1](https://sehwan-choi.github.io/assets/img/spring/JPA-MAPPING3/jpa10.jpg)

<br><br><br>
## References 및 사진 출처

> __김영한의 자바 ORM 표준 JPA 프로그래밍 - 기본편__
https://gmlwjd9405.github.io/2019/08/03/reason-why-use-jpa.html