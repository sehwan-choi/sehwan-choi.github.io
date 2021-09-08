---
layout: post
title:  "spring JPA JQPL 다형성 쿼리"
subtitle:   "spring JPA JPQL 다형성 쿼리"
date:   2021-09-09 03:28:27 +0900
categories: spring
tags: spring JPA ORM Mapping JPQL 다형성 쿼리
comments: true
---


<br>

- 목차
	- [TYPE](#type)
    - [TREAT](#treat)
    
<br>

# TYPE

- 조회 대상을 특정 자식으로 한정
    - 예) Item 중에 Book, Movie를 조회해라

```
[JPQL]
select i from Item i
where type(i) IN (Book, Movie) 

[SQL]
select i from i
where i.DTYPE in (‘B’, ‘M’)
```

```java
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

@Entity
@DiscriminatorValue("A")
public class Album extends Item{

    private String album;
    ...
}

@Entity
@DiscriminatorValue("B")
public class Book extends Item{

    private String book;
    ...
}

@Entity
@DiscriminatorValue("M")
public class Movie  extends Item {

    private String movie;
    ...
}

public class PolymorphismMain {

    public static void main(String[] args) {
        ...
        try {
            
            Book book = new Book();
            book.setName("this is book");
            em.persist(book);

            Movie movie = new Movie();
            movie.setName("this is movie");
            em.persist(movie);

            Album album = new Album();
            album.setName("this is movie");
            em.persist(album);
            
            em.flush();
            em.clear();
            
            String query = "select i from Item i where type(i) IN (Book, Movie)";
            List<Item> resultList = em.createQuery(query, Item.class).getResultList();

            for (Item item : resultList) {
                System.out.println("item.getName() = " + item.getName());
            }
        ...
        }
    }
}
```

- 결과

```
item.getName() = this is book
item.getName() = this is movie
```

<br>

# TREAT

- 타입캐스팅과 같음.

```
[JPQL]
select i from Item i
where treat(i as Book).auther = ‘kim’ 

[SQL]
select i.* from Item i
where i.DTYPE = ‘B’ and i.auther = ‘kim’
```

```
public class PolymorphismMain {

    public static void main(String[] args) {
        ...
        try {
            
            Book book = new Book();
            book.setName("this is book");
            book.setBook("book");
            em.persist(book);

            Movie movie = new Movie();
            movie.setName("this is movie");
            movie.setMovie("movie");
            em.persist(movie);

            Album album = new Album();
            album.setName("this is movie");
            album.setAlbum("album");
            em.persist(album);
            
            em.flush();
            em.clear();

            /**
             * TREAT
             */
            String query = "select i from Item i where treat(i as Book).book = 'book'";
            List<Item> resultList = em.createQuery(query, Item.class).getResultList();

            for (Item item : resultList) {
                System.out.println("item.getName() = " + item.getName());
            }
        ...
        }
    }
}
```

- 결과

```
item.getName() = this is book
```

<br><br><br>
## References 및 사진 출처

> __김영한의 자바 ORM 표준 JPA 프로그래밍 - 기본편__