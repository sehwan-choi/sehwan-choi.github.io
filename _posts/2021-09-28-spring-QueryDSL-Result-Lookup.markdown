---
layout: post
title:  "spring Querydsl 기본문법 - 결과 조회"
subtitle:   "spring Querydsl 기본문법 - 결과 조회"
date:   2021-09-28 02:46:27 +0900
categories: spring
tags: spring JPA ORM Mapping QueryDSL ResultLookup
comments: true
---


<br>

- 목차
	- [fetch](#fetch)
	- [fetchOne](#fetchone)
	- [fetchFirst](#fetchfirst)
	- [fetchResults](#fetchresults)
	- [fetchCount](#fetchcount)
    
<br>

# fetch

fetch() : 리스트 조회, 데이터 없으면 빈 리스트 반환

```java
@Test
public void fetch() {

	// 멤버를 모두 조회
	List<Member> fetch = queryFactory
			.selectFrom(member)
			.fetch();
```

- 결과

```SQL
JPQL :
select
	member1 
from
	Member member1
		
SQL :
select
	member0_.member_id as member_i1_0_,
	member0_.age as age2_0_,
	member0_.team_id as team_id4_0_,
	member0_.username as username3_0_ 
from
	member member0_
```

<br><br>

# fetchOne

fetchOne() : 단 건 데이터 조회한다. <br>
결과가 없으면 : null 반환 <br>
결과가 둘 이상이면 : com.querydsl.core.NonUniqueResultException 예외 반환한다. <br>

```java
@Test
public void fetchOne() {

	// 단건 조회
	Member fetchOne = queryFactory
		.selectFrom(member)
		.fetchOne();
}
```

- 결과

```SQL
JPQL :
select
	member1 
from
	Member member1 
where
	member1.username = 'member1'

SQL :
select
		member0_.member_id as member_i1_0_,
		member0_.age as age2_0_,
		member0_.team_id as team_id4_0_,
		member0_.username as username3_0_ 
	from
		member member0_ 
	where
		member0_.username='member1'
```

<br><br>

# fetchFirst

fetchFirst() : 처음 한 건의 데이터를 조회한다. <br>
fetchFirst()는 limit(1).fetchOne() 과 같으며 실제로 내부에 소스를 보면 아래와 같이 선언 되어있다. <br>

```java
public abstract class FetchableQueryBase<T, Q extends FetchableQueryBase<T, Q>>
        extends QueryBase<Q> implements Fetchable<T> {
	
	...
    @Override
    public final T fetchFirst() {
        return limit(1).fetchOne();
    }
	...
}
```

- 예제

```java
@Test
public void fetchFirst() {

	// 처음 한 건 조회
	Member fetchFirst = queryFactory
			.selectFrom(QMember.member)
			//.limit(1).fetch() //  fetchFirst()와 같다
			.fetchFirst();
}
```

- 결과

```SQL
JPQL :
select
	member1 
from
	Member member1

SQL :
select
	member0_.member_id as member_i1_0_,
	member0_.age as age2_0_,
	member0_.team_id as team_id4_0_,
	member0_.username as username3_0_ 
from
	member member0_ limit 1
```

<br><br>

# fetchResults

fetchResults() : 페이징 정보, 내부 데이터, total count 를 조회한다. <br>
내부 데이터(content)를 가져오는 쿼리실행후 total count 쿼리 추가 실행되어 총 2번의 쿼리가 실행된다.

```java
@Test
public void fetchResults() {

	// 페이징에서 사용
	QueryResults<Member> result = queryFactory
			.selectFrom(member)
			.fetchResults();

	long total = result.getTotal(); //  총 개수
	System.out.println("total = " + total);

	List<Member> content = result.getResults(); //  실제 데이터
	for (Member member : content) {
		System.out.println("member = " + member);
	}

}
```

- 결과

```SQL
JPQL : 
select
        count(member1) 
    from
        Member member1

SQL :
select
	count(member0_.member_id) as col_0_0_ 
from
	member member0_

total = 4
member = Member(id=3, username=member1, age=10)
member = Member(id=4, username=member2, age=10)
member = Member(id=5, username=member3, age=10)
member = Member(id=6, username=member4, age=10)
```

위 결과와 같이 count 쿼리후, 내부 데이터를 가져오는 쿼리가 실행되어 총 2번의 쿼리가 실행된다. <br>
result.getTotal(); 로 총 개수를, <br>
List<Member> content = result.getResults(); 로 실제 내부데이터를 핸들링 할 수 있다.<br>

<br><br>

# fetchCount

fetchCount() : count 쿼리로 변경해서 count를 조회한다.

```java
@Test
public void fetchCount() {
	// count 조회
	long count = queryFactory
			.selectFrom(member)
			.fetchCount();
	System.out.println("count = " + count);
}
```

- 결과

```SQL
JPQL :
select
	count(member1) 
from
	Member member1

SQL :		
select
	count(member0_.member_id) as col_0_0_ 
from
	member member0_

count = 4
```

위 결과와 같이 count의 개수만 가져온다.

<br><br><br>
## References 및 사진 출처

> __김영한의 실전! Querydsl__