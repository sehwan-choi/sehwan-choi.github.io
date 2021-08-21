---
layout: post
title:  "spring JPA 값 타입의 비교"
subtitle:   "spring JPA ValueType equals"
date:   2021-08-19 18:50:27 +0900
categories: spring
tags: spring JPA ORM Mapping ValueType Equals
comments: true
---


<br>

- 목차
	- [값 타입의 비교](#값-타입의-비교)
    
<br>

# 값 타입의 비교

- 값 타입: 인스턴스가 달라도 그 안에 값이 같으면 같은 것으로 봐야 한다.

![그림1](https://sehwan-choi.github.io/assets/img/spring/JPA-MAPPING4/jpa26.jpg)

<br>

- 동일성(identity) 비교: 인스턴스의 참조 값을 비교, == 사용
- 동등성(equivalence) 비교: 인스턴스의 값을 비교, equals() 사용
- 값 타입은 a.equals(b)를 사용해서 동등성 비교를 해야 함
- 값 타입의 equals() 메소드를 적절하게 재정의(주로 모든 필드사용)

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

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        Address address = (Address) o;
        return Objects.equals(city, address.city) && Objects.equals(street, address.street) && Objects.equals(zipcode, address.zipcode);
    }
    ...
}

public class testMain {

    public static void main(String[] args) {

        Address address1 = new Address("CITY", "STREET", "ZIPCODE");
        Address address2 = new Address("CITY", "STREET", "ZIPCODE");

        System.out.println("address equls address2 = " + address1.equals(address2));
    }
}
```

위 코드와 같이 Address 클래스에 equals를 override하여 클래스 내부의 변수값을 비교한다.

<br><br><br>
## References 및 사진 출처

> __김영한의 자바 ORM 표준 JPA 프로그래밍 - 기본편__