---
layout: post
title:  "spring JPA JQPL 지연 로딩과 조회 성능 최적화"
subtitle:   "spring JPA JPQL 지연 로딩과 조회 성능 최적화"
date:   2021-09-13 02:31:27 +0900
categories: spring
tags: spring JPA ORM Mapping JPQL 지연 로딩과 조회 성능 최적화
comments: true
---


<br>

- 목차
	- [지연 로딩과 조회 성능 최적화](#지연-로딩과-조회-성능-최적화)
	- [V1 엔티티를 직접 노출](#v1-엔티티를-직접-노출)
	- [V2: 엔티티를 DTO로 변환](#v2-엔티티를-dto로-변환)
	- [V3: 엔티티를 DTO로 변환 - 페치 조인 최적화](#v3-엔티티를-dto로-변환---페치-조인-최적화)
	- [V4: JPA에서 DTO로 바로 조회](#v4-jpa에서-dto로-바로-조회)
	- [정리](#정리)
    
<br>

# 지연 로딩과 조회 성능 최적화

<br>

# V1 엔티티를 직접 노출

```java
/**
 * X to One(Many to One, One to One) 성능 최적화
 */
@RestController
@RequiredArgsConstructor
public class OrderSImpleApiController {

    private final OrderRepository orderRepository;

    @GetMapping("/api/v1/simple-orders")
    public List<Order> ordersV1() {
        List<Order> all = orderRepository.findAll();
        
        for (Order order : all) {
            order.getMember().getName();        //  LAZY 강제초기화(Member Proxy 초기화)
            order.getDelivery().getAddress();   //  LAZY 강제초기화(Delivery Proxy 초기화)
        }

        return all;
    }
}
```

- 문제점

1. orderRepository.findAll 함수 호출시 Member, orderItem, delivery를 가져오게 되는데 모두 LAZY LOADING으로 설정 되어 있기 때문에 위 함수 호출 시점에는 실제 데이터가 아닌 Proxy 객체가 들어가 있게 된다. 이 프록시 객체를 json으로 어떻게 생성해야 하는지 모르기 때문에 에러가 발생한다. <br>
에러내용 : [com.fasterxml.jackson.databind.exc.InvalidDefinitionException: No serializer found for class org.hibernate.proxy.pojo.bytebuddy.ByteBuddyInterceptor and no properties discovered to create BeanSerializer (to avoid exception, disable SerializationFeature.FAIL_ON_EMPTY_BEANS) (through reference chain: java.util.ArrayList[0]->jpabook.jpashop.domain.Order["member"]->jpabook.jpashop.domain.Member$HibernateProxy$Y7sAZ8N9["hibernateLazyInitializer"])]

2. 엔티티를 return 하고 있음. 이렇게 되면 엔티티의 값이 변경 되는 경우 API 스펙이 변경되는 치명적인 단점이 발생

3. 1번에 이유로 Proxy 객체를 강제 초기화 해야하는 문제가 발생생

4. 엔티티를 직접 노출 하는 경우네는 양방향 연관관계가 걸린 곳은 반드시 한곳을 @JsonIgnore 처리 해야한다. 안그러면 양쪽을 서로 호출하면서 무한 루프에 빠지게 된다.

5. LAZY 로딩으로 인한 데이터베이스 쿼리가 너무 많이 호출 되는 문제가 있다.(성능상 이슈가 발생)

<br>

> 참고: 앞에서 계속 강조했듯이 정말 간단한 애플리케이션이 아니면 엔티티를 API 응답으로 외부로 노출하는 것은 좋지 않다. 따라서 Hibernate5Module 를 사용하기 보다는 DTO로 변환해서 반환하는 것이 더 좋은 방법이다.

<br>

> 주의: 지연 로딩(LAZY)을 피하기 위해 즉시 로딩(EARGR)으로 설정하면 안된다! 즉시 로딩 때문에 연관관계가 필요 없는 경우에도 데이터를 항상 조회해서 성능 문제가 발생할 수 있다.
즉시 로딩으로 설정하면 성능 튜닝이 매우 어려워 진다. 항상 지연 로딩을 기본으로 하고, 성능 최적화가 필요한 경우에는 페치 조인(fetch join)을 사용해야한다(V3 에서 설명)

<br><br>

# V2: 엔티티를 DTO로 변환

```java
@GetMapping("/api/v2/simple-orders")
public Result ordersV2() {
    List<Order> orders = orderRepository.findAllByString(new OrderSearch());
    List<SimpleOrderDto> collect = orders.stream().map(m -> new SimpleOrderDto(m)).collect(Collectors.toList());

    return new Result(collect);
}

@Data
static class SimpleOrderDto {
    private Long orderId;
    private String name;
    private LocalDateTime orderDate;
    private OrderStatus orderStatus;
    private Address address;

    public SimpleOrderDto(Order order) {
        this.orderId = order.getId();
        this.name = order.getMember().getName();    //  LAZY 초기화
        this.orderDate = order.getOrderDate();
        this.orderStatus = order.getStatus();
        this.address = order.getDelivery().getAddress();    //  LAZY 초기화
    }
}

@Data
@AllArgsConstructor
static class Result<T> {
    private T data;
}
```

- 문제점
     
1. LAZY 로딩으로 인한 데이터베이스 쿼리가 너무 많이 호출 되는 문제가 있다.(성능상 이슈가 발생) -> ordersV1의 5번 문제와 같음 <br>
Order - Member ( N : 1 ) <br>
Order - Delivery ( 1 : 1 ) <br>
의 관계 이므로 Order를 가져올때 Member와 Delivery를 가져와야 한다. <br>
결국 1개의 Order 쿼리 후 Member, Delivery를 가져오는 쿼리가 추가 실행되어야 하므로 총 3번의 쿼리가 나가게 된다. <br>
만약 100개의 Order를 가져온다면? 최악의 경우 1 + 100 + 100 = 201번의 쿼리가 실행 될것이다. 
쿼리가 1 + N + N 번이 실행된 셈이다. <br>

> 지연로딩은 영속성 컨텍스트에서 조회하므로, 이미 조회된 경우 쿼리를 생략한다.

<br>

# V3: 엔티티를 DTO로 변환 - 페치 조인 최적화

```java

@RestController
@RequiredArgsConstructor
public class OrderSImpleApiController {

    private final OrderRepository orderRepository;

    ...
    @GetMapping("/api/v3/simple-orders")
    public Result ordersV3() {
        List<Order> orders = orderRepository.findAllWithMemberDelivery();
        List<SimpleOrderDto> collect = orders.stream().map(m -> new SimpleOrderDto(m)).collect(Collectors.toList());

        return new Result(collect);
    }
    ...
}

public class OrderRepository {

    ...

    public List<Order> findAllWithMemberDelivery() {
        String query = "select o from Order o join fetch o.member join fetch  o.delivery";

        return em.createQuery(query, Order.class).getResultList();
    }

    ...
}
```

- 결과

```sql
select
    order0_.order_id as order_id1_5_0_,
    member1_.member_id as member_i1_4_1_,
    delivery2_.delivery_id as delivery1_2_2_,
    order0_.delivery_id as delivery4_5_0_,
    order0_.member_id as member_i5_5_0_,
    order0_.order_date as order_da2_5_0_,
    order0_.status as status3_5_0_,
    member1_.city as city2_4_1_,
    member1_.street as street3_4_1_,
    member1_.zipcode as zipcode4_4_1_,
    member1_.name as name5_4_1_,
    delivery2_.city as city2_2_2_,
    delivery2_.street as street3_2_2_,
    delivery2_.zipcode as zipcode4_2_2_,
    delivery2_.status as status5_2_2_ 
from
    oders order0_ 
inner join
    member member1_ 
        on order0_.member_id=member1_.member_id 
inner join
    delivery delivery2_ 
        on order0_.delivery_id=delivery2_.delivery_id
```

fetch join으로 1번의 쿼리로 데이터를 가져올수 있게 되었다. 하지만 ordersV3 에서는 데이터를 가져온후 DTO로 변환화는 과정이 있지만 JPA에서 바로 DTO로 가져올수 있도록 할수 있다. V4에서 확인해보자

# V4: JPA에서 DTO로 바로 조회

```java
@RestController
@RequiredArgsConstructor
public class OrderSImpleApiController {

    private final OrderRepository orderRepository;

    ...
    @GetMapping("/api/v4/simple-orders")
    public Result ordersV4() {
        List<SimpleOrderQueryDto> collect = orderSimpleQueryRepository.findOrderDtos();
        return new Result(collect);
    }
    ...
}


@Repository
@RequiredArgsConstructor
@Transactional(readOnly = true)
public class OrderSimpleQueryRepository {

    ...
    public List<SimpleOrderQueryDto> findOrderDtos() {
        return em.createQuery("select new jpabook.jpashop.repository.order.simplequery.SimpleOrderQueryDto(o.id, m.name, o.orderDate, o.status, d.address) from Order o join o.member m join o.delivery d", SimpleOrderQueryDto.class).getResultList();
    }
}

@Data
public class SimpleOrderQueryDto {

    private Long orderId;
    private String name;
    private LocalDateTime orderDate;
    private OrderStatus orderStatus;
    private Address address;

    public SimpleOrderQueryDto(Long orderId, String name, LocalDateTime orderDate, OrderStatus orderStatus, Address address) {
        this.orderId = orderId;
        this.name = name;
        this.orderDate = orderDate;
        this.orderStatus = orderStatus;
        this.address = address;
    }
}
```

- 결과

```sql
select
    order0_.order_id as col_0_0_,
    member1_.name as col_1_0_,
    order0_.order_date as col_2_0_,
    order0_.status as col_3_0_,
    delivery2_.city as col_4_0_,
    delivery2_.street as col_4_1_,
    delivery2_.zipcode as col_4_2_ 
from
    oders order0_ 
inner join
    member member1_ 
        on order0_.member_id=member1_.member_id 
inner join
    delivery delivery2_ 
        on order0_.delivery_id=delivery2_.delivery_id
```

위와 같이 DTO로 바로 받는 경우에는 리포지토리 재사용성 떨어지고 API 스펙에 맞춘 코드가 리포지토리에 들어가는 단점이 있다. 하지만 결과와 같이 원하는 값만 선택해서 조회가 가능하다. new 명령어를 통해서 가져 올수 있으며 DTO의 전체 경로를 써야한다(jpabook.jpashop.repository.order.simplequery.SimpleOrderQueryDto). 그리고 인자로는 SimpleOrderQueryDto 생성자에 정의한 대로 파라미터를 넘겨야 한다. 이러한 메소드는 일반 Repository에 넣기에는 다소 무리가 있다. 그렇기 때문에 Dto Qurey용 Repository를 새로 만드는 것이 바람직해 보인다.

<br>

# 정리

엔티티를 DTO로 변환하거나, DTO로 바로 조회하는 두가지 방법은 각각 장단점이 있다. 둘중 상황에 따라서 더 나은 방법을 선택하면 된다. 엔티티로 조회하면 리포지토리 사용성도 좋고, 개발도 단순해진다. 따라서 권장하는 방법은 다음과 같다. <br>
쿼리 방식 선택 권장 순서
1. 우선 엔티티를 DTO로 변환하는 방법을 선택한다.
2. 필요하면 페치 조인으로 성능을 최적화 한다. 대부분의 성능 이슈가 해결된다.
3. 그래도 안되면 DTO로 직접 조회하는 방법을 사용한다.
4. 최후의 방법은 JPA가 제공하는 네이티브 SQL이나 스프링 JDBC Template을 사용해서 SQL을 직접
사용한다

<br><br><br>
## References 및 사진 출처

> __김영한의 실전! 스프링 부트와 JPA 활용2 - API 개발과 성능 최적화__