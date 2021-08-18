---
layout: post
title:  "spring JPA 영속성 전이(CASCADE)와 고아 객체"
subtitle:   "spring JPA CASCADE"
date:   2021-08-19 02:10:27 +0900
categories: spring
tags: spring JPA ORM Mapping CASCADE
comments: true
---


<br>

- 목차
	- [영속성 전이(CASCADE)](#영속성-전이cascade)
	    - [CASCADE 주의!](#cascade-주의)
	    - [CASCADE의 종류](#cascade의-종류)
	- [고아객체](#고아객체)
	    - [주의](#주의)
<br>

# 영속성 전이(CASCADE)

- 특정 엔티티를 영속 상태로 만들 때 연관된 엔티티도 함께 영속
상태로 만들도 싶을 때
- 예: 부모 엔티티를 저장할 때 자식 엔티티도 함께 저장

![그림1](https://sehwan-choi.github.io/assets/img/spring/JPA-MAPPING4/jpa12.jpg)

<br>

- CASCADE 미적용시

```java
@Entity
public class Parent {

    @Id
    @GeneratedValue
    @Column(name = "PARENT_ID")
    private Long id;

    private String name;

    @OneToMany(mappedBy = "parent")
    private List<Child> childList = new ArrayList<>();

    public void addChild(Child child) {
        childList.add(child);
        child.setParent(this);
    }

    ...
}

@Entity
public class Child {

    @Id
    @GeneratedValue
    @Column(name = "CHILD_ID")
    private Long id;

    private String name;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "PARENT_ID")
    private Parent parent;

    ...
}


public class jpaMain {

    public static void main(String[] args) {
        ...

        try {

            Child child1 = new Child();
            child1.setName("child1");
            Child child2 = new Child();
            child2.setName("child2");

            Parent parent = new Parent();
            parent.addChild(child1);
            parent.addChild(child2);

            em.persist(parent);
            em.persist(child1);
            em.persist(child2);

            em.flush();
            em.clear();
        }
        ...
    }
}
```

위 코드와 같이 parent 객체에 child1, child2를 add 하게 된다면 총 3번의 persist가 호출이 된다. 만일 parent만 persist한다면 child1, child2는 DB에 저장이 안될 것이다.

<br>

- CASCADE 적용시

```java
@Entity
public class Parent {

    @Id
    @GeneratedValue
    @Column(name = "PARENT_ID")
    private Long id;

    private String name;

    @OneToMany(mappedBy = "parent", cascade = CascadeType.ALL)  //  CASCADE 적용
    private List<Child> childList = new ArrayList<>();

    public void addChild(Child child) {
        childList.add(child);
        child.setParent(this);
    }

    ...
}

@Entity
public class Child {

    @Id
    @GeneratedValue
    @Column(name = "CHILD_ID")
    private Long id;

    private String name;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "PARENT_ID")
    private Parent parent;

    ...
}


public class jpaMain {

    public static void main(String[] args) {
        ...

        try {

            Child child1 = new Child();
            child1.setName("child1");
            Child child2 = new Child();
            child2.setName("child2");

            Parent parent = new Parent();
            parent.addChild(child1);
            parent.addChild(child2);

            em.persist(parent);

            em.flush();
            em.clear();
        }
        ...
    }
}
```

CASCADE 적용시 부모 객체인 parent만 persist한다면 연관된 child 객체가 자동으로 DB에 저장이 된다.

<br>

## CASCADE 주의!

- 영속성 전이는 연관관계를 매핑하는 것과 아무 관련이 없음
- 엔티티를 영속화할 때 연관된 엔티티도 함께 영속화하는 편리함을 제공할 뿐
- 참조하는 곳이 하나일때 사용!!
    - Parent - Child 관계에서 다른 Member가 Child를 참조한다면 사용하면 안됨
    - 만일 Parent - Child 관계에서 Child가 다른 Member를 참조한다면 상관없음

<br>

## CASCADE의 종류

- __ALL: 모두 적용__
- __PERSIST: 영속__
- __REMOVE: 삭제__
- MERGE: 병합
- REFRESH: REFRESH
- DETACH: DETACH

<br><br>

# 고아객체

고아객체란 부모 엔티티와 연관관계가 끊어진 자식을 말한다. <br>

- orphanRemoval = true 옵션으로 사용가능하다.
    - 예) 
    @OneToMany(mappedBy = "parent", cascade = CascadeType.ALL, orphanRemoval = true)

```java
@Entity
public class Parent {

    @Id
    @GeneratedValue
    @Column(name = "PARENT_ID")
    private Long id;

    private String name;

    @OneToMany(mappedBy = "parent", cascade = CascadeType.ALL, orphanRemoval = true)    //  orphanRemoval 적용
    private List<Child> childList = new ArrayList<>();

    public void addChild(Child child) {
        childList.add(child);
        child.setParent(this);
    }
    ...
}

@Entity
public class Child {

    @Id
    @GeneratedValue
    @Column(name = "CHILD_ID")
    private Long id;

    private String name;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "PARENT_ID")
    private Parent parent;

    ...
}


public class jpaMain {

    public static void main(String[] args) {
        try {

            Child child1 = new Child();
            child1.setName("child1");
            Child child2 = new Child();
            child2.setName("child2");


            Parent parent = new Parent();
            parent.addChild(child1);
            parent.addChild(child2);

            em.persist(parent);

            em.flush();
            em.clear();

            Parent parent1 = em.find(Parent.class, parent.getId());
            parent1.getChildList().remove(0);

        }
        ...
    }
}
```
```
delete 
        from
            Child 
        where
            CHILD_ID=?
```

위 코드에서 parent의 자식으로 있었던 child가 list에서 삭제되면 orphanRemoval 옵션으로 인해 child가 삭제 된다.

<br>

## 주의

- 참조가 제거된 엔티티는 다른 곳에서 참조하지 않는 고아 객체로 보고 삭제하는 기능
- 참조하는 곳이 하나일 때 사용해야함!
- 특정 엔티티가 개인 소유할 때 사용
- @OneToOne, @OneToMany만 가능
- 참고: 개념적으로 부모를 제거하면 자식은 고아가 된다. 따라서 고아 객체 제거 기능을 활성화 하면, 부모를 제거할 때 자식도 함께 제거된다. 이것은 CascadeType.REMOVE처럼 동작한다.

<br><br><br>
## References 및 사진 출처

> __김영한의 자바 ORM 표준 JPA 프로그래밍 - 기본편__