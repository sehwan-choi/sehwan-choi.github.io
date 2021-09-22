---
layout: post
title:  "spring Data JPA 구현체 분석"
subtitle:   "spring Data JPA 구현체 분석"
date:   2021-09-23 02:29:27 +0900
categories: spring
tags: spring JPA ORM Mapping String-Data-JPA Analysis
comments: true
---


<br>

- 목차
	- [스프링 데이터 JPA 구현체 분석](#스프링-데이터-jpa-구현체-분석)
    
<br>

# 스프링 데이터 JPA 구현체 분석

- 스프링 데이터 JPA가 제공하는 공통 인터페이스의 구현체
- org.springframework.data.jpa.repository.support.SimpleJpaRepositor

```java
@Repository
@Transactional(readOnly = true)
public class SimpleJpaRepository<T, ID> ...{

    @Transactional
    public <S extends T> S save(S entity) {
        if (entityInformation.isNew(entity)) {
        em.persist(entity);
            return entity;
        } else {
            return em.merge(entity);
        }
    }
 ...
}
```

- @Repository 적용: JPA 예외를 스프링이 추상화한 예외로 변환
- @Transactional 트랜잭션 적용
    - JPA의 모든 변경은 트랜잭션 안에서 동작
    - 스프링 데이터 JPA는 변경(등록, 수정, 삭제) 메서드를 트랜잭션 처리
    - 서비스 계층에서 트랜잭션을 시작하지 않으면 리파지토리에서 트랜잭션 시작
    - 서비스 계층에서 트랜잭션을 시작하면 리파지토리는 해당 트랜잭션을 전파 받아서 사용
    - 그래서 스프링 데이터 JPA를 사용할 때 트랜잭션이 없어도 데이터 등록, 변경이 가능했음(사실은 트랜잭션이 리포지토리 계층에 걸려있는 것임)

<br>

- @Transactional(readOnly = true)
- 데이터를 단순히 조회만 하고 변경하지 않는 트랜잭션에서 readOnly = true 옵션을 사용하면 플러시를 생략해서 약간의 성능 향상을 얻을 수 있음

<br>

> 매우 중요!!! <br>
> save() 메서드 <br>
> 새로운 엔티티면 저장( persist ) <br>
> 새로운 엔티티가 아니면 병합( merge )한다. <br>
> 병합은 모든 데이터가 수정된다. 데이터가 없을경우 null이 들어가게 되므로 매우 위험하다.

<br><br><br>
## References 및 사진 출처

> __김영한의 실전! 스프링 데이터 JPA__