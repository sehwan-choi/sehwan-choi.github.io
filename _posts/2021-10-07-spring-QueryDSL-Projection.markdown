---
layout: post
title:  "spring Querydsl 프로젝션과 결과 반환 - DTO 조회"
subtitle:   "spring Querydsl 프로젝션과 결과 반환 - DTO 조회"
date:   2021-10-07 02:35:27 +0900
categories: spring
tags: spring JPA ORM Mapping QueryDSL Projection
comments: true
---


<br>

- 목차
	- [순수 JPA에서 DTO 조회](#순수-jpa에서-dto-조회)
	- [QueryDSL에서 DTO조회](#querydsl에서-dto조회)
	    - [프로퍼티 접근 - Setter](#프로퍼티-접근---setter)
	    - [필드 직접 접근](#필드-직접-접근)
	    - [생성자 접근](#생성자-접근)
	    - [별칭이 다를 때](#별칭이-다를-때)
	
<br>

# 순수 JPA에서 DTO 조회

```java
@Getter
@Setter
@ToString(of = {"username","age"})
public class MemberDto {

    private String username;
    private int age;

    public MemberDto() {
    }

    public MemberDto(String username, int age) {
        this.username = username;
        this.age = age;
    }
}

@SpringBootTest
@Transactional
public class QuerydslBasicTest {
    
    JPAQueryFactory queryFactory;

    @Test
    public void findDtoByJPQL() {
        List<MemberDto> result = em.createQuery("select new study.querydsl.dto.MemberDto(m.username, m.age) from Member m", MemberDto.class)
                .getResultList();

        for (MemberDto memberDto : result) {
            System.out.println("memberDto = " + memberDto);
        }
    }
}
```

- 결과

```sql
JPQL :
select
        member1.username,
        member1.age 
    from
        Member member1 

SQL :        
select
    member0_.username as col_0_0_,
    member0_.age as col_1_0_ 
from
    member member0_

memberDto = MemberDto(username=member1, age=10)
memberDto = MemberDto(username=member2, age=20)
memberDto = MemberDto(username=member3, age=30)
memberDto = MemberDto(username=member4, age=40)
```

위와같이 순수 JPA에서 DTO를 조회할 때는 new 명령어를 사용해야한다.
DTO의 package이름을 다 적어줘야해서 지저분하며 생성자 방식만 지원한다.

<br><br>

# QueryDSL에서 DTO조회

<br><br>

## 프로퍼티 접근 - Setter

```java
@Getter
@Setter
@ToString(of = {"username","age"})
public class MemberDto {

    private String username;
    private int age;

    public MemberDto() {
    }

    public MemberDto(String username, int age) {
        this.username = username;
        this.age = age;
    }
}

@SpringBootTest
@Transactional
public class QuerydslBasicTest {
    
    JPAQueryFactory queryFactory;

    @Test
    public void findDtoBySetter() {
        List<MemberDto> result = queryFactory
                .select(Projections.bean(MemberDto.class,
                        member.username, member.age))
                .from(member)
                .fetch();

        for (MemberDto memberDto : result) {
            System.out.println("memberDto = " + memberDto);
        }
    }
}
```

- 결과

```sql
select
        member1.username,
        member1.age 
    from
        Member member1 */ select
            member0_.username as col_0_0_,
            member0_.age as col_1_0_ 
        from
            member member0_
```

위 코드와 같이 Projections.bean을 사용하게 되면 MemberDto의 필드값을 setter로 접근하여 값을 채워넣게 된다.

> 주의! <br>
MemberDto에 Setter가 없다면 어떻게 될까? <br>
모든 필드가 아래와 같이 null혹은 0으로 초기화 된다.<br>
반드시 Setter를 만들고 사용해야한다.

```java
memberDto = MemberDto(username=null, age=0)
memberDto = MemberDto(username=null, age=0)
memberDto = MemberDto(username=null, age=0)
memberDto = MemberDto(username=null, age=0)
```

<br><br>

## 필드 직접 접근

```java
@Getter
@Setter
@ToString(of = {"username","age"})
public class MemberDto {

    private String username;
    private int age;

    public MemberDto() {
    }

    public MemberDto(String username, int age) {
        this.username = username;
        this.age = age;
    }
}

@SpringBootTest
@Transactional
public class QuerydslBasicTest {
    
    JPAQueryFactory queryFactory;

    @Test
    public void findDtoByFeild() {
        List<MemberDto> result = queryFactory
                .select(Projections.fields(MemberDto.class,
                        member.username, member.age))
                .from(member)
                .fetch();

        for (MemberDto memberDto : result) {
            System.out.println("username = " + memberDto.getUsername());
            System.out.println("age = " + memberDto.getAge());
        }
    }
}
```

- 결과

```sql
JPQL :
select
    member1.username,
    member1.age 
from
    Member member1 

SQL :    
select
    member0_.username as col_0_0_,
    member0_.age as col_1_0_ 
from
    member member0_

username = member1
age = 10
username = member2
age = 20
username = member3
age = 30
username = member4
age = 40
```

위 코드와 같이 Projections.fields 사용하게 되면 MemberDto의 필드에 직접 접근하여 값을 채워넣게 된다.

> 참고! <br>
MemberDto에 Setter가 없다면 어떻게 될까? <br>
필드 직접 접근은 Setter가 없어도 필드에 직접 접근하기 때문에 상관없이 값이 채워진다.



<br><br>

## 생성자 접근

```java
@Getter
@Setter
@ToString(of = {"username","age"})
public class MemberDto {

    private String username;
    private int age;

    public MemberDto() {
    }

    public MemberDto(String username, int age) {
        this.username = username;
        this.age = age;
    }
}

@SpringBootTest
@Transactional
public class QuerydslBasicTest {
    
    JPAQueryFactory queryFactory;

    @Test
    public void findDtoByContructor() {
        List<MemberDto> result = queryFactory
                .select(Projections.constructor(MemberDto.class,
                        member.username, member.age))
                .from(member)
                .fetch();

        for (MemberDto memberDto : result) {
            System.out.println("username = " + memberDto.getUsername());
            System.out.println("age = " + memberDto.getAge());
        }
    }
}
```

- 결과

```sql
JPQL :
select
    member1.username,
    member1.age 
from
    Member member1 

SQL :
select
    member0_.username as col_0_0_,
    member0_.age as col_1_0_ 
from
    member member0_

username = member1
age = 10
username = member2
age = 20
username = member3
age = 30
username = member4
age = 40
```

위와같이 Projections.constructor를 사용하여 MemberDto의 생성자로 값을 채워넣을수 있다. 반드시 MemberDto에 Projections.constructor안에 들어가는 타입 String(member.username), int(member.age) 타입의 생성자가 있어야한다.

> 주의! <br>
만일 MemberDto의 생성자가 없다면 아래와 같은 예외가 발생한다.

```
No constructor found for class study.querydsl.dto.MemberDto with parameters: [class java.lang.String, class java.lang.Integer]
```


<br><br>

## 별칭이 다를 때

만일 프로퍼티접근(setter), 필드 직접 접근, 생성자 접근시 Dto의 필드 명과 Entity의 필드명이 맞지않는 경우가 발생할때 별칭을 사용할 수 있다. <br>
ExpressionUtils.as(source,alias) : 필드나, 서브 쿼리에 별칭 적용한다. <br>
username.as("memberName") : 필드에 별칭 적용 한다.<br>


```java
@Getter
@Setter
@ToString(of = {"username","age"})
public class MemberDto {

    private String name;
    private int age;

    public MemberDto() {
    }

    public MemberDto(String username, int age) {
        this.username = username;
        this.age = age;
    }
}

@SpringBootTest
@Transactional
public class QuerydslBasicTest {
    
    JPAQueryFactory queryFactory;

    @Test
    public void findDtoByFeildAlias() {

        QMember memberSub = new QMember("memberSub");

        List<UserDto> fetch = queryFactory
                .select(Projections.fields(UserDto.class,
                                member.username.as("name"),
                                ExpressionUtils.as(
                                        JPAExpressions
                                                .select(memberSub.age.max())
                                                .from(memberSub), "age")
                        )
                ).from(member)
                .fetch();

        for (UserDto userDto : fetch) {
            System.out.println("userDto = " + userDto);
        }
    }
}
```

- 결과

```sql
JPQL :
select
    member1.username as name,
    (select
        max(memberSub.age) 
    from
        Member memberSub) as age 
from
    Member member1

SQL :
select
    member0_.username as col_0_0_,
    (select
        max(member1_.age) 
    from
        member member1_) as col_1_0_ 
from
    member member0_

userDto = UserDto(name=member1, age=40)
userDto = UserDto(name=member2, age=40)
userDto = UserDto(name=member3, age=40)
userDto = UserDto(name=member4, age=40)
```


> 주의! <br>
만일 필드명이 맞지 않는경우 Dto의 값은 null 혹은 0으로 값이 채워진다.

<br><br><br>
## References 및 사진 출처

> __김영한의 실전! Querydsl__