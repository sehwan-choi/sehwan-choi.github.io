---
layout: post
title:  "spring Data JPA"
subtitle:   "spring Data JPA"
date:   2021-09-17 02:45:27 +0900
categories: spring
tags: spring JPA ORM Mapping String-Data-JPA
comments: true
---


<br>

- 목차
	- [Spring Data JPA](#spring-data-jpa)
	- [주요 메서드](#주요-메서드)
	- [상속 구조](#상속-구조)
    
<br>

# Spring Data JPA

Spring Data JPA는 스프링에서 JPA를 편리하게 사용할 수 있도록 지원하는 프로젝트다. <br>
데이터 접근 계층을 개발할 때 지루하게 반복되는 CRUD 문제를 세련된 방법으로 해결할 수 있게 해준다. <br>

- CRUD 처리를 위한 공통 인터페이스 제공한다.
- 인터페이스만 작성하면 동적으로 구현체를 생성해서 주입해준다.
- 따라서 인터페이스만 작성해도 개발을 완료할 수 있다.

```java
public interface MemberRepository extends JpaRepository<Member, Long> {

}
```

위 소스와 같이 JpaRepository <Entity 명, PK 타입> 를 extends 하게 되면 끝이다.

<br><br>

# 주요 메서드

JpaRepository interface를 상속 받으면 사용할 수 있는 주요 메서드

- save(S) : 새로운 엔티티는 저장하고 이미 잇는 엔티티는 수정 (식별자 값이 없으면 em.persist(), 있으면 em.merge() 호출)

- delete(T) : 엔티티 하나를 삭제 (내부에서 em.remove() 호출)

- findOne(ID) : 엔티티 하나를 조회 (내부에서 em.find() 호출)

- getOne(ID) : 엔티티를 프록시로 조회 (내부에서 em.getReference() 호출)

- findAll(..) : 모든 엔티티를 조회 (sort 또는 pageable 조건을 파라미터로 제공)

<br><br>

# 상속 구조

```
[스프링 데이터 모듈]

      Repository

          |

    CrudRepository

          |

PagingAndSortingRepository

          |

[스프링 데이터 JPA 모듈]

          |    

JpaRepository (+ 추가로 JPA에 특화된 기능 제공)
```

<br><br><br>
## References 및 사진 출처

> __김영한의 실전! 스프링 데이터 JPA__