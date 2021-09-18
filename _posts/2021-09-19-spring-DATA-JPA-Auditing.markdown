---
layout: post
title:  "spring Data JPA Auditing"
subtitle:   "spring Data JPA Auditing"
date:   2021-09-19 01:59:27 +0900
categories: spring
tags: spring JPA ORM Mapping String-Data-JPA Auditing
comments: true
---


<br>

- 목차
	- [Auditing](#auditing)
	- [순수 JPA 사용할 경우](#순수-jpa-사용할-경우)
	- [스프링 데이터 JPA 사용](#스프링-데이터-jpa-사용)
	- [전체 적용](#전체-적용)
    
<br>

# Auditing

- 엔티티를 생성, 변경할 때 변경한 사람과 시간을 추적하고 싶으면?
    - 등록일
    - 수정일
    - 등록자
    - 수정자

# 순수 JPA 사용할 경우

```java
@MappedSuperclass
@Getter
public class JpaBaseEntity {

    @Column(updatable = false)
    private LocalDateTime createdDate;
    private LocalDateTime lastModifiedDate;

    @PrePersist
    public void prePersist() {
        LocalDateTime now = LocalDateTime.now();
        this.createdDate = now;
        this.lastModifiedDate = now;
    }

    @PreUpdate
    public void preUpdate() {
        lastModifiedDate = LocalDateTime.now();
    }
}

@Entity
@Getter @Setter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
@ToString(of = {"id","username","age"})
@NamedQuery(
        name="Member.findByUsername",
        query ="select m from Member m where m.username = :username"
)
public class Member extends JpaBaseEntity {

    @Id
    @GeneratedValue
    @Column(name = "MEMBER_ID")
    private Long id;

    private String username;

    private int age;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "TEAM_ID")
    private Team team;

    ...
}

@SpringBootTest
@Transactional
@Rollback(false)
class MemberTest {

    @PersistenceContext
    EntityManager em;

    @Autowired
    MemberRepository memberRepository;

    @Test
    public void JpaEventBaseEntity() throws Exception {
        //given
        Member member = new Member("member1");
        memberRepository.save(member);  // @RrePersist 발생

        Thread.sleep(100);

        member.setUsername("member2");

        em.flush(); // @PreUpdate 발생
        em.clear();

        //when
        Member findMember = memberRepository.findById(member.getId()).get();

        //then
        System.out.println("getCreatedDate = " + findMember.getCreatedDate());
        System.out.println("getLastModifiedDate = " + findMember.getLastModifiedDate());
    }
}
```

- 결과

```
findMember = 2021-09-19T02:02:31.212449
findMember = 2021-09-19T02:02:31.370331
```

JpaBaseEntity에서 @PrePersist 어노테이션을 사용하면 Persist할 때 해당 메소드를 실행하고, @PreUpdate 어노테이션을 사용하면 update가 되었을때 해당 메소드를 실행한다.

<br><br>

# 스프링 데이터 JPA 사용

```java
@EnableJpaAuditing  <-- @EnableJpaAuditing 적용
@SpringBootApplication
public class DataJpaApplication {

	public static void main(String[] args) {
		SpringApplication.run(DataJpaApplication.class, args);
	}

    // createdBy, lastModifiedBy를 세팅하기위해 임시로 UUID를 랜덤으로 만들어서 넣기 위함
    // 실제로는 세션에서 데이터를 가져와서 넣으면 된다.
	@Bean
	public AuditorAware<String> auditorProvider() {
		return () -> Optional.of(UUID.randomUUID().toString());
	}
}

@EntityListeners(AuditingEntityListener.class)  <-- @EntityListeners 적용
@MappedSuperclass
@Getter
public class BaseEntity {

    @CreatedDate                // persist시 createdDate에 값을 넣어준다
    @Column(updatable = false)  // update를 금지한다.
    private LocalDateTime createdDate;

    @LastModifiedDate           //  update시 lastModifiedDate에 값을 넣어준다
    private LocalDateTime lastModifiedDate;

    @CreatedBy                  // persist시 createdBy에 값을 넣어준다. DataJpaApplication에 auditorProvider를 호출하여 데이터를 넣는다.
    @Column(updatable = false)  // update를 금지한다.
    private String createdBy;

    @LastModifiedBy           //  update시 lastModifiedBy 값을 넣어준다. DataJpaApplication에 auditorProvider를 호출하여 데이터를 넣는다.
    private String lastModifiedBy;
}

@Entity
@Getter @Setter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
@ToString(of = {"id","username","age"})
@NamedQuery(
        name="Member.findByUsername",
        query ="select m from Member m where m.username = :username"
)
public class Member extends BaseEntity {

    @Id
    @GeneratedValue
    @Column(name = "MEMBER_ID")
    private Long id;

    private String username;

    private int age;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "TEAM_ID")
    private Team team;

    ...
}

@SpringBootTest
@Transactional
@Rollback(false)
class MemberTest {

    @PersistenceContext
    EntityManager em;

    @Autowired
    MemberRepository memberRepository;

    @Test
    public void eventBaseEntity() throws Exception {
        //given
        Member member = new Member("member1");
        memberRepository.save(member);  // @CreatedDate, @CreatedBy 발생

        Thread.sleep(100);

        member.setUsername("member2");

        em.flush(); // @LastModifiedDate, @LastModifiedBy 발생
        em.clear();

        //when
        Member findMember = memberRepository.findById(member.getId()).get();

        //then
        System.out.println("getCreatedDate = " + findMember.getCreatedDate());
        System.out.println("getLastModifiedDate = " + findMember.getLastModifiedDate());
        System.out.println("getCreatedBy = " + findMember.getCreatedBy());
        System.out.println("getLastModifiedBy = " + findMember.getLastModifiedBy());
    }
}
```

- 결과

```
getCreatedDate = 2021-09-19T02:06:27.917959
getLastModifiedDate = 2021-09-19T02:06:28.078103
getCreatedBy = 9cd9b1ec-e125-4c14-890f-0c43abe95e2b
getLastModifiedBy = a7bc603a-2165-4931-a9b6-e90b9a7b34a0
```

Auditing을 사용하기 위해선 @EnableJpaAuditing를 스프링 부트 설정 클래스에 적용해야한다.(DataJpaApplication.class) 또한 Auditing을 적용할 Entity에 @EntityListeners(AuditingEntityListener.class) 어노테이션을 적용하고 @CreatedDate(등록시간), @LastModifiedDate(수정시간), @CreatedBy(등록자), @LastModifiedBy(수정자) 어노테이션을 사용한다. 등록자, 수정자를 처리해주는 AuditorAware 스프링 빈 등록

```java
@Bean
public AuditorAware<String> auditorProvider() {
 return () -> Optional.of(UUID.randomUUID().toString());
}
```

실무에서는 세션 정보나, 스프링 시큐리티 로그인 정보에서 ID를 받는다.

<br>

> 참고: 실무에서 대부분의 엔티티는 등록시간, 수정시간이 필요하지만, 등록자, 수정자는 없을 수도 있다. 그래서 다음과 같이 Base 타입을 분리하고, 원하는 타입을 선택해서 상속한다.

```java
public class BaseTimeEntity {

    @CreatedDate
    @Column(updatable = false)
    private LocalDateTime createdDate;

    @LastModifiedDate
    private LocalDateTime lastModifiedDate;
}

public class BaseEntity extends BaseTimeEntity {

    @CreatedBy
    @Column(updatable = false)
    private String createdBy;

    @LastModifiedBy
    private String lastModifiedBy;
}
```

> 참고: 저장시점에 등록일, 등록자는 물론이고, 수정일, 수정자도 같은 데이터가 저장된다. 데이터가 중복 저장되는 것 같지만, 이렇게 해두면 변경 컬럼만 확인해도 마지막에 업데이트한 유저를 확인 할 수 있으므로 유지보수 관점에서 편리하다. 이렇게 하지 않으면 변경 컬럼이 null 일때 등록 컬럼을 또 찾아야 한다.

> 참고로 수정시점은 필요없고, 저장시점에 저장데이터만 입력하고 싶으면 @EnableJpaAuditing(modifyOnCreate = false) 옵션을 사용하면 된다

<br><br>

# 전체 적용

@EntityListeners(AuditingEntityListener.class) 를 생략하고 스프링 데이터 JPA 가 제공하는 이벤트를 엔티티 전체에 적용하려면 META-INF/orm.xml에 다음과 같이 등록하면 된다.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<entity-mappings xmlns="http://xmlns.jcp.org/xml/ns/persistence/orm"
 xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
 xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/persistence/
orm http://xmlns.jcp.org/xml/ns/persistence/orm_2_2.xsd"
 version="2.2">
    <persistence-unit-metadata>
        <persistence-unit-defaults>
            <entity-listeners>
                <entity-listener class="org.springframework.data.jpa.domain.support.AuditingEntityListener"/>
            </entity-listeners>
        </persistence-unit-defaults>
    </persistence-unit-metadata>
 
</entity-mappings>

```

<br><br><br>
## References 및 사진 출처

> __김영한의 실전! 스프링 데이터 JPA__