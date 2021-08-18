---
layout: post
title:  "spring JPA 즉시로딩과 지연로딩"
subtitle:   "spring JPA LAZY EAGER"
date:   2021-08-19 01:10:27 +0900
categories: spring
tags: spring JPA ORM Mapping LAZY EAGER
comments: true
---


<br>

- 목차
	- [지연로딩](#지연로딩)
	- [즉시로딩](#즉시로딩)
	- [프록시와 즉시로딩 주의](#프록시와-즉시로딩-주의)
	- [지연로딩의 활용](#지연로딩의-활용)
	- [결론](#결론)
<br>

# 지연로딩

JPA 사용시 지연로딩으로 설정되어 있으면 연관관계가 있는 Entity를 직접 사용하기 전까지는 해당 데이터를 가져오지않고 프록시객체로 대체한다.

```java
@Entity
public class Member extends BaseEntity{

    @Id
    @GeneratedValue
    @Column(name = "MEMBER_ID")
    private Long id;
    
    @ManyToOne(fetch = FetchType.LAZY)  //  지연로딩 설정
    @JoinColumn(name = "ITEM_ID")
    private OrderItem item;

    ...
}

@Entity
public class OrderItem {

    @Id
    @GeneratedValue
    @Column(name = "ORDER_ITEM_ID")
    private Long id;

    private int orderPrice;

    private int count;

    ...
}


public static void main(String[] args) {

    ...

    try {

        OrderItem orderItem = new OrderItem();
        orderItem.setOrderPrice(10000);
        orderItem.setCount(10);
        em.persist(orderItem);

        Member member = new Member();
        member.setName("Name");
        member.setItem(orderItem);

        em.persist(member);

        em.flush();
        em.clear();

        Member member1 = em.find(Member.class, member.getId());
        System.out.println("member class = " + member1.getClass());
        System.out.println("member.getTeam class = " + member1.getItem().getClass());

        System.out.println("============");
        System.out.println("member item count = " + member1.getItem().getCount());
        System.out.println("============");
    }
    ...
}
```

실행 결과

```java

member class = class jpabook.jpashop.domain.Member
member.getTeam class = class jpabook.jpashop.domain.OrderItem$HibernateProxy$bLxZIBuN
============
select Query문(생략)
member item count = 10
============
```

위 와같이 member는 DB에서 조회 해왔기 때문에 실제 Member Class로 나오지만 내부의 Item은 지연로딩이기 때문에 실제 값을 사용하기 전까지는 Proxy 객체가 들어있게 된다. <br>
이후에 member1.getItem().getCount() 한 시점에 DB에서 실제 데이터를 쿼리해 온다.

<br>

![그림1](https://sehwan-choi.github.io/assets/img/spring/JPA-MAPPING4/jpa5.jpg)

![그림1](https://sehwan-choi.github.io/assets/img/spring/JPA-MAPPING4/jpa6.jpg)

위 그림과 같이 지연 로딩을 사용하게 되면 프록시로 조회해오게 된다. 이후 실제 값을 사용할때 프록시의 초기화가 일어나게 되며 DB에서 값을 가져오게된다.

<br><br>

# 즉시로딩

JPA 사용시 즉시로딩으로 설정되어 있으면 연관관계가 있는 Entity를 모두 가져온다.

```java
@Entity
public class Member extends BaseEntity{

    @Id
    @GeneratedValue
    @Column(name = "MEMBER_ID")
    private Long id;
    
    @ManyToOne(fetch = FetchType.EAGER)  //  즉시로딩 설정
    @JoinColumn(name = "ITEM_ID")
    private OrderItem item;

    ...
}

@Entity
public class OrderItem {

    @Id
    @GeneratedValue
    @Column(name = "ORDER_ITEM_ID")
    private Long id;

    private int orderPrice;

    private int count;

    ...
}


public static void main(String[] args) {

    ...

    try {

        OrderItem orderItem = new OrderItem();
        orderItem.setOrderPrice(10000);
        orderItem.setCount(10);
        em.persist(orderItem);

        Member member = new Member();
        member.setName("Name");
        member.setItem(orderItem);

        em.persist(member);

        em.flush();
        em.clear();

        Member member1 = em.find(Member.class, member.getId());
        System.out.println("member class = " + member1.getClass());
        System.out.println("member.getTeam class = " + member1.getItem().getClass());

        System.out.println("============");
        System.out.println("member item count = " + member1.getItem().getCount());
        System.out.println("============");
    }
    ...
}
```

실행 결과

```java

member class = class jpabook.jpashop.domain.Member
member.getTeam class = class jpabook.jpashop.domain.OrderItem
============
member item count = 10
============
```

위 와같이 즉시로딩으로 설정되어 있다면 Member를 조회하면서 연관관계가 있는 Item을 같이 DB에서 가져오기 때문에 프록시객체가아닌 실제 객체가 나오게 된다.

<br>

![그림1](https://sehwan-choi.github.io/assets/img/spring/JPA-MAPPING4/jpa7.jpg)

![그림1](https://sehwan-choi.github.io/assets/img/spring/JPA-MAPPING4/jpa8.jpg)

위 그림과 같이 연관관계가 있는 Entity를 모두 DB에서 가져오게 된다.

<br>

# 프록시와 즉시로딩 주의

- __가급적 지연 로딩만 사용(특히 실무에서)__
- __즉시 로딩을 적용하면 예상하지 못한 SQL이 발생__
- __즉시 로딩은 JPQL에서 N+1 문제를 일으킨다.__
- @ManyToOne, @OneToOne은 기본이 즉시 로딩 -> LAZY로 설정
- @OneToMany, @ManyToMany는 기본이 지연 로딩

<br>

# 지연로딩의 활용

- Member와 Team은 자주 함께 사용 -> 즉시 로딩
- Member와 Order는 가끔 사용 -> 지연 로딩
- Order와 Product는 자주 함께 사용 -> 즉시 로딩

![그림1](https://sehwan-choi.github.io/assets/img/spring/JPA-MAPPING4/jpa9.jpg)

![그림1](https://sehwan-choi.github.io/assets/img/spring/JPA-MAPPING4/jpa10.jpg)

![그림1](https://sehwan-choi.github.io/assets/img/spring/JPA-MAPPING4/jpa11.jpg)

<br>

# 결론

- 모든 연관관계에 지연 로딩을 사용해라!
- 실무에서 즉시 로딩을 사용하지 마라!
- JPQL fetch 조인이나, 엔티티 그래프 기능을 사용해라!
- 즉시 로딩은 상상하지 못한 쿼리가 나간다.

<br><br>

<br><br>

<br><br><br>
## References 및 사진 출처

> __김영한의 자바 ORM 표준 JPA 프로그래밍 - 기본편__