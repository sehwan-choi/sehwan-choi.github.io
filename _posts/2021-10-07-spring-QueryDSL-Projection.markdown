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

<br><br>

## @QueryProjection

이 방법은 컴파일러로 타입을 체크할 수 있으므로 가장 안전한 방법이다. 다만 DTO에 QueryDSL 어노테이션을 유지해야 하는 점과 DTO까지 Q 파일을 생성해야 하는 단점이 있다. <br>

- Q 파일 생성

1. 사용할 생성자에 @QueryProjection 어노테이션을 추가한다.

```java
@Getter
@Setter
@ToString(of = {"username","age"})
public class MemberDto {

    private String username;
    private int age;

    public MemberDto() {
    }

    @QueryProjection    // @QueryProjection 어노테이션을 추가한다.
    public MemberDto(String username, int age) {
        this.username = username;
        this.age = age;
    }
}
```

2. 아래의 사진과 같이 우측 Gradle -> querydsl -> Tasks -> other -> compileQuerydsl 실행

![그림1](https://sehwan-choi.github.io/assets/img/spring/QueryDSL/jpa4.jpg)

3. 아래의 사진과 같이 build -> generated -> querydsl -> study.querydsl -> dto -> QMemberDto 생성되었는지 확인. <br>

참고로 querydsl -> study.querydsl 이부분은 각각 프로젝트마다 Group, Artifact명이 다르기 때문에 각자 생성된 이름을 따라가면 된다.

![그림1](https://sehwan-choi.github.io/assets/img/spring/QueryDSL/jpa5.jpg)

<br><br>

- @QueryProjection 사용 예제

```java
@Getter
@Setter
@ToString(of = {"username","age"})
public class MemberDto {

    private String username;
    private int age;

    public MemberDto() {
    }

    @QueryProjection
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
    public void findDtoByQueryProjection() {
        List<MemberDto> result = queryFactory
                .select(new QMemberDto(member.username, member.age))
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

<br><br>

## @QueryProjection 방법과 Setter, 필드, 생성자 접근방법의 차이점

```
import com.querydsl.core.annotations.QueryProjection;
```
@QueryProjection 은 위와 같이 querydsl의 라이브러리를 사용하고 있다. DTO는 대부분 Repository 뿐만 아니라 Service, API까지 넘겨서 사용하기 때문에 QueryDsl에 의존적이게 된다. 하지만 위의 @QueryProjection 설명과 같이 이 방법은 컴파일러로 타입을 체크할 수 있으므로 가장 안전한 방법이다. (컴파일 레벨에서 에러를 발견할 수 있다) <br>

예 )
```java
// 생성자 접근 방식
List<MemberDto> result = queryFactory
        .select(Projections.constructor(MemberDto.class,
                member.username, member.age))
        .from(member)
        .fetch();

// @QueryProjection 방식
List<MemberDto> result = queryFactory
        .select(new QMemberDto(member.username, member.age))
        .from(member)
        .fetch();
```
위 두개의 차이점은 생성자 접근 방식은 Projections.constructor에서 인자로 MemberDto의 클래스 타입과, 선언된 생성자에 맞게 인자를 넘기게 되어있는데, 실수로 member.id 라는 값을 하나더 넘기게 된다면 컴파일 시점에서는 문제가 없지만 실제 동작과정에서 에러가 발생할 것이다. <br>
반대로 @QueryProjection 방식은 QmemberDto를 생성하면서 인자를 직접 체크하기 때문에 컴파일 시점에서 에러를 잡을수 있다는 엄청난 장점이 있다.

<br><br><br>
## References 및 사진 출처

> __김영한의 실전! Querydsl__