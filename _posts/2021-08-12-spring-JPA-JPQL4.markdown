---
layout: post
title:  "spring JPA JPQL Join SubQuery Type표현"
subtitle:   "spring JPA JPQL Join SubQuery Type표현"
date:   2021-08-27 02:00:27 +0900
categories: spring
tags: spring JPA ORM Mapping JPQL Join SubQuery Type표현
comments: true
---


<br>

- 목차
	- [서브쿼리](#서브쿼리)
	    - [서브 쿼리 지원 함수](#서브-쿼리-지원-함수)
	    - [JPA 서브 쿼리 한계](#jpa-서브-쿼리-한계)
	- [JPQL 타입 표현](#jpql-타입-표현)
	- [JPQL 기타](#jpql-기타)
    
<br>

# 서브쿼리

- 나이가 평균보다 많은 회원

```java
String Query = "select m from Member m where m.age > (select avg(m2.age) from Member m2) ";
List<Member> resultList = em.createQuery(Query, Member.class)
            .getResultList();
```


- 한 건이라도 주문한 고객

```java
String Query = "select m from Member m where (select count(o) from Order o where m = o.member) > 0 ";
List<Member> resultList = em.createQuery(Query, Member.class)
            .getResultList();
```

<br>

## 서브 쿼리 지원 함수
- [NOT] EXISTS (subquery): 서브쿼리에 결과가 존재하면 참

```java
String Query = "select m from Member m where exists (select t from m.team t where t.name = ‘팀A') ";
List<Member> resultList = em.createQuery(Query, Member.class)
            .getResultList();
```

- {ALL | ANY | SOME} (subquery) 
    - ALL 모두 만족하면 참
    - ANY, SOME: 같은 의미, 조건을 하나라도 만족하면 참
    
```java
// ALL
String Query = "select o from Order o where o.orderAmount > ALL (select p.stockAmount from Product p)";
List<Member> resultList = em.createQuery(Query, Member.class)
            .getResultList();

// ANY SOME
String Query = "select o from Order o where o.orderAmount > ALL (select p.stockAmount from Product p)";
List<Member> resultList = em.createQuery(Query, Member.class)
            .getResultList();
```

- [NOT] IN (subquery): 서브쿼리의 결과 중 하나라도 같은 것이 있으면 참

<br>

## JPA 서브 쿼리 한계

- JPA는 WHERE, HAVING 절에서만 서브 쿼리 사용 가능
- SELECT 절도 가능(하이버네이트에서 지원) 
- FROM 절의 서브 쿼리는 현재 JPQL에서 불가능
- 조인으로 풀 수 있으면 풀어서 해결

<br>

# JPQL 타입 표현

- 문자: ‘HELLO’, ‘She’’s’ 
- 숫자: 10L(Long), 10D(Double), 10F(Float) 
- Boolean: TRUE, FALSE 

```java
List<Object[]> resultList = em.createQuery("select m, 'HELLO', TRUE, 10D, 10L, 10F from Member m ")
    .getResultList();

Object[] objects = resultList.get(0);
System.out.println("objects[0] = " + objects[0]);
System.out.println("objects[1] = " + objects[1]);   // 'HELLO'
System.out.println("objects[2] = " + objects[2]);   //  'true'
System.out.println("objects[3] = " + objects[3]);   //  10.0(DOUBLE)
System.out.println("objects[4] = " + objects[4]);   //  10
System.out.println("objects[4] = " + objects[5]);   //  10.0(FLOAT)
```

- ENUM: jpabook.MemberType.Admin (패키지명 포함) 

```java

package jpql.Domain;

public enum MemberType {
    ADMIN, USER
}

public class JpqlTypeMain {

    public static void main(String[] args) {
        ...
        List<Member> resultList = em.createQuery("select m from Member m where m.type = jpql.Domain.MemberType.ADMIN", Member.class)
            .getResultList();
        ...
    }
}
```

- 엔티티 타입: TYPE(m) = Member (상속 관계에서 사용)

```java
@Entity
public class Album extends Item{

    private String album;
    ...
}

@Entity
public class Book extends Item{

    private String book;
    ...
}

@Entity
public class Movie  extends Item {

    private String movie;
    ...
}

@Entity
@Inheritance(strategy = InheritanceType.SINGLE_TABLE)
public abstract class Item{

    @Id
    @GeneratedValue
    @Column(name = "ITEM_ID")
    private Long id;

    private String name;
    ...
}

public class JpqlTypeMain {

    public static void main(String[] args) {
        ...
        Book book = new Book();
        book.setBook("BOOK");
        book.setName("BOOK_NAME");
        em.persist(book);

        Album album = new Album();
        album.setAlbum("ALBUM");
        album.setName("ALBUM_NAME");
        em.persist(album);

        Movie movie = new Movie();
        movie.setMovie("MOVIE");
        movie.setName("MOVIE_NAME");
        em.persist(movie);
        
        // 상속 관계에서 Book 타입만 조회할경우 TYPE(i) = Book, Album 타입만 조회할 경우 TYPE(i) = Album 으로 사용한다.
        List<Item> resultList = em.createQuery("select i from Item i where TYPE(i) = Book ", Item.class)
                .getResultList();
        ...
    }
}

SQL: 
select
        i 
    from
        Item i 
    where
        TYPE(i) = Book  */ select
            item0_.ITEM_ID as item_id2_0_,
            item0_.name as name3_0_,
            item0_.movie as movie4_0_,
            item0_.book as book5_0_,
            item0_.album as album6_0_,
            item0_.DTYPE as dtype1_0_ 
        from
            Item item0_ 
        where
            item0_.DTYPE='Book'    <-- Type(i) = 에 정의 했던 타입으로 쿼리가 나간다.
```

<br>

# JPQL 기타
- 아래와 같이 SQL과 문법이 같은 식 사용 가능
    - EXISTS, IN 
    - AND, OR, NOT 
    - =, >, >=, <, <=, <> 
    - BETWEEN, LIKE, IS NULL


<br><br><br>
## References 및 사진 출처

> __김영한의 자바 ORM 표준 JPA 프로그래밍 - 기본편__