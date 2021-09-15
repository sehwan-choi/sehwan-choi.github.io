---
layout: post
title:  "spring JPA OSIV"
subtitle:   "spring JPA OSIV"
date:   2021-09-16 02:45:27 +0900
categories: spring
tags: spring JPA ORM Mapping OSIV
comments: true
---


<br>

- 목차
	- [OSIV와 성능 최적화](#osiv와-성능-최적화)
	- [Spring Boot 실행시 WARN 메세지](#spring-boot-실행시-warn-메세지)
	- [OSIV 설정 방법](#osiv-설정-방법)
	- [OSIV ON](#osiv-on)
	- [OSIV OFF](#osiv-off)
	- [에러 수정](#에러-수정)
	    - [커맨드와 쿼리 분리](#커맨드와-쿼리-분리)
	    - [비즈니스로직 분리](#비즈니스로직-분리)
    
<br>

# OSIV와 성능 최적화

<br>

JPA가 없던 시절, 하이버네이트를 사용할 때는 EntitiyMenager가 아닌 Session이 였다. 그래서 이름이 Open Session In View 이였는데 JPA의 등장으로 인해 Open EntityManager In View 으로 해야 하지만 관례 상으로 OSIV라 한다. <br>

OSIV 전략이란 데이터 베이스 커넥션과 영속성 컨텍스트를 언제까지 유지할 것 인지에 대한 전략이다. 

<br><br>

# Spring Boot 실행시 WARN 메세지

<br>

```
2021-09-16 02:53:58.673  WARN 19784 --- [           main] JpaBaseConfiguration$JpaWebConfiguration : spring.jpa.open-in-view is enabled by default. Therefore, database queries may be performed during view rendering. Explicitly configure spring.jpa.open-in-view to disable this warning
```

<br>

스프링 부트 실행시 위와 같은 WARN 메세지를 본적이 있는가? <br>
Spring Boot에서는 spring.jpa.open-in-view를 기본 값으로 true로 설정하고 있는데, 기본 값으로 사용하고 있다는 경고 메세지 이다. 이렇게 기본 값으로 사용할 경우에 경고메세지를 띄운다는 것은 OSIV가 기본값으로 되어있으니 확인하여 미연에 심각한 장애를 방지 할수 있도록 하기위함이 아닐까라는 생각이 든다.

<br>

# OSIV 설정 방법

<br>

```yml
  jpa:
    hibernate:
      ddl-auto: create
    properties:
      hibernate:
        show_sql: true
        format_sql: true
        default_batch_fetch_size: 100
    open-in-view: false     <-- OSIV 설정
```

<br>

위와같이 application.yml에 jpa.open-in-view: false로 설정할수 있다. (기본값은 true이다)

<br><br>

# OSIV ON

<br>

![그림1](https://sehwan-choi.github.io/assets/img/spring/OSIV/jpa1.jpg)

<br>

application.yml에 jpa.open-in-view 관련된 설정않으면 기본값이 true다. <br>
OSIV 전략은 트랜잭션 시작처럼 최초 데이터베이스 커넥션 시작 시점부터 API 응답이 끝날 때 까지 영속성 컨텍스트와 데이터베이스 커넥션을 유지한다. 그래서 지금까지 View Template이나 API 컨트롤러에서 지연 로딩이 가능했던 것이다. <br>
지연 로딩은 영속성 컨텍스트가 살아있어야 가능하고, 영속성 컨텍스트는 기본적으로 데이터베이스 커넥션을 유지한다. 이것 자체가 큰 장점이다. <br>
그런데 이 전략은 너무 오랜시간동안 데이터베이스 커넥션 리소스를 사용하기 때문에, 실시간 트래픽이 중요한 애플리케이션에서는 커넥션이 모자랄 수 있다. 이것은 결국 장애로 이어진다. <br>
예를 들어서 컨트롤러에서 외부 API를 호출하면 외부 API 대기 시간 만큼 커넥션 리소스를 반환하지 못하고, 유지해야 한다. <br>

<br><br>

# OSIV OFF

<br>

![그림1](https://sehwan-choi.github.io/assets/img/spring/OSIV/jpa2.jpg)

<br>

application.yml에 jpa.open-in-view: false 로 설정한다.<br>
OSIV를 끄면 트랜잭션을 종료할 때 영속성 컨텍스트를 닫고, 데이터베이스 커넥션도 반환한다. 따라서 커넥션 리소스를 낭비하지 않는다. <br>
OSIV를 끄면 모든 지연로딩을 트랜잭션 안에서 처리해야 한다. 따라서 지금까지 작성한 많은 지연 로딩 코드를 트랜잭션 안으로 넣어야 하는 단점이 있다. 그리고 view template에서 지연로딩이 동작하지 않는다. 결론적으로 트랜잭션이 끝나기 전에 지연 로딩을 강제로 호출해 두어야 한다.

<br>

```java

@RestController
@RequiredArgsConstructor
public class OrderApiController {

    private final OrderRepository orderRepository;
    private final OrderQueryRepository orderQueryRepository;
    ...
    
    @GetMapping("/api/osiv1")
    public List<Order> ordersOsiv() {
        List<Order> all = orderRepository.osivTest();
        for (Order order : all) {
            order.getMember().getName();        //  LAZY 강제초기화(Member Proxy 초기화)
            order.getDelivery().getAddress();   //  LAZY 강제초기화(Delivery Proxy 초기화)
            List<OrderItem> orderItems = order.getOrderItems();         //  LAZY 강제초기화(OrderItem Proxy 초기화)
            orderItems.stream().forEach(o -> o.getItem().getName());    //  LAZY 강제초기화(Item Proxy 초기화)
        }
        return all;
    }
}


@Repository
@RequiredArgsConstructor
@Transactional(readOnly = true)
public class OrderRepository {

    private final EntityManager em;

    ...

    
    public List<Order> osivTest() {
        String query = "select o from Order o join fetch o.member join fetch  o.delivery";

        return em.createQuery(query, Order.class)
                .getResultList();
    }

    ...
}
```

- 결과

```
failed to lazily initialize a collection of role: jpabook.jpashop.domain.Order.orderItems, could not initialize proxy - no Session
```

위 에러가 발생할 것이다. Transaction의 시작과 끝은 OrderRepository에 osivTest() 이다. 그렇기 때문에 osivTest() 메서드가 아닌 OrderApiController에 ordersOsiv()에서는 영속성 컨텍스트를 사용할수 없기 때문에, 지연로딩시 에러가 발생한다.

<br><br>

# 에러 수정

<br>

위와 같이 에러가 발생하는 경우 아래 두가지 방법으로 해결 할수 있다.

<br>

## 커맨드와 쿼리 분리

<br>

- 커맨드

```java
private EntityManager em;

    @Transactional
    public void update(Item item) {
        em.persist(item);
    }
```

위와 같이 update의 경우에는 DB의 데이터에 변경이 발생하지만 아무것도 반환하지 않으므로 커맨드이다.

<br>

- 쿼리

```java
private EntityManager em;

    @Transactional
    public void select(Long id) {
        return em.find(Item.class, id);
    }
```

위와 같이 select는 DB의 데이터에 변경없이 결과값만 반환하므로 쿼리다.

<br>

## 비즈니스로직 분리

<br>

보통 비즈니스 로직은 특정 엔티티 몇개를 등록하거나 수정하는 것이므로 성능이 크게 문제가 되지 않는다. <br>
그런데 복잡한 화면을 출력하기 위한 쿼리는 화면에 맞추어 성능을 최적화 하는 것이 중요하다. 하지만 그 복잡성에 비해 핵심 비즈니스에 큰 영향을 주는 것은 아니다. <br>
그래서 크고 복잡한 애플리케이션을 개발한다면, 이 둘의 관심사를 명확하게 분리하는 선택은 유지보수 관점에서 충분히 의미 있다. <br>
단순하게 설명해서 다음처럼 분리하는 것이다. <br><br>

OrderService: 핵심 비즈니스 로직 <br>
OrderQueryService: 화면이나 API에 맞춘 서비스 (주로 읽기 전용 트랜잭션 사용)

```java

@RestController
@RequiredArgsConstructor
public class OrderApiController {

    private final OrderRepository orderRepository;
    private final OrderQueryRepository orderQueryRepository;
    ...
    
    @GetMapping("/api/osiv2")
    public List<Order> ordersOsiv2() {
        return orderQueryService.osivTest2();
    }
}

@Service
@Transactional(readOnly = true)
@RequiredArgsConstructor
public class OrderQueryService {

    private final OrderRepository orderRepository;

    public List<Order> osivTest2() {
        List<Order> all = orderRepository.osivTest2();

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

위와 같은 경우 화면이나 API에 맞춘 서비스인 OrderQueryService를 따로 두어 관리하는 것도 좋은 방법이다.

<br><br><br>
## References 및 사진 출처

> __김영한의 실전! 스프링 부트와 JPA 활용2 - API 개발과 성능 최적화__