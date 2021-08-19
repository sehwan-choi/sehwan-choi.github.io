---
layout: post
title:  "spring JPA 값타입 - 기본값 타입"
subtitle:   "spring JPA ValueType"
date:   2021-08-19 15:44:27 +0900
categories: spring
tags: spring JPA ORM Mapping ValueType
comments: true
---


<br>

- 목차
	- [JPA의 데이터 타입 분류](#jpa의-데이터-타입-분류)
	- [값 타입 분류](#값-타입-분류)
	- [기본값 타입](#기본값-타입)
	- [참고: 자바의 기본 타입은 절대 공유X](#참고-자바의-기본-타입은-절대-공유x)
    
<br>

# JPA의 데이터 타입 분류

- 엔티티 타입
    - @Entity로 정의하는 객체
    - 데이터가 변해도 식별자로 지속해서 추적 가능
    - 예) 회원 엔티티의 키나 나이 값을 변경해도 식별자로 인식 가능
- 값 타입
    - int, Integer, String처럼 단순히 값으로 사용하는 자바 기본 타입이나 객체
    - 식별자가 없고 값만 있으므로 변경시 추적 불가
    - 예) 숫자 100을 200으로 변경하면 완전히 다른 값으로 대체

<br>

# 값 타입 분류

- 기본값 타입
    - 자바 기본 타입(int, double) 
    - 래퍼 클래스(Integer, Long) 
    - String 
- 임베디드 타입(embedded type, 복합 값 타입) 
- 컬렉션 값 타입(collection value type)

<br>

# 기본값 타입

- 예): String name, int age 
- 생명주기를 엔티티의 의존
    - 예) 회원을 삭제하면 이름, 나이 필드도 함께 삭제
- 값 타입은 공유하면 안된다
    - 예) 회원 이름 변경시 다른 회원의 이름도 함께 변경되면 안됨

<br>

# 참고: 자바의 기본 타입은 절대 공유X

- int, double 같은 기본 타입(primitive type)은 절대 공유X 
- 기본 타입은 항상 값을 복사함

```java
        int a = 10;
        int b = a;

        a = 20;

        System.out.println("a = " + a);
        System.out.println("b = " + b);
```

위 코드와 같이 b에는 a의 레퍼런스가 저장되는 것이 아닌 값이 복사되어 저장되기 때문에 a가 변경이 되어도 b는 영향을 받지 않는다.


<br><br><br>
## References 및 사진 출처

> __김영한의 자바 ORM 표준 JPA 프로그래밍 - 기본편__