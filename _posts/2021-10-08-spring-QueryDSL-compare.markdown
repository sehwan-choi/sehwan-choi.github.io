---
layout: post
title:  "spring Querydsl 순수 JPA와 Querydsl 코드 비교"
subtitle:   "spring Querydsl 순수 JPA와 Querydsl 코드 비교"
date:   2021-10-08 02:20:27 +0900
categories: spring
tags: spring JPA ORM Mapping QueryDSL JPA,Querydsl Comapre
comments: true
---


<br>

- 목차
	- [순수 JPA와 Querydsl 코드 비교](#순수-jpa와-querydsl-코드-비교)
	
<br>

# 순수 JPA와 Querydsl 코드 비교

```java
@Repository
public class MemberRepository {

    private EntityManager em;
    private JPAQueryFactory queryFactory;

    public MemberDslRepository(EntityManager em) {
        this.em = em;
        queryFactory = new JPAQueryFactory(em);
    }

    public void save(Member member) {
        em.persist(member);
    }

    //--- findById
    //JPA
    public Optional<Member> findById(Long id) {
        Member member = em.find(Member.class, id);
        return Optional.ofNullable(member);
    }

    //QueryDSL
    public Optional<Member> findById(Long id) {
        Member member = queryFactory
                .selectFrom(QMember.member)
                .where(memberIdEq(id))
                .fetchOne();
        return Optional.ofNullable(member);
    }

    private BooleanExpression memberIdEq(Long id) {
        if (id == null)
            return null;
        return member.id.eq(id);
    }
    //---



    //--- findAll
    //JPA
    public List<Member> findAll() {
        return em.createQuery("select m from Member m", Member.class)
                .getResultList();
    }

    //QueryDSL
    public List<Member> findAll() {
        return queryFactory
                .selectFrom(member)
                .fetch();
    }
    //---



    //--- findByUsername
    //JPA
    public List<Member> findByUsername(String username) {
        return em.createQuery("select m from Member m where m.username = :username", Member.class)
                .setParameter("username", username)
                .getResultList();
    }

    //QueryDSL
    public List<Member> findByUsername(String username) {
        return queryFactory
                .selectFrom(member)
                .where(usernameEq(username))
                .fetch();
    }

    private BooleanExpression usernameEq(String username) {
        if (username == null)
            return null;
        return member.username.eq(username);
    }
    //---
}
```

간단한 쿼리같은 경우 순수JPA가 더 코드가 깔끔하지만, 동적쿼리와 같이 좀 더 어렵고 복잡한 쿼리를 사용해야할 경우에는 QueryDSL이 더 가독성이 좋고 사용하기 편하다. <br>

또한, 순수 JPA는 컴파일 시점에 createQuery의 JPQL이 문자열이기 때문에 오타가 있어도 발견되지 않지만, QueryDSL은 메서드를 사용하기 때문에 오타가 있으면 컴파일 시점에 확인할수 있다.

<br><br><br>
## References 및 사진 출처

> __김영한의 실전! Querydsl__