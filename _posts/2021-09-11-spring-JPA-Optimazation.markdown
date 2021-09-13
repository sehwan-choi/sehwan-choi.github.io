---
layout: post
title:  "spring JPA JQPL 지연 로딩과 조회 성능 최적화"
subtitle:   "spring JPA JPQL 지연 로딩과 조회 성능 최적화"
date:   2021-09-11 02:31:27 +0900
categories: spring
tags: spring JPA ORM Mapping JPQL 지연 로딩과 조회 성능 최적화
comments: true
---


<br>

- 목차
	- [](#)
    
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
        List<Order> all = orderRepository.findAllByString(new OrderSearch());
        
        for (Order order : all) {
            order.getMember().getName();        //  LAZY 강제초기화(Member Proxy 초기화)
            order.getDelivery().getAddress();   //  LAZY 강제초기화(Delivery Proxy 초기화)
        }

        return all;
    }
}
```

- 문제점

1. orderRepository.findAllByString 함수 호출시 Member, orderItem, delivery를 가져오게 되는데 모두 LAZY LOADING으로 설정 되어 있기 때문에 위 함수 호출 시점에는 실제 데이터가 아닌 Proxy 객체가 들어가 있게 된다. 이 프록시 객체를 json으로 어떻게 생성해야 하는지 모르기 때문에 에러가 발생한다. <br>
에러내용 : [com.fasterxml.jackson.databind.exc.InvalidDefinitionException: No serializer found for class org.hibernate.proxy.pojo.bytebuddy.ByteBuddyInterceptor and no properties discovered to create BeanSerializer (to avoid exception, disable SerializationFeature.FAIL_ON_EMPTY_BEANS) (through reference chain: java.util.ArrayList[0]->jpabook.jpashop.domain.Order["member"]->jpabook.jpashop.domain.Member$HibernateProxy$Y7sAZ8N9["hibernateLazyInitializer"])]

2. 엔티티를 return 하고 있음. 이렇게 되면 엔티티의 값이 변경 되는 경우 API 스펙이 변경되는 치명적인 단점이 발생

3. 1번에 이유로 Proxy 객체를 강제 초기화 해야하는 문제가 발생생

4. 엔티티를 직접 노출 하는 경우네는 양방향 연관관계가 걸린 곳은 반드시 한곳을 @JsonIgnore 처리 해야한다. 안그러면 양쪽을 서로 호출하면서 무한 루프에 빠지게 된다.

<br>

> 참고: 앞에서 계속 강조했듯이 정말 간단한 애플리케이션이 아니면 엔티티를 API 응답으로 외부로 노출하는 것은 좋지 않다. 따라서 Hibernate5Module 를 사용하기 보다는 DTO로 변환해서 반환하는 것이 더 좋은 방법이다.

<br>

> 주의: 지연 로딩(LAZY)을 피하기 위해 즉시 로딩(EARGR)으로 설정하면 안된다! 즉시 로딩 때문에 연관관계가 필요 없는 경우에도 데이터를 항상 조회해서 성능 문제가 발생할 수 있다.
즉시 로딩으로 설정하면 성능 튜닝이 매우 어려워 진다. 항상 지연 로딩을 기본으로 하고, 성능 최적화가 필요한 경우에는 페치 조인(fetch join)을 사용해야한다(V3 에서 설명)

<br><br><br>
## References 및 사진 출처

> __김영한의 실전! 스프링 부트와 JPA 활용2 - API 개발과 성능 최적화__