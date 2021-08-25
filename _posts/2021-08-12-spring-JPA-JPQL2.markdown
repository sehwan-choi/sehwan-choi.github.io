---
layout: post
title:  "spring JPA JPQL 프로젝션과 페이징"
subtitle:   "spring JPA JPQL 프로젝션과 페지이"
date:   2021-08-25 21:00:27 +0900
categories: spring
tags: spring JPA ORM Mapping JPQL 프로젝션 페이징
comments: true
---


<br>

- 목차
	- [프로젝션](#프로젝션)
	    - [프로젝션 - 여러 값 조회](#프로젝션---여러-값-조회)
	- [페이징](#페이징)
    	- [방언](#방언)
    
<br>

# 프로젝션

![그림1](https://sehwan-choi.github.io/assets/img/spring/JPQL/jpa1.jpg)

- SELECT 절에 조회할 대상을 지정하는 것
- 프로젝션 대상: 엔티티, 임베디드 타입, 스칼라 타입(숫자, 문자등 기본 데이터 타
입) 
- SELECT m FROM Member m -> 엔티티 프로젝션
- SELECT m.team FROM Member m -> 엔티티 프로젝션
- SELECT m.address FROM Member m -> 임베디드 타입 프로젝션
- SELECT m.username, m.age FROM Member m -> 스칼라 타입 프로젝션
- DISTINCT로 중복 제거

<br>

## 프로젝션 - 여러 값 조회
- SELECT m.username, m.age FROM Member m 
- 1. Query 타입으로 조회

```java
List resultList = em.createQuery("select m.username, m.age from Member as m").getResultList();
Object o = resultList.get(0);
Object[] result = (Object[])o;
System.out.println("result[0] = " + result[0]);
System.out.println("result[1] = " + result[1]);
```

- 2. Object[] 타입으로 조회

```java
List<Object[]> resultList = em.createQuery("select m.username, m.age from Member as m").getResultList();
Object[] result = resultList.get(0);
System.out.println("result[0] = " + result[0]);
System.out.println("result[1] = " + result[1]);
```

- 3. new 명령어로 조회

```java
public class MemberDTO {

    private String username;
    private int age;

    public String getUsername() {
        return username;
    }

    public MemberDTO(String username, int age) {
        this.username = username;
        this.age = age;
    }
    ...
}


// new 키워드 뒤에 MemberDTO가 있는 경로는 적어준다.
List<MemberDTO> resultList = em.createQuery("select new jpql.Domain.MemberDTO(m.username, m.age) from Member as m", MemberDTO.class).getResultList();
            for (MemberDTO memberDTO : resultList) {
                System.out.println("memberDTO.getUsername() = " + memberDTO.getUsername());
            }
```

    - 단순 값을 DTO로 바로 조회
    - 패키지 명을 포함한 전체 클래스 명 입력
    - 순서와 타입이 일치하는 생성자 필요

<br>

# 페이징

- JPA는 페이징을 다음 두 API로 추상화
- setFirstResult(int startPosition) : 조회 시작 위치(0부터 시작) 
- setMaxResults(int maxResult) : 조회할 데이터 수

```java
List<Member> resultList = em.createQuery("select m from Member m order by m.age desc", Member.class)
                    .setFirstResult(50)
                    .setMaxResults(10)
                    .getResultList();
```

<br>

## 방언

- MYSQL 방언

```java
SELECT
 M.ID AS ID,
 M.AGE AS AGE,
 M.TEAM_ID AS TEAM_ID,
 M.NAME AS NAME 
FROM
 MEMBER M 
ORDER BY
 M.NAME DESC LIMIT ?, ?
 ```

 - Oracle 방언

 ```java
 SELECT * FROM
    ( SELECT ROW_.*, ROWNUM ROWNUM_ 
    FROM
        ( SELECT
            M.ID AS ID,
            M.AGE AS AGE,
            M.TEAM_ID AS TEAM_ID,
            M.NAME AS NAME 
            FROM MEMBER M 
            ORDER BY M.NAME 
        ) ROW_ 
        WHERE ROWNUM <= ?
    ) 
WHERE ROWNUM_ > ?
```

persistence.xml 에 ```<property name="hibernate.dialect" value="org.hibernate.dialect.H2Dialect"/>``` 어떤값을 넣느냐에 따라서 Oracle, MYSQL, MSSQL 등의 문법으로 방언에 맞게 쿼리를 만든다. <br>

- Oracle <br>
``` <property name="hibernate.dialect" value="org.hibernate.dialect.Oracle12cDialect"/> ```

- MSSQL SERVER 2012
``` <property name="hibernate.dialect" value="org.hibernate.dialect.SQLServer2012Dialect"/> ```

- MYSQL
``` <property name="hibernate.dialect" value="org.hibernate.dialect.MySQLDialect"/> ```

<br><br><br>
## References 및 사진 출처

> __김영한의 자바 ORM 표준 JPA 프로그래밍 - 기본편__