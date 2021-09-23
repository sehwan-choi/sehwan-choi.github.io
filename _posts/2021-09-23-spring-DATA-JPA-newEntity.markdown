---
layout: post
title:  "spring Data JPA 새로운 엔티티를 구별하는 방법"
subtitle:   "spring Data JPA 새로운 엔티티를 구별하는 방법"
date:   2021-09-23 21:29:27 +0900
categories: spring
tags: spring JPA ORM Mapping String-Data-JPA newEntity
comments: true
---


<br>

- 목차
	- [](#)
    
<br>

# 새로운 엔티티를 구별하는 방법

> save() 메서드 <br>
> 새로운 엔티티면 저장( persist ) <br>
> 새로운 엔티티가 아니면 병합( merge ) <br>

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

위와같이 JPA save() 구현체를 보면 isNew로 새로운 엔티티인지 확인을 한다. 이후 새로운 엔티티라면 persist, 새로운 엔티티가 아니라면 merge 한다.

- 새로운 엔티티를 판단하는 기본 전략
    - 식별자가 객체일 때 null 로 판단
    - 식별자가 자바 기본 타입일 때 0 으로 판단
    - Persistable 인터페이스를 구현해서 판단 로직 변경 가능

<br><br>

만일 JPA 식별자 생성 전략이 @GenerateValue가 아니라 직접 할당 이라면 save()를 호출하면 어떻게 될까?

```java
@Entity
public class Item {

    @Id
    private String id;

    public Item() {
    }

    public Item(String id) {
        this.id = id;
    }

    ...
}

@SpringBootTest
class ItemRepositoryTest {

    @Autowired
    ItemRepository itemRepository;

    @Test
    public void save() {
        Item item = new Item("A");
        itemRepository.save(item);
    }

}
```

- 결과

```
select
    item0_.id as id1_0_0_ 
from
    item item0_ 
where
    item0_.id=?

...

insert 
into
    item
    (id) 
values
    (?)
```

위 결과와 같이 JPA 식별자 생성 전략이 @Id 만 사용해서 직접 할당이면 이미 식별자 값이 있는 상태로 save() 를 호출한다. 따라서 이 경우 merge() 가 호출된다. merge() 는 우선 DB를 호출해서 값을 확인하고, DB에 값이 없으면 새로운 엔티티로 인지하므로 매우 비효율 적이다. 따라서 Persistable 를 사용해서 새로운 엔티티 확인 여부를 직접 구현하게는 효과적이다.

> 참고: JPA 식별자 생성 전략이 @GenerateValue 면 save() 호출 시점에 식별자가 없으므로 새로운 엔티티로 인식해서 저장(persist)으로 정상 동작한다.

<br><br>

# Persistable 구현

```java
@Entity
@EntityListeners(AuditingEntityListener.class)
public class Item implements Persistable<String> {

    @Id
    private String id;

    @CreatedDate
    private LocalDateTime createdDate;

    public Item() {
    }

    public Item(String id) {
        this.id = id;
    }

    @Override
    public String getId() {
        return id;
    }

    @Override
    public boolean isNew() {
        return createdDate == null;
    }
}

@EnableJpaAuditing
@SpringBootApplication
public class DataJpaApplication {

	public static void main(String[] args) {
		SpringApplication.run(DataJpaApplication.class, args);
	}
}

@SpringBootTest
class ItemRepositoryTest {

    @Autowired
    ItemRepository itemRepository;

    @Test
    public void save() {
        Item item = new Item("A");
        itemRepository.save(item);
    }

}
```

- 결과

```
insert 
into
    item
    (created_date, id) 
values
    (?, ?)
```

위 결과와 같이 select후 insert가 아닌, insert만 하는것을 볼수있다. merge가 아닌 persist 정상 동작 확인! <br>
Persistable를 implement한후 getId(), isNew() 를 Override하여 구현하면 된다. Persistable`<ID>` 에서 ID는 Entity의 @Id의 타입을 넣어준다. 그리고 Auditing 기능을 사용하기 위해 메인 클래스에 @EnableJpaAuditing, 엔티티 클래스에 @EntityListeners(AuditingEntityListener.class)를 추가한다. 그리고 등록시간( @CreatedDate )을 조합해서 사용하면 이 필드로 새로운 엔티티 여부를 편리하게 확인할
수 있다. (@CreatedDate에 값이 없으면 새로운 엔티티로 판단)

<br><br><br>
## References 및 사진 출처

> __김영한의 실전! 스프링 데이터 JPA__