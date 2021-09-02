---
layout: post
title:  "spring JPA JQPL 기본함수와 사용자 정의함수"
subtitle:   "spring JPA JPQL 기본함수와 사용자 정의함수"
date:   2021-09-02 02:30:27 +0900
categories: spring
tags: spring JPA ORM Mapping JPQL
comments: true
---


<br>

- 목차
	- [](#)
    
<br>

# JPQL 기본 함수

## CONCAT 

```java

public class JpqlNativeFuncMain {

    public static void main(String[] args) {
        ...
        Member member = new Member();
        member.setUsername("Member");
        em.persist(member);

        em.flush();
        em.clear();

        /**
         * Concat
         * 설명 : 문자열 이어붙이기
         * 결과 : 이어붙인 문자열
        */
        String query = "select concat('a','b') from Member m";
        //String query = "select 'a' || 'b' from Member m"; // concat대신 || 로 사용가능
        List<String> resultList = em.createQuery(query, String.class).getResultList();

        for (String s : resultList) {
            System.out.println("s = " + s);
        }
        ...
    }
}
```

- 결과

```
s = ab
```

<br>

## SUBSTRING 

```java

public class JpqlNativeFuncMain {

    public static void main(String[] args) {
        ...
        Member member = new Member();
        member.setUsername("Member");
        em.persist(member);

        em.flush();
        em.clear();
        /**
         * Substring
         * 설명 : 문자열 자르기
         * 결과 : 전체 문자열에서 n번째 부터 m개만큼 잘려진 문자열( substring(문자열, n,m) )
        */
        String query = "select substring(m.username, 2,3) from Member m";
        List<String> resultList = em.createQuery(query, String.class).getResultList();

        for (String s : resultList) {
            System.out.println("s = " + s);
        }
        ...
    }
}
```

- 결과
```
s = emb
```

<br>

## TRIM 

```java

public class JpqlNativeFuncMain {

    public static void main(String[] args) {
        ...
        /**
         * Trim
         * 설명 : 공백 제거
         * 결과 : 공백 제거된 문자열
        */
        Member trimMember = new Member();
        trimMember.setUsername("    trimMember");
        em.persist(trimMember);

        em.flush();
        em.clear();

        String query = "select trim(m.username) from Member m";
        List<String> resultList = em.createQuery(query, String.class).getResultList();

        for (String s : resultList) {
            System.out.println("s = " + s);
        }
        ...
    }
}
```

- 결과
```
s = trimMember
```

<br>

## LOWER 

```java

public class JpqlNativeFuncMain {

    public static void main(String[] args) {
        ...
        Member member = new Member();
        member.setUsername("Member");
        em.persist(member);

        em.flush();
        em.clear();

        /**
         * Lower
         * 설명 : 문자열을 소문자로 변환
         * 결과 : 소문자로 변환된 문자열 반환
        */
        String query = "select lower('ABCDE') from Member m";
        List<String> resultList = em.createQuery(query, String.class).getResultList();

        for (String s : resultList) {
            System.out.println("s = " + s);
        }
        ...
    }
}
```

- 결과
```
s = abcde
```

<br>

## UPPER 

```java

public class JpqlNativeFuncMain {

    public static void main(String[] args) {
        ...
        Member member = new Member();
        member.setUsername("Member");
        em.persist(member);

        em.flush();
        em.clear();
        /**
         * Upper
         * 설명 : 문자열을 대문자로 변환
         * 결과 : 대문자로 변환된 문자열 반환
        */
        String query = "select lower('abcde') from Member m";
        List<String> resultList = em.createQuery(query, String.class).getResultList();

        for (String s : resultList) {
            System.out.println("s = " + s);
        }
        ...
    }
}
```

- 결과
```
s = ABCDE
```

<br>

## LENGTH 

```java

public class JpqlNativeFuncMain {

    public static void main(String[] args) {
        ...
        Member member = new Member();
        member.setUsername("Member");
        em.persist(member);

        em.flush();
        em.clear();

        /**
         * Length
         * 설명 : 문자열 길이 구하기
         * 결과 : 문자열의 길이(Integer)
        */
        String query = "select length(m.username) from Member m";
        List<Integer> resultList = em.createQuery(query, Integer.class).getResultList();

        for (Integer s : resultList) {
            System.out.println("s = " + s);
        }
        ...
    }
}
```

- 결과
```
s = 6
```

<br>

## LOCATE 

```java

public class JpqlNativeFuncMain {

    public static void main(String[] args) {
        ...
        Member member = new Member();
        member.setUsername("Member");
        em.persist(member);

        em.flush();
        em.clear();

        /**
         * Locate
         * 설명 : 일치하는 문자열이 있는지 확인
         * 결과 : 일치하는 문자열이 위치하는 번째수(Integer), 일치하는 문자열이 없는경우 0 반환
        */
        String query = "select locate('de','abcdefg') from Member m";
        List<Integer> resultList = em.createQuery(query, Integer.class).getResultList();

        for (Integer s : resultList) {
            System.out.println("s = " + s);
        }
        ...
    }
}
```

- 결과
```
s = 4
```

<br>

## ABS 

```java

public class JpqlNativeFuncMain {

    public static void main(String[] args) {
        ...
        Member member = new Member();
        member.setUsername("Member");
        em.persist(member);

        em.flush();
        em.clear();

        /**
         * Abs
         * 설명 : 절대값
         * 결과 : 절대값 반환
        */
        String query = "select abs(-3) from Member m";
        List<Integer> resultList = em.createQuery(query, Integer.class).getResultList();

        for (Integer s : resultList) {
            System.out.println("s = " + s);
        }
        ...
    }
}
```

- 결과
```
s = 3
```

<br>

## SQRT 

```java

public class JpqlNativeFuncMain {

    public static void main(String[] args) {
        ...
        Member member = new Member();
        member.setUsername("Member");
        em.persist(member);

        em.flush();
        em.clear();

        /**
         * Sqrt
         * 설명 : 제곱근
         * 결과 : 제곱근 반환(Double)
        */
        String query = "select sqrt(16) from Member m";
        List<Double> resultList = em.createQuery(query, Double.class).getResultList();

        for (Double s : resultList) {
            System.out.println("s = " + s);
        }
        ...
    }
}
```

- 결과
```
s = 4.0
```

<br>

## MOD 

```java

public class JpqlNativeFuncMain {

    public static void main(String[] args) {
        ...
        Member member = new Member();
        member.setUsername("Member");
        em.persist(member);

        em.flush();
        em.clear();

        /**
         * Mod
         * 설명 : 나머지 구하기
         * 결과 : m을 n으로 나누어 남은 값을 반환한다. (Mod(m,n) 과 같이 사용)
        */
        String query = "select mod(5,3) from Member m";
        List<Integer> resultList = em.createQuery(query, Integer.class).getResultList();

        for (Integer s : resultList) {
            System.out.println("s = " + s);
        }
        ...
    }
}
```

- 결과
```
s = 2
```

<br>

## SIZE, INDEX(JPA 용도)

```java

public class JpqlNativeFuncMain {

    public static void main(String[] args) {
        ...
        Member member = new Member();
        member.setUsername("Member");

        Team team = new Team();
        team.setName("Team");
        member.addteam(team);

        em.persist(team);
        em.persist(member);

        em.flush();
        em.clear();

        /**
         * Size
         * 설명 : JPA가 지원하는 함수로 연관관계가 있는 엔티티의 개수를 확인할 수 있다.
         * 결과 : 연관관계가 있는 엔티티의 개수를 반환
        */
        String query = "select size(t.members) from Team t";
        List<Integer> resultList = em.createQuery(query, Integer.class).getResultList();

        for (Integer s : resultList) {
            System.out.println("s = " + s);
        }
        ...
    }
}
```

- 결과
```
s = 1
```

<br><br>

# 사용자 정의함수

사용자 정의함수란 예를 들어 now() 함수를 사용하고 싶은데, JPA는 DB종속 적이지 않기 때문에 now()라는 함수를 지원하지 않는다. <br> 
그렇기 때문에 직접 now()함수를 등록하여 사용할수 있다. <br>
ex). MySQL에서 지원하는 now()함수를 사용하고 싶은데 JPA 기본함수 에서는 now() 지원하지 않기 때문에, 직접 now()함수를 등록하여 사용한다.

<br>

> DB Dialect에서는 대부분의 함수를 지원한다.

<br>

아래 예제는 이미 등록되어있는 함수를 사용한다. <br>
등록되어 있는 함수는 각각의 DB의 Dialect를 들어가면 아래 사진과 같이 볼수 있다.

![그림1](https://sehwan-choi.github.io/assets/img/spring/JPQL/jpa2.jpg)

1. 사용자 정의 Dialect클래스를 생성한다.

```java
public class MyH2Dialect extends H2Dialect {

    public MyH2Dialect() {
        // H2에서 지원하는 group_concat 이라는 함수를 사용하기 위해 정의한다.
        // JPQL에서 group_concat 함수를 사용하면 H2 DB에 group_concat함수에 매칭시킨다.
        registerFunction("group_concat", new StandardSQLFunction("group_concat", StandardBasicTypes.STRING));

        // H2에서 지원하는 now 이라는 함수를 사용하기 위해 정의한다.
        // JPQL에서 now_date 함수를 사용하면 H2 DB에 now함수에 매칭시킨다.
        registerFunction("now_date", new StandardSQLFunction("now", StandardBasicTypes.TIMESTAMP));
    }
}
```

2. persistence.xml의 dialect수정

```xml
<?xml version="1.0" encoding="UTF-8"?>
<persistence version="2.2"
             xmlns="http://xmlns.jcp.org/xml/ns/persistence" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
             xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/persistence http://xmlns.jcp.org/xml/ns/persistence/persistence_2_2.xsd">
    <persistence-unit name="hello">
        <properties>
            <!-- 필수 속성 -->
            <property name="javax.persistence.jdbc.driver" value="org.h2.Driver"/>
            <property name="javax.persistence.jdbc.user" value="sa"/>
            <property name="javax.persistence.jdbc.password" value=""/>
            <property name="javax.persistence.jdbc.url" value="jdbc:h2:tcp://localhost/~/jpashop2"/>
            <!-- <property name="hibernate.dialect" value="org.hibernate.dialect.H2Dialect"/> -->
            <property name="hibernate.dialect" value="jpql.MyH2Dialect"/>   <!-- 이부분을 위에 MyH2Dialect 생성한것으로 변경 -->

            <!-- 옵션 -->
            <property name="hibernate.show_sql" value="true"/>
            <property name="hibernate.format_sql" value="true"/>
            <property name="hibernate.use_sql_comments" value="true"/>
            <property name="hibernate.jdbc.batch_size" value="10"/>
            <property name="hibernate.hbm2ddl.auto" value="create" />
        </properties>
    </persistence-unit>
</persistence>
```

3. 사용자 정의 클래스에서 정의한 함수 사용

- now_date() 함수 사용

```java
public class JpqlUserFuncMain {

    public static void main(String[] args) {
        ...
        Member member1 = new Member();
        member1.setUsername("관리자1");
        em.persist(member1);

        em.flush();
        em.clear();

        // now_date() -> now()
        String query = "select function('now_date') from Member m";
        //String query = "select now_date() from Member m"; // function 키워드 대신 그냥 함수 명으로 사용할 수 있다.
        List<Date> resultList = em.createQuery(query, Date.class).getResultList();

        for (Date s : resultList) {
            System.out.println("s = " + s);
        }
        ...
    }
}
```

- 결과

```
s = 2021-09-03 02:33:04.696147
```

<br>

- group_concat() 함수 사용

```java
public class JpqlUserFuncMain {

    public static void main(String[] args) {
        ...
        Member member1 = new Member();
        member1.setUsername("관리자1");
        em.persist(member1);

        Member member2 = new Member();
        member2.setUsername("관리자2");
        em.persist(member2);

        String query = "select function('group_concat',m.username) from Member m";
        //String query = "select group_concat(m.username) from Member m"; // function 키워드 대신 그냥 함수 명으로 사용할 수 있다.
        List<String> resultList = em.createQuery(query, String.class).getResultList();

        for (String s : resultList) {
            System.out.println("s = " + s);
        }
        ...
    }
}
```

- 결과

```
s = 관리자1,관리자2
```

<br><br><br>
## References 및 사진 출처

> __김영한의 자바 ORM 표준 JPA 프로그래밍 - 기본편__