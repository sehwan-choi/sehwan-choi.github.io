---
layout: post
title:  "spring JPA 값타입과 불변 객체"
subtitle:   "spring JPA ValueType"
date:   2021-08-19 18:00:27 +0900
categories: spring
tags: spring JPA ORM Mapping ValueType 
comments: true
---


<br>

- 목차
	- [값 타입과 불변 객체](#값-타입과-불변-객체)
	- [값 타입 공유 참조](#값-타입-공유-참조)
	- [값 타입 복사](#값-타입-복사)
	- [객체 타입의 한계](#객체-타입의-한계)
	- [불변 객체](#불변-객체)
	- [결론](#결론)
    
<br>

# 값 타입과 불변 객체

값 타입은 복잡한 객체 세상을 조금이라도 단순화하려고 만든 개념이다. 따라서 값 타입은 단순하고 안전하게 다룰 수 있어야 한다.

<br>

# 값 타입 공유 참조

- 임베디드 타입 같은 값 타입을 여러 엔티티에서 공유하면 위험함
- 부작용(side effect) 발생

![그림1](https://sehwan-choi.github.io/assets/img/spring/JPA-MAPPING4/jpa21.jpg)

위 사진과 같이 회원1과 회원2가 주소라는 객체를 공유 참조 하고 있을 때 회원1의 city컬럼의 값인 OldCity를 NewCity로 변경하면 어떻게 될까?

```java
public class jpaMain {

    public static void main(String[] args) {
        ...
        try {
            Address address = new Address("CITY", "STREET", "ZIPCODE");

            Member member1 = new Member();
            member1.setUsername("NAME");
            member1.setHomeAddress(address);
            em.persist(member1);

            Member member2 = new Member();
            member2.setUsername("NAME2");
            member2.setHomeAddress(address);
            em.persist(member2);

            member1.getHomeAddress().setCity("newCity");    // Member1의 city값 변경

            em.flush();
            em.clear();
        }
        ...
    }
}
```

- 실행결과

![그림1](https://sehwan-choi.github.io/assets/img/spring/JPA-MAPPING4/jpa22.jpg)

위 사진과 같이 Member1의 city값을 newCity로 변경하였는데, Member2까지 city값이 newCity로 변경되었다. 왜냐하면 member1, member2가 setHomeAddress 할 때 똑같은 Address객체를 공유 참조하였기 때문에 __부작용(side effect)__ 이 발생한 것이다.

<br>

## 값 타입 복사

- 값 타입의 실제 인스턴스인 값을 공유하는 것은 위험
- 대신 값(인스턴스)를 복사해서 사용

![그림1](https://sehwan-choi.github.io/assets/img/spring/JPA-MAPPING4/jpa23.jpg)

```java
public class jpaMain {

    public static void main(String[] args) {
        ...
        try {
            Address address = new Address("CITY", "STREET", "ZIPCODE");

            Member member1 = new Member();
            member1.setUsername("NAME");
            member1.setHomeAddress(address);
            em.persist(member1);

            Address newAddress = new Address(address.getCity(), address.getStreet(), address.getZipcode()); // 기존 address 객체에서 newAddress로 값을 복사

            Member member2 = new Member();
            member2.setUsername("NAME2");
            member2.setHomeAddress(newAddress); // 새로운 newAddress 객체를 set한다.
            em.persist(member2);

            member1.getHomeAddress().setCity("newCity");

            em.flush();
            em.clear();
        }
    }
}
```

- 실행결과

![그림1](https://sehwan-choi.github.io/assets/img/spring/JPA-MAPPING4/jpa24.jpg)

위 사진과 같이 기존 Address 객체에서 newAddress로 값을 복사하여 set하게 되면, address 객체의 공유참조가 일어나지 않게 되므로 member1의 city를 newCity로 수정해도 다른 member들은 영향을 받지 않게 된다.

<br>

## 객체 타입의 한계

- 항상 값을 복사해서 사용하면 공유 참조로 인해 발생하는 부작용을 피할 수 있다. 
- 문제는 임베디드 타입처럼 직접 정의한 값 타입은 자바의 기본타입이 아니라 객체 타입이다. 
- 자바 기본 타입에 값을 대입하면 값을 복사한다. 
- 객체 타입은 참조 값을 직접 대입하는 것을 막을 방법이 없다. 
- 객체의 공유 참조는 피할 수 없다.

![그림1](https://sehwan-choi.github.io/assets/img/spring/JPA-MAPPING4/jpa25.jpg)

<br>

## 불변 객체

- 객체 타입을 수정할 수 없게 만들면 부작용을 원천 차단
- 값 타입은 불변 객체(immutable object)로 설계해야함
- 불변 객체: 생성 시점 이후 절대 값을 변경할 수 없는 객체
- 생성자로만 값을 설정하고 수정자(Setter)를 만들지 않으면 됨
- 참고: Integer, String은 자바가 제공하는 대표적인 불변 객체

```java
@Embeddable
public class Address {

    private String city;
    private String street;
    private String zipcode;

    public Address() {
    }

    public Address(String city, String street, String zipcode) {
        this.city = city;
        this.street = street;
        this.zipcode = zipcode;
    }

    public String getCity() {
        return city;
    }

    public String getStreet() {
        return street;
    }

    public String getZipcode() {
        return zipcode;
    }
}
```

위 코드와 같이 setter를 없애고 생성자로만 값을 세팅할수 있게 변경한다. 이렇게 한다면 객체가 공유 참조 되어도 값을 변경하는 것이 불가능 하기 때문에 공유 참조된 값들이 모두 변경되는 문제는 없을 것이다. 만약에 수정이 필요하다면 Address를 새로 생성하여 set하는것이 바람직하다.

<br>

# 결론

> __불변이라는 작은 제약으로 부작용이라는 큰 재앙을 막을수 있다.__

<br><br><br>
## References 및 사진 출처

> __김영한의 자바 ORM 표준 JPA 프로그래밍 - 기본편__