---
layout: post
title:  "spring JPA 컬렉션 조회 최적화"
subtitle:   "spring JPA 컬렉션 조회 최적화"
date:   2021-09-15 04:31:27 +0900
categories: spring
tags: spring JPA ORM Mapping 컬렉션 조회 최적화
comments: true
---


<br>

- 목차
	- [V1: 엔티티 직접 노출](#v1-엔티티-직접-노출)
	- [V2: 엔티티를 DTO로 변환](#v2-엔티티를-dto로-변환)
	- [V3: 엔티티를 DTO로 변환 - 페치 조인 최적화](#v3-엔티티를-dto로-변환---페치-조인-최적화)
	- [V3.1: 엔티티를 DTO로 변환 - 페이징과 한계 돌파](#v31-엔티티를-dto로-변환---페이징과-한계-돌파)
	    - [한계 돌파](#한계-돌파)
	- [V4: JPA에서 DTO 직접 조회](#v4-jpa에서-dto-직접-조회)
	- [V5: JPA에서 DTO 직접 조회 - 컬렉션 조회 최적화](#v5-jpa에서-dto-직접-조회---컬렉션-조회-최적화)
	- [V6: JPA에서 DTO로 직접 조회, 플랫 데이터 최적화](#v6-jpa에서-dto로-직접-조회-플랫-데이터-최적화)
	- [정리](#정리)
    	- [엔티티 조회](#엔티티-조회)
    	- [최적화 권장 순서](#최적화-권장-순서)
    	- [DTO 조회 방식의 선택지](#dto-조회-방식의-선택지)
    
<br>

# 컬렉션 조회 최적화

<br>

- 도메인 모델

![그림1](https://sehwan-choi.github.io/assets/img/spring/JPA2/jpa1.jpg)

위 사진은 아래에서 사용될 도메인 모델이다.

<br>

# V1: 엔티티 직접 노출

```java
/**
 * X to Many( One to Many, Many to Many) 성능 최적화
 *
 */
@RestController
@RequiredArgsConstructor
public class OrderApiController {

    private final OrderRepository orderRepository;

    @GetMapping("/api/v1/orders")
    public List<Order> ordersV1() {
        List<Order> all = orderRepository.findAll();
        for (Order order : all) {
            order.getMember().getName();        //  LAZY 강제초기화(Member Proxy 초기화)
            order.getDelivery().getAddress();   //  LAZY 강제초기화(Delivery Proxy 초기화)
            List<OrderItem> orderItems = order.getOrderItems();         //  LAZY 강제초기화(OrderItem Proxy 초기화)
            orderItems.stream().forEach(o -> o.getItem().getName());    //  LAZY 강제초기화(Item Proxy 초기화)
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
즉시 로딩으로 설정하면 성능 튜닝이 매우 어려워 진다. 항상 지연 로딩을 기본으로 하고, 성능 최적화가 필요한 경우에는 페치 조인(fetch join)을 사용해야한다.

<br>

# V2: 엔티티를 DTO로 변환

```java
@GetMapping("/api/v2/orders")
public Result ordersV2() {
    List<Order> all = orderRepository.findAll();
    List<OrderDto> collect = all.stream().map(o -> new OrderDto(o)).collect(Collectors.toList());

    return new Result(collect);
}

@Getter
static class OrderDto {

    private Long orderId;
    private String name;
    private LocalDateTime orderDate;
    private OrderStatus orderStatus;
    private Address address;
    private List<OrderItemDto> orderItems;

    public OrderDto(Order o) {
        orderId = o.getId();
        name = o.getMember().getName();
        orderDate = o.getOrderDate();
        orderStatus = o.getStatus();
        address = o.getDelivery().getAddress();;
        orderItems = o.getOrderItems().stream().map(m -> new OrderItemDto(m)).collect(Collectors.toList());
    }
}

@Getter
static class OrderItemDto {

    private String itemName;
    private int orderPrice;
    private int count;

    public OrderItemDto(OrderItem m) {
        itemName = m.getItem().getName();
        orderPrice = m.getOrderPrice();
        count = m.getCount();
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
Order - OrderItem ( 1 : N ) <br>
의 관계 이므로 Order를 가져올때 Member와 Delivery, OrderItem를 가져와야 한다. <br>
결국 1개의 Order 쿼리 후 Member, Delivery, OrderItem을 가져오는 쿼리가 추가 실행되어야 하므로 1 + N + N + N번의 쿼리가 나가게 된다. <br>

> 지연로딩은 영속성 컨텍스트에서 조회하므로, 이미 조회된 경우 쿼리를 생략한다.

<br>

# V3: 엔티티를 DTO로 변환 - 페치 조인 최적화

```java

@RestController
@RequiredArgsConstructor
public class OrderApiController {

    private final OrderRepository orderRepository;

    ...

    @GetMapping("/api/v3/orders")
    public Result ordersV3() {
        List<Order> all = orderRepository.findAllWithItem();
        List<OrderDto> collect = all.stream().map(o -> new OrderDto(o)).collect(Collectors.toList());

        return new Result(collect);
    }
    ...
}

@Repository
@RequiredArgsConstructor
@Transactional(readOnly = true)
public class OrderRepository {

    ...
    public List<Order> findAllWithItem() {

    return em.createQuery(
            "select distinct o from Order o " +
                    "join fetch o.member m " +
                    "join fetch o.delivery d " +
                    "join fetch o.orderItems oi " +
                    "join fetch oi.item i", Order.class)
            .getResultList();
    }
    ...
}
```

페치 조인으로 SQL이 1번만 실행된다. <br>
findAllWithItem에서 distinct 를 사용한 이유는 1대다 조인이 있으므로 데이터베이스 row가 증가한다. 그 결과 같은 order 엔티티의 조회 수도 증가하게 된다. JPA의 distinct는 SQL에 distinct를 추가하고, 더해서 같은 엔티티가 조회되면, 애플리케이션에서 중복을 걸러준다. 이 예에서 order가 컬렉션 페치 조인 때문에 중복 조회 되는 것을 막아준다. <br>
하지만, 페이징이 불가능 하다는 단점이 있다. 페이징을 하게되면

```
firstResult/maxResults specified with collection fetch; applying in memory!
```

위 경고가 나오게 된다.

> 참고: 컬렉션 페치 조인을 사용하면 페이징이 불가능하다. 하이버네이트는 경고 로그를 남기면서 모든
데이터를 DB에서 읽어오고, 메모리에서 페이징 해버린다. ___(매우 위험하다)___

> 참고: 컬렉션 페치 조인은 1개만 사용할 수 있다. 컬렉션 둘 이상에 페치 조인을 사용하면 안된다. 데이터가 부정합하게 조회될 수 있다.

<br>

# V3.1: 엔티티를 DTO로 변환 - 페이징과 한계 돌파

- 컬렉션을 페치 조인하면 페이징이 불가능하다.
    - 컬렉션을 페치 조인하면 일대다 조인이 발생하므로 데이터가 예측할 수 없이 증가한다.
    - 일다대에서 일(1)을 기준으로 페이징을 하는 것이 목적이다. 그런데 데이터는 다(N)를 기준으로 row가 생성된다.
    - Order를 기준으로 페이징 하고 싶은데, 다(N)인 OrderItem을 조인하면 OrderItem이 기준이 되어버린다.
- 이 경우 하이버네이트는 경고 로그를 남기고 모든 DB 데이터를 읽어서 메모리에서 페이징을 시도한다. 최악의 경우 장애로 이어질 수 있다.

<br>

## 한계 돌파

그러면 페이징 + 컬렉션 엔티티를 함께 조회하려면 어떻게 해야할까? <br>
코드도 단순하고, 성능 최적화도 보장하는 매우 강력한 방법이 있다. <br>
대부분의 페이징 + 컬렉션 엔티티 조회 문제는 이 방법으로 해결할 수 있다. <br>

```java
@RestController
@RequiredArgsConstructor
public class OrderApiController {

    private final OrderRepository orderRepository;

    ...
    @GetMapping("/api/v3.1/orders")
    public Result ordersV3_1(
            @RequestParam(value = "offset", defaultValue = "0") int offset,
            @RequestParam(value = "limit", defaultValue = "100") int limit) {
        List<Order> all = orderRepository.findAllWithMemberDelivery(offset,limit);
        List<OrderDto> collect = all.stream().map(o -> new OrderDto(o)).collect(Collectors.toList());

        return new Result(collect);
    }
    ...
}

@Repository
@RequiredArgsConstructor
@Transactional(readOnly = true)
public class OrderRepository {

    ...
    public List<Order> findAllWithMemberDelivery(int offset, int limit) {
    String query = "select o from Order o join fetch o.member join fetch  o.delivery";

    return em.createQuery(query, Order.class)
            .setFirstResult(offset)
            .setMaxResults(limit)
            .getResultList();
    }
    ...
}
```

- application.yml

```yml
  jpa:
    hibernate:
      ddl-auto: create
    properties:
      hibernate:
        show_sql: true
        format_sql: true
        default_batch_fetch_size: 100   <-- 추가
```

1. 먼저 ToOne(OneToOne, ManyToOne) 관계를 모두 페치조인 한다. ToOne 관계는 row수를 증가시키지 않으므로 페이징 쿼리에 영향을 주지 않는다.(Member, Delivery는 X To One 관계 이므로 페치조인을 한다.)
2. 컬렉션은 지연 로딩으로 조회한다.
3. 지연 로딩 성능 최적화를 위해 hibernate.default_batch_fetch_size , @BatchSize 를 적용한다.
    - hibernate.default_batch_fetch_size: 글로벌 설정
    - @BatchSize: 개별 최적화
    - 이 옵션을 사용하면 컬렉션이나, 프록시 객체를 한꺼번에 설정한 size 만큼 IN 쿼리로 조회한다.

- IN Query

```sql
select
    orderitems0_.order_id as order_id5_6_1_,
    orderitems0_.order_item_id as order_it1_6_1_,
    orderitems0_.order_item_id as order_it1_6_0_,
    orderitems0_.count as count2_6_0_,
    orderitems0_.item_id as item_id4_6_0_,
    orderitems0_.order_id as order_id5_6_0_,
    orderitems0_.order_price as order_pr3_6_0_ 
from
    order_item orderitems0_ 
where
    orderitems0_.order_id in (  <-- IN Query가 실행된다.
        ?, ?
    )
```

hibernate.default_batch_fetch_size 를 설정하지 않은경우에는 단건으로 N번의 쿼리가 실행되지만 설정하게 된다면 IN 으로 사이즈 많큼 가져오게 되므로 쿼리가 엄청나게 줄어든다. <br>
실행 결과로 쿼리 호출 수가 1 + N 에서 1 + 1 로 최적화 된다. <br>
조인보다 DB 데이터 전송량이 최적화 된다. (Order와 OrderItem을 조인하면 Order가
OrderItem 만큼 중복해서 조회된다. 이 방법은 각각 조회하므로 전송해야할 중복 데이터가 없다.) <br>
페치 조인 방식과 비교해서 쿼리 호출 수가 약간 증가하지만, DB 데이터 전송량이 감소한다. <br>
컬렉션 페치 조인은 페이징이 불가능 하지만 이 방법은 페이징이 가능하다. <br>

<br>

- 결론

ToOne 관계는 페치 조인해도 페이징에 영향을 주지 않는다. 따라서 ToOne 관계는 페치조인으로 쿼리 수를 줄이고 해결하고, 나머지는 hibernate.default_batch_fetch_size 로 최적화 하자.

<br>

>  참고: default_batch_fetch_size 의 크기는 적당한 사이즈를 골라야 하는데, 100~1000 사이를 선택하는 것을 권장한다. 이 전략을 SQL IN 절을 사용하는데, 데이터베이스에 따라 IN 절 파라미터를 1000으로 제한하기도 한다. 1000으로 잡으면 한번에 1000개를 DB에서 애플리케이션에 불러오므로 DB 에 순간 부하가 증가할 수 있다. 하지만 애플리케이션은 100이든 1000이든 결국 전체 데이터를 로딩해야 하므로 메모리 사용량이 같다. 1000으로 설정하는 것이 성능상 가장 좋지만, 결국 DB든 애플리케이션이든 순간 부하를 어디까지 견딜 수 있는지로 결정하면 된다.

<br>

# V4: JPA에서 DTO 직접 조회

```java
@RestController
@RequiredArgsConstructor
public class OrderApiController {

    private final OrderRepository orderRepository;

    ...
    @GetMapping("/api/v4/orders")
    public Result ordersV4() {
        List<OrderQueryDto> orderQueryDtos = orderQueryRepository.findOrderQueryDtos();
        return new Result(orderQueryDtos);
    }
    ...
}


@Repository
@RequiredArgsConstructor
public class OrderQueryRepository {

    private final EntityManager em;

    public List<OrderQueryDto> findOrderQueryDtos() {
        List<OrderQueryDto> result = findOrders();
        result.forEach(o -> {
            List<OrderItemQueryDto> orderItems = findOrderItems(o.getOrderId());
            o.setOrderItems(orderItems);
        });

        return result;
    }

    private List<OrderItemQueryDto> findOrderItems(Long orderId) {
        return em.createQuery("select new jpabook.jpashop.repository.order.query.OrderItemQueryDto(oi.order.id, i.name, oi.orderPrice, oi.count) " +
                "from OrderItem oi " +
                "join oi.item i " +
                "where oi.order.id = :orderId", OrderItemQueryDto.class)
                .setParameter("orderId",orderId)
                .getResultList();
    }

    private List<OrderQueryDto> findOrders() {
        return em.createQuery("select new jpabook.jpashop.repository.order.query.OrderQueryDto(o.id, m.name, o.orderDate, o.status, d.address) from Order o " +
                "join o.member m " +
                "join o.delivery d", OrderQueryDto.class)
                .getResultList();
    }
}

@Data
public class OrderItemQueryDto {

    private Long orderId;
    private String itemName;
    private int orderPrice;
    private int count;

    public OrderItemQueryDto(Long orderId, String itemName, int orderPrice, int count) {
        this.orderId = orderId;
        this.itemName = itemName;
        this.orderPrice = orderPrice;
        this.count = count;
    }
}

@Data
public class OrderQueryDto {

    private Long orderId;
    private String name;
    private LocalDateTime orderDate;
    private OrderStatus orderStatus;
    private Address address;
    private List<OrderItemQueryDto> orderItems;

    public OrderQueryDto(Long orderId, String name, LocalDateTime orderDate, OrderStatus orderStatus, Address address) {
        this.orderId = orderId;
        this.name = name;
        this.orderDate = orderDate;
        this.orderStatus = orderStatus;
        this.address = address;
    }
}
```

위와같이 DTO로 바로 가져오는 경우에는 ToOne(N:1, 1:1) 관계들을 먼저 조회하고, ToMany(1:N) 관계는 각각 별도로 처리한다. 이런 방식을 선택한 이유는 
ToOne 관계는 조인해도 데이터 row 수가 증가하지 않고,
ToMany(1:N) 관계는 조인하면 row 수가 증가한다.
row 수가 증가하지 않는 ToOne 관계는 조인으로 최적화 하기 쉬우므로 한번에 조회하고, ToMany 관계는 최적화 하기 어려우므로 findOrderItems() 같은 별도의 메서드로 조회한다. <br>
하지만 1 + N 문제가 발생하기 때문에 성능이 좋지않다. 왜냐하면 findOrders()에서 OrderQueryDto만들때 쿼리 1번, 

```
result.forEach(o -> {
            List<OrderItemQueryDto> orderItems = findOrderItems(o.getOrderId());
            o.setOrderItems(orderItems);
        });
```

위의 부분에서 가져온 쿼리 row만큼 쿼리를 해야하기 때문에 N번의 쿼리가 나가게 되기 때문이다.

<br>

# V5: JPA에서 DTO 직접 조회 - 컬렉션 조회 최적화

```java
@RestController
@RequiredArgsConstructor
public class OrderApiController {

    private final OrderRepository orderRepository;

    ...
    @GetMapping("/api/v5/orders")
    public Result ordersV5() {
        List<OrderQueryDto> orderQueryDtos = orderQueryRepository.findAllByDto_opmization();
        return new Result(orderQueryDtos);
    }
    ...
}

@Repository
@RequiredArgsConstructor
public class OrderQueryRepository {

    private final EntityManager em;
    ...

    public List<OrderQueryDto> findAllByDto_opmization() {
        List<OrderQueryDto> result = findOrders();

        List<Long> orderIds = toOrderIts(result);
        Map<Long, List<OrderItemQueryDto>> orderItemMap = findOrderItemMap(orderIds);

        result.forEach(o -> o.setOrderItems(orderItemMap.get(o.getOrderId())));
        return result;
    }

    private Map<Long, List<OrderItemQueryDto>> findOrderItemMap(List<Long> orderIds) {
        List<OrderItemQueryDto> orderItems = em.createQuery("select new jpabook.jpashop.repository.order.query.OrderItemQueryDto(oi.order.id, i.name, oi.orderPrice, oi.count) " +
                        "from OrderItem oi " +
                        "join oi.item i " +
                        "where oi.order.id in :orderIds", OrderItemQueryDto.class)
                .setParameter("orderIds", orderIds)
                .getResultList();

        Map<Long, List<OrderItemQueryDto>> orderItemMap = orderItems.stream().collect(Collectors.groupingBy(o -> o.getOrderId()));
        return orderItemMap;
    }

    private List<Long> toOrderIts(List<OrderQueryDto> result) {
        List<Long> orderIds = result.stream().map(o -> o.getOrderId())
                .collect(Collectors.toList());
        return orderIds;
    }
}
```

위와같은 방법을 사용하는 경우 최초 루트조회 (findOrders()) 1번, 각각의 Item조회(findOrderItemMap()) 1번으로 총 2번의 쿼리로 최적화 할수있다.

<br>

# V6: JPA에서 DTO로 직접 조회, 플랫 데이터 최적화

```java
@RestController
@RequiredArgsConstructor
public class OrderApiController {

    private final OrderRepository orderRepository;

    ...
    @GetMapping("/api/v6/orders")
    public Result ordersV6() {
        List<OrderFlatDto> flatDtos = orderQueryRepository.findAllByDto_flat();
        List<OrderQueryDto> collect = flatDtos.stream()
                .collect(groupingBy(o -> new OrderQueryDto(o.getOrderId(),
                                o.getName(), o.getOrderDate(), o.getOrderStatus(), o.getAddress()),
                        mapping(o -> new OrderItemQueryDto(o.getOrderId(),
                                o.getItemName(), o.getOrderPrice(), o.getCount()), toList())
                )).entrySet().stream()
                .map(e -> new OrderQueryDto(e.getKey().getOrderId(),
                        e.getKey().getName(), e.getKey().getOrderDate(), e.getKey().getOrderStatus(),
                        e.getKey().getAddress(), e.getValue()))
                .collect(toList());
        return new Result(collect);
    }
    ...
}

@Repository
@RequiredArgsConstructor
public class OrderQueryRepository {

    private final EntityManager em;

    public List<OrderFlatDto> findAllByDto_flat() {
        return em.createQuery(
                "select new jpabook.jpashop.repository.order.query.OrderFlatDto(o.id, m.name, o.orderDate, o.status, d.address, i.name, oi.orderPrice, oi.count) "  +
                        "from Order o " +
                        "join o.member m " +
                        "join o.delivery d " +
                        "join o.orderItems oi " +
                        "join oi.item i", OrderFlatDto.class)
                .getResultList();

    }
}

@Data
public class OrderFlatDto {

    private Long orderId;
    private String name;
    private LocalDateTime orderDate; //주문시간
    private Address address;
    private OrderStatus orderStatus;
    private String itemName;//상품 명
    private int orderPrice; //주문 가격
    private int count; //주문 수량

    public OrderFlatDto(Long orderId, String name, LocalDateTime orderDate,
                        OrderStatus orderStatus, Address address, String itemName, int orderPrice, int
                                count) {
        this.orderId = orderId;
        this.name = name;
        this.orderDate = orderDate;
        this.orderStatus = orderStatus;
        this.address = address;
        this.itemName = itemName;
        this.orderPrice = orderPrice;
        this.count = count;
    }
}

@Data
@EqualsAndHashCode(of = "orderId")
public class OrderQueryDto {

    private Long orderId;
    private String name;
    private LocalDateTime orderDate;
    private OrderStatus orderStatus;
    private Address address;
    private List<OrderItemQueryDto> orderItems;

    public OrderQueryDto(Long orderId, String name, LocalDateTime orderDate, OrderStatus orderStatus, Address address) {
        this.orderId = orderId;
        this.name = name;
        this.orderDate = orderDate;
        this.orderStatus = orderStatus;
        this.address = address;
    }

    public OrderQueryDto(Long orderId, String name, LocalDateTime orderDate, OrderStatus orderStatus, Address address, List<OrderItemQueryDto> orderItems) {
        this.orderId = orderId;
        this.name = name;
        this.orderDate = orderDate;
        this.orderStatus = orderStatus;
        this.address = address;
        this.orderItems = orderItems;
    }
}

@Data
public class OrderItemQueryDto {

    private Long orderId;
    private String itemName;
    private int orderPrice;
    private int count;

    public OrderItemQueryDto(Long orderId, String itemName, int orderPrice, int count) {
        this.orderId = orderId;
        this.itemName = itemName;
        this.orderPrice = orderPrice;
        this.count = count;
    }
}
```

V6의 경우 쿼리는 단 1번으로 모든 데이터를 가져올수 있지만, 조인으로 인해 DB에서 애플리케이션에 전달하는 데이터에 중복 데이터가 추가되므로 상황에 따라 V5 보다 더 느릴 수 도 있다. 그리고 애플리케이션에서 추가 작업이 크고 페이징 불가능하다.

<br>

# 정리

<br>

## 엔티티 조회

- 엔티티를 조회해서 그대로 반환: V1
- 엔티티 조회 후 DTO로 변환: V2
- 페치 조인으로 쿼리 수 최적화: V3
- 컬렉션 페이징과 한계 돌파: V3.1
    - 컬렉션은 페치 조인시 페이징이 불가능
    - ToOne 관계는 페치 조인으로 쿼리 수 최적화
    - 컬렉션은 페치 조인 대신에 지연 로딩을 유지하고, hibernate.default_batch_fetch_size , @BatchSize 로 최적화
- DTO 직접 조회
    - JPA에서 DTO를 직접 조회: V4
    - 컬렉션 조회 최적화 - 일대다 관계인 컬렉션은 IN 절을 활용해서 메모리에 미리 조회해서 최적화: V5
    - 플랫 데이터 최적화 - JOIN 결과를 그대로 조회 후 애플리케이션에서 원하는 모양으로 직접 변환: V6

<br>

## 최적화 권장 순서

1. 엔티티 조회 방식으로 우선 접근
    1. 페치조인으로 쿼리 수를 최적화
    2. 컬렉션 최적화
        1. 페이징 필요 hibernate.default_batch_fetch_size , @BatchSize 로 최적화
        2. 페이징 필요X 페치 조인 사용
2. 엔티티 조회 방식으로 해결이 안되면 DTO 조회 방식 사용
3. DTO 조회 방식으로 해결이 안되면 NativeSQL or 스프링 JdbcTemplate

<br>

> 참고: 엔티티 조회 방식은 페치 조인이나, hibernate.default_batch_fetch_size , @BatchSize 같이 코드를 거의 수정하지 않고, 옵션만 약간 변경해서, 다양한 성능 최적화를 시도할 수 있다. 반면에 DTO를 직접 조회하는 방식은 성능을 최적화 하거나 성능 최적화 방식을 변경할 때 많은 코드를 변경해야 한다. <br>

> 참고: 개발자는 성능 최적화와 코드 복잡도 사이에서 줄타기를 해야 한다. 항상 그런 것은 아니지만, 보통 성능 최적화는 단순한 코드를 복잡한 코드로 몰고간다. <br> 엔티티 조회 방식은 JPA가 많은 부분을 최적화 해주기 때문에, 단순한 코드를 유지하면서, 성능을 최적화 할 수 있다. <br>
반면에 DTO 조회 방식은 SQL을 직접 다루는 것과 유사하기 때문에, 둘 사이에 줄타기를 해야 한다.

<br>

## DTO 조회 방식의 선택지

- DTO로 조회하는 방법도 각각 장단이 있다. V4, V5, V6에서 단순하게 쿼리가 1번 실행된다고 V6이 항상 좋은 방법인 것은 아니다.
- V4는 코드가 단순하다. 특정 주문 한건만 조회하면 이 방식을 사용해도 성능이 잘 나온다. 예를 들어서 조회한 Order 데이터가 1건이면 OrderItem을 찾기 위한 쿼리도 1번만 실행하면 된다.
- V5는 코드가 복잡하다. 여러 주문을 한꺼번에 조회하는 경우에는 V4 대신에 이것을 최적화한 V5 방식을 사용해야 한다. 예를 들어서 조회한 Order 데이터가 1000건인데, V4 방식을 그대로 사용하면, 쿼리가 총 1 + 1000번 실행된다. 여기서 1은 Order 를 조회한 쿼리고, 1000은 조회된 Order의 row 수다. V5 방식으로 최적화 하면 쿼리가 총 1 + 1번만 실행된다. 상황에 따라 다르겠지만 운영 환경에서 100배 이상의 성능 차이가 날 수 있다.
- V6는 완전히 다른 접근방식이다. 쿼리 한번으로 최적화 되어서 상당히 좋아보이지만, Order를 기준으로 페이징이 불가능하다. 실무에서는 이정도 데이터면 수백이나, 수천건 단위로 페이징 처리가 꼭 필요하므로, 이 경우 선택하기 어려운 방법이다. 그리고 데이터가 많으면 중복 전송이 증가해서 V5와 비교해서 성능 차이도 미비하다.

<br><br><br>
## References 및 사진 출처

> __김영한의 실전! 스프링 부트와 JPA 활용2 - API 개발과 성능 최적화__