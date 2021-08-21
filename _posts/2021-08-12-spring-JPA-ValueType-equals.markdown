---
layout: post
title:  "spring JPA 값 타입 컬렉션"
subtitle:   "spring JPA ValueType Collection"
date:   2021-08-12 01:22:27 +0900
categories: spring
tags: spring JPA ORM Mapping ValueType Collection
comments: true
---


<br>

- 목차
	- [값 타입 컬렉션](#값-타입-컬렉션)
	- [값 타입 컬렌셕 사용](#값-타입-컬렌셕-사용)
	- [값 타입 컬렉션의 제약사항](#값-타입-컬렉션의-제약사항)
	- [값 타입 컬렉션 대안](#값-타입-컬렉션-대안)
	- [정리](#정리)
	- [결론](#결론)
    
<br>

# 값 타입 컬렉션

<br>

![그림1](https://sehwan-choi.github.io/assets/img/spring/JPA-MAPPING4/jpa27.jpg)

<br>

- 값 타입을 하나 이상 저장할 때 사용
- @ElementCollection, @CollectionTable 사용
- 데이터베이스는 컬렉션을 같은 테이블에 저장할 수 없다. 
- 컬렉션을 저장하기 위한 별도의 테이블이 필요함

```java

@Entity
public class Member{

    @Id
    @GeneratedValue
    @Column(name = "MEMBER_ID")
    private Long id;

    private String username;

    @ElementCollection  // 값타입 컬렉션으로 사용하겠다는 @ElementCollection 사용
    @CollectionTable(name = "FAVORITE_FOOD", joinColumns = @JoinColumn(name = "MEMBER_ID")) //  Table명은 FAVORITE_FOOD, Member의 MEMBER_ID PK값을 FAVORITE_FOOD에 FK로 설정
    @Column(name = "FOOD_NAME") // 컬럼명을 FOOD_NAME으로 설정
    private Set<String> favoriteFoods = new HashSet<>();

    @ElementCollection  // 값타입 컬렉션으로 사용하겠다는 @ElementCollection 사용
    @CollectionTable(name = "ADDRESS",joinColumns = @JoinColumn(name = "MEMBER_ID"))    // TABLE명은 ADDRESS, Member의 MEMBER_ID PK값을 ADDRESS에 FK로 설정
    private List<Address> addressHistory = new ArrayList<>();

    ...
}

@Embeddable
public class Address {

    private String city;
    private String street;
    private String zipcode;

    public Address() {
    }

    public Address(String city, String street, String zipcode) {
        this.city = city;
        this.street = street;
        this.zipcode = zipcode;
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        Address address = (Address) o;
        return Objects.equals(city, address.city) && Objects.equals(street, address.street) && Objects.equals(zipcode, address.zipcode);
    }

    ...
}
```

<br>

# 값 타입 컬렌셕 사용

- 값 타입 저장 예제

```java
public class jpaMain {

    public static void main(String[] args) {

        ...

        try {
            Member member1 = new Member();
            member1.setUsername("NAME");
            member1.setHomeAddress(new Address("HOMECITY", "STREET", "ZIPCODE"));

            member1.getFavoriteFoods().add("치킨");
            member1.getFavoriteFoods().add("족발");
            member1.getFavoriteFoods().add("피자");

            member1.getAddressHistory().add(new Address("OLDCITY1", "STREET", "ZIPCODE"));
            member1.getAddressHistory().add(new Address("OLDCITY2", "STREET", "ZIPCODE"));
            em.persist(member1);
        }
        ...
    }
}
```

![그림1](https://sehwan-choi.github.io/assets/img/spring/JPA-MAPPING4/jpa28.jpg)

![그림1](https://sehwan-choi.github.io/assets/img/spring/JPA-MAPPING4/jpa29.jpg)

![그림1](https://sehwan-choi.github.io/assets/img/spring/JPA-MAPPING4/jpa30.jpg)

ADDRESS, FAVORITE_FOOD의 생명주기는 MEMBER에 소속되게 된다.

<br>

- 값 타입 조회 예제

```java
public class jpaMain {

    public static void main(String[] args) {
        EntityManagerFactory emf = Persistence.createEntityManagerFactory("hello");

        EntityManager em = emf.createEntityManager();

        EntityTransaction tx = em.getTransaction();
        tx.begin();
        try {
            Member member1 = new Member();
            member1.setUsername("NAME");
            member1.setHomeAddress(new Address("HOMECITY", "STREET", "ZIPCODE"));

            member1.getFavoriteFoods().add("치킨");
            member1.getFavoriteFoods().add("족발");
            member1.getFavoriteFoods().add("피자");

            member1.getAddressHistory().add(new Address("OLDCITY1", "STREET", "ZIPCODE"));
            member1.getAddressHistory().add(new Address("OLDCITY2", "STREET", "ZIPCODE"));
            em.persist(member1);

            em.flush();
            em.clear();

            Member findMember = em.find(Member.class, member1.getId());
            List<Address> addressHistory = findMember.getAddressHistory();
            for (Address address : addressHistory) {
                System.out.println("address.getCity() = " + address.getCity());
            }

            Set<String> favoriteFoods = findMember.getFavoriteFoods();
            for (String favoriteFood : favoriteFoods) {
                System.out.println("favoriteFood = " + favoriteFood);
            }
        }
        ...
    }
}
```

    위 코드에서 em.find로 Member를 가져오게 되면 값타입 컬렌션인 ADDRESS, FAVORITE_FOOD값은 빼고 가져오게 된다. 값 타입 컬렉션도 지연 로딩 전략 사용하기 때문에 실제 findMember가 사용될때 System.out.println에서 실제 값이 사용될때 DB에 Query로 데이터를 가져오게 된다.

<br>

- 값 타입 수정 예제

```java
public class jpaMain {

    public static void main(String[] args) {
        EntityManagerFactory emf = Persistence.createEntityManagerFactory("hello");

        EntityManager em = emf.createEntityManager();

        EntityTransaction tx = em.getTransaction();
        tx.begin();
        try {
            Member member1 = new Member();
            member1.setUsername("NAME");
            member1.setHomeAddress(new Address("HOMECITY", "STREET", "ZIPCODE"));

            member1.getFavoriteFoods().add("치킨");
            member1.getFavoriteFoods().add("족발");
            member1.getFavoriteFoods().add("피자");

            member1.getAddressHistory().add(new Address("OLDCITY1", "STREET", "ZIPCODE"));
            member1.getAddressHistory().add(new Address("OLDCITY2", "STREET", "ZIPCODE"));
            em.persist(member1);

            em.flush();
            em.clear();

            Member findMember = em.find(Member.class, member1.getId());
            List<Address> addressHistory = findMember.getAddressHistory();
            for (Address address : addressHistory) {
                System.out.println("address.getCity() = " + address.getCity());
            }

            Set<String> favoriteFoods = findMember.getFavoriteFoods();
            for (String favoriteFood : favoriteFoods) {
                System.out.println("favoriteFood = " + favoriteFood);
            }

            findMember.setHomeAddress(new Address("NewCity", "NewStreet","NewZipcode"));
            
            //  findMember.getAddressHistory().get(0).setCity("newCity1"); -> 절때 이렇게 수정하면 안된다. 아래와 같이 new Address로 새로운 객체를 생성하여 갈아껴야한다. 

            findMember.getFavoriteFoods().remove("치킨");
            findMember.getFavoriteFoods().add("한식");

            findMember.getAddressHistory().remove(new Address("OLDCITY1", "STREET", "ZIPCODE"));
            findMember.getAddressHistory().add(new Address("newCity1", "STREET", "ZIPCODE"));
        }
        ...
    }
}
```
    
    주의할 점은 위 코드와 같이 findMember.getAddressHistory().get(0).setCity("newCity1"); 와 같이 수정하면 안된다는 것이다!. 왜냐하면 공유참조로 인해 다른 값들도 변경될수 있음!

<br>

- 참고: 값 타입 컬렉션은 영속성 전에(Cascade) + 고아 객체 제거 기능을 필수로 가진다고 볼 수 있다. Member가 삭제되면 Member에 소속되어있언 ADDRESS, FAVORITE_FOOD가 같이 삭제되며 persist 할 때 Member에 소속된 ADDRESS, FAVORITE_FOOD가 같이 저장된다.

<br><br>

# 값 타입 컬렉션의 제약사항

- 값 타입은 엔티티와 다르게 식별자 개념이 없다. 
- 값은 변경하면 추적이 어렵다. 
- __값 타입 컬렉션에 변경 사항이 발생하면, 주인 엔티티와 연관된 모든 데이터를 삭제하고, 값 타입 컬렉션에 있는 현재 값을 모두다시 저장한다.__
- 값 타입 컬렉션을 매핑하는 테이블은 모든 컬럼을 묶어서 기본 키를 구성해야 함: null 입력X, 중복 저장X

<br>

# 값 타입 컬렉션 대안

- 실무에서는 상황에 따라 값 타입 컬렉션 대신에 일대다 관계를 고려
- 일대다 관계를 위한 엔티티를 만들고, 여기에서 값 타입을 사용
- 영속성 전이(Cascade) + 고아 객체 제거를 사용해서 값 타입 컬렉션 처럼 사용
- EX) AddressEntity

```java
@Entity
public class AddressEntity {

    @Id
    @GeneratedValue
    private Long id;

    @Embedded
    private Address address;

    public AddressEntity() {
    }

    public AddressEntity(String city, String street, String zipcode) {
        this.address = new Address(city,street, zipcode);
    }

    public AddressEntity(Long id, Address address) {
        this.id = id;
        this.address = address;
    }
}

@Embeddable
public class Address {

    private String city;
    private String street;
    private String zipcode;

    public Address() {
    }

    public Address(String city, String street, String zipcode) {
        this.city = city;
        this.street = street;
        this.zipcode = zipcode;
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        Address address = (Address) o;
        return Objects.equals(city, address.city) && Objects.equals(street, address.street) && Objects.equals(zipcode, address.zipcode);
    }

    @Override
    public int hashCode() {
        return Objects.hash(city, street, zipcode);
    }
}

@Entity
public class Member{

    @Id
    @GeneratedValue
    @Column(name = "MEMBER_ID")
    private Long id;

    private String username;

    @Embedded
    private Period period;

    @Embedded
    private Address homeAddress;

    @ElementCollection
    @CollectionTable(name = "FAVORITE_FOOD", joinColumns = @JoinColumn(name = "MEMBER_ID"))
    @Column(name = "FOOD_NAME")
    private Set<String> favoriteFoods = new HashSet<>();

    @OneToMany(cascade = CascadeType.ALL, orphanRemoval = true)
    @JoinColumn(name = "MEMBER_ID")
    private List<AddressEntity> addressHistory = new ArrayList<>();

    ...
}
```

<br>

# 정리

- 엔티티 타입의 특징
    - 식별자O 
    - 생명 주기 관리
    - 공유
- 값 타입의 특징
    - 식별자X 
    - 생명 주기를 엔티티에 의존
    - 공유하지 않는 것이 안전(복사해서 사용) 
    - 불변 객체로 만드는 것이 안전

<br>

# 결론

값 타입은 정말 값 타입이라 판단될 때만 사용 <br>
엔티티와 값 타입을 혼동해서 엔티티를 값 타입으로 만들면 안됨 <br>
식별자가 필요하고, 지속해서 값을 추적, 변경해야 한다면 그것은 값 타입이 아닌 엔티티

<br><br><br>
## References 및 사진 출처

> __김영한의 자바 ORM 표준 JPA 프로그래밍 - 기본편__