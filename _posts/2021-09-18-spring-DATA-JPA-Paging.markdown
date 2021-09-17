---
layout: post
title:  "spring Data JPA Paging"
subtitle:   "spring Data JPA Paging"
date:   2021-09-18 01:30:27 +0900
categories: spring
tags: spring JPA ORM Mapping String-Data-JPA Paging
comments: true
---


<br>

- 목차
	- [순수 JPA 페이징과 정렬](#순수-jpa-페이징과-정렬)
	- [스프링 데이터 JPA 페이징과 정렬](#스프링-데이터-jpa-페이징과-정렬)
	    - [Page 사용](#page-사용)
	    - [Slice 사용](#slice-사용)
	    - [List 사용](#list-사용)
	- [count 쿼리 문제점](#count-쿼리-문제점)
	    - [count 쿼리 분리](#count-쿼리-분리)
	- [페이지를 유지하면서 엔티티를 DTO로 변환하기](#페이지를-유지하면서-엔티티를-dto로-변환하기)
	- [정리](#정리)
    
<br>

# 순수 JPA 페이징과 정렬

JPA에서 페이징을 어떻게 할 것인가?

다음 조건으로 페이징과 정렬을 사용하는 예제 코드를 보자.

- 검색 조건: 나이가 10살
- 정렬 조건: 이름으로 내림차순
- 페이징 조건: 첫 번째 페이지, 페이지당 보여줄 데이터는 3건

```java
@Repository
@RequiredArgsConstructor
public class MemberJpaRepository {

    private final EntityManager em;
    
    public List<Member> findByPage(int age, int offset, int limit) {
        return em.createQuery("select m from Member m where m.age = :age order by m.username desc")
                .setParameter("age",age)
                .setFirstResult(offset)
                .setMaxResults(limit)
                .getResultList();
    }

    public long totalCount(int age) {
        return em.createQuery("select count(m) from Member m where m.age = :age", Long.class)
                .setParameter("age", age)
                .getSingleResult();
    }
}


@SpringBootTest
@Transactional
@Rollback(false)
class MemberJpaRepositoryTest {

    @Autowired
    MemberJpaRepository memberJpaRepository;

    @Test
    public void paging() {
        //given
        memberJpaRepository.save(new Member("member1", 10));
        memberJpaRepository.save(new Member("member2", 10));
        memberJpaRepository.save(new Member("member3", 10));
        memberJpaRepository.save(new Member("member4", 10));
        memberJpaRepository.save(new Member("member5", 10));

        int age = 10;
        int offset = 0;
        int limit = 3;

        //when
        List<Member> members = memberJpaRepository.findByPage(age, offset, limit);
        long totalCount = memberJpaRepository.totalCount(age);

        //then
        Assertions.assertThat(members.size()).isEqualTo(3);
        Assertions.assertThat(totalCount).isEqualTo(5);
    }
}
```

- 결과

```
// findByPage 쿼리
select
    member0_.member_id as member_i1_0_,
    member0_.age as age2_0_,
    member0_.team_id as team_id4_0_,
    member0_.username as username3_0_ 
from
    member member0_ 
where
    member0_.age=? 
order by
    member0_.username desc limit ?

// totalCount 쿼리
select
        count(member0_.member_id) as col_0_0_ 
    from
        member member0_ 
    where
        member0_.age=?
```

위와 같이 순수 JPA 페이징 코드를 작성할 수 있다.

<br><br>

# 스프링 데이터 JPA 페이징과 정렬

<br>

- 페이징과 정렬 파라미터
    - org.springframework.data.domain.Sort : 정렬 기능
    - org.springframework.data.domain.Pageable : 페이징 기능 (내부에 Sort 포함)

<br>

- 특별한 반환 타입
    - org.springframework.data.domain.Page : 추가 count 쿼리 결과를 포함하는 페이징
    - org.springframework.data.domain.Slice : 추가 count 쿼리 없이 다음 페이지만 확인 가능 (내부적으로 limit + 1조회)
    - List (자바 컬렉션): 추가 count 쿼리 없이 결과만 반환

<br>

- 페이징과 정렬 사용 예제

```java
public interface MemberRepository extends JpaRepository<Member, Long> {

    Page<Member> findByUsername(String name, Pageable pageable); //count 쿼리 사용
    Slice<Member> findByUsername(String name, Pageable pageable); //count 쿼리 사용
    안함
    List<Member> findByUsername(String name, Pageable pageable); //count 쿼리 사용
    안함
    List<Member> findByUsername(String name, Sort sort);
}
```

<br><br>

다음 조건으로 페이징과 정렬을 사용하는 예제 코드를 보자.

- 검색 조건: 나이가 10살
- 정렬 조건: 이름으로 내림차순
-  페이징 조건: 첫 번째 페이지, 페이지당 보여줄 데이터는 3건

<br>

## Page 사용

```java
public interface MemberRepository extends JpaRepository<Member, Long> {

    Page<Member> findByAge(int age, Pageable pageable);
}

@SpringBootTest
@Transactional
@Rollback(false)
public class MemberRepositoryTest {

    @Autowired
    MemberRepository memberRepository;

    @Test
    public void paging() {
        //given
        memberRepository.save(new Member("member1", 10));
        memberRepository.save(new Member("member2", 10));
        memberRepository.save(new Member("member3", 10));
        memberRepository.save(new Member("member4", 10));
        memberRepository.save(new Member("member5", 10));

        //when
        int age = 10;

        // 첫번째 파라미터 : 현재 페이지
        // 두번째 파라미터 : 조회할 데이터 수
        // 세번째 파라미터 : 소팅 방법
        // 네번째 파라미터 : 소팅할 컬럼
        PageRequest pageRequest = PageRequest.of(0, 3, Sort.by(Sort.Direction.DESC, "username"));

        Page<Member> page = memberRepository.findByAge(age, pageRequest);

        //then
        List<Member> content = page.getContent();
        long totalElements = page.getTotalElements();

        Assertions.assertThat(content.size()).isEqualTo(3); //조회된 데이터 수
        Assertions.assertThat(page.getTotalElements()).isEqualTo(5); //전체 데이터 수
        Assertions.assertThat(page.getNumber()).isEqualTo(0); //페이지 번호
        Assertions.assertThat(page.getTotalPages()).isEqualTo(2); //전체 페이지 번호
        Assertions.assertThat(page.isFirst()).isTrue(); //첫번째 항목인가?
        Assertions.assertThat(page.hasNext()).isTrue(); //다음 페이지가 있는가?
    }
}
```

- 결과

```
// content 가져오는 쿼리
select
    member0_.member_id as member_i1_0_,
    member0_.age as age2_0_,
    member0_.team_id as team_id4_0_,
    member0_.username as username3_0_ 
from
    member member0_ 
left outer join
    team team1_ 
        on member0_.team_id=team1_.team_id 
order by
    member0_.username desc limit ?

// count 가져오는 쿼리
select
    count(member0_.member_id) as col_0_0_ 
from
    member member0_
```

위와 같이 Page<Member>로 반환 받을 경우 content, count 값을 가져오는 쿼리가 나가게 된다. findByAge메소드에서 두 번째 파라미터로 받은 Pagable 은 인터페이스다. 따라서 실제 사용할 때는 해당 인터페이스를 구현한 org.springframework.data.domain.PageRequest 객체를 사용한다. PageRequest 생성자의 첫 번째 파라미터에는 현재 페이지를, 두 번째 파라미터에는 조회할 데이터 수를 입력한다. 여기에 추가로 정렬 정보도 파라미터로 사용할 수 있다. 참고로 페이지는 0부터 시작한다.

<br>

> 주의: Page는 1부터 시작이 아니라 0부터 시작이다.

<br>

## Slice 사용

```java
public interface MemberRepository extends JpaRepository<Member, Long> {

    Slice<Member> findByAge(int age, Pageable pageable);
}

@SpringBootTest
@Transactional
@Rollback(false)
public class MemberRepositoryTest {

    @Autowired
    MemberRepository memberRepository;

    @Test
    public void pagingSlice() {
        //given
        memberRepository.save(new Member("member1", 10));
        memberRepository.save(new Member("member2", 10));
        memberRepository.save(new Member("member3", 10));
        memberRepository.save(new Member("member4", 10));
        memberRepository.save(new Member("member5", 10));

        //when
        int age = 10;

        // 첫번째 파라미터 : 현재 페이지
        // 두번째 파라미터 : 조회할 데이터 수
        // 세번째 파라미터 : 소팅 방법
        // 네번째 파라미터 : 소팅할 컬럼
        PageRequest pageRequest = PageRequest.of(0, 3, Sort.by(Sort.Direction.DESC, "username"));

        Slice<Member> page = memberRepository.findByAge(age, pageRequest);

        //then
        List<Member> content = page.getContent();

        Assertions.assertThat(content.size()).isEqualTo(3); //조회된 데이터 수
        Assertions.assertThat(page.getNumber()).isEqualTo(0); //페이지 번호
        Assertions.assertThat(page.isFirst()).isTrue(); //첫번째 항목인가?
        Assertions.assertThat(page.hasNext()).isTrue(); //다음 페이지가 있는가?
    }
}
```

- 결과

```
select
    member0_.member_id as member_i1_0_,
    member0_.age as age2_0_,
    member0_.team_id as team_id4_0_,
    member0_.username as username3_0_ 
from
    member member0_ 
where
    member0_.age=? 
order by
    member0_.username desc limit 4
```

위 와같이 Slice는 조회할 데이터 수 +1 해서 가져오게 된다. PageRequest pageRequest = PageRequest.of(0, 3, Sort.by(Sort.Direction.DESC, "username")); 에서 조회할 데이터 수를 3으로 했지만 실제 쿼리에는 limit 4 로해서 실행된 것을 볼수 있다. 그리고 count 쿼리가 실행되지 않는다.

<br>

## List 사용


```java
public interface MemberRepository extends JpaRepository<Member, Long> {

    List<Member> findByAge(int age, Pageable pageable);
}

@SpringBootTest
@Transactional
@Rollback(false)
public class MemberRepositoryTest {

    @Autowired
    MemberRepository memberRepository;

    @Test
    public void pagingList() {
        //given
        memberRepository.save(new Member("member1", 10));
        memberRepository.save(new Member("member2", 10));
        memberRepository.save(new Member("member3", 10));
        memberRepository.save(new Member("member4", 10));
        memberRepository.save(new Member("member5", 10));

        //when
        int age = 10;

        // 첫번째 파라미터 : 현재 페이지
        // 두번째 파라미터 : 조회할 데이터 수
        // 세번째 파라미터 : 소팅 방법
        // 네번째 파라미터 : 소팅할 컬럼
        PageRequest pageRequest = PageRequest.of(0, 3, Sort.by(Sort.Direction.DESC, "username"));

        List<Member> page = memberRepository.findByAge(age, pageRequest);

        for (Member member : page) {
            System.out.println("member = " + member);
        }
    }
}
```

- 결과

```
select
    member0_.member_id as member_i1_0_,
    member0_.age as age2_0_,
    member0_.team_id as team_id4_0_,
    member0_.username as username3_0_ 
from
    member member0_ 
where
    member0_.age=? 
order by
    member0_.username desc limit ?
```

위와 같이 List로 반환받는 페이징을 위함이 아닌, 온전히 데이터만 조회하기 위함이다. 따로 count쿼리가 실행되지 않는다.

<br>

- Page 인터페이스

```java
public interface Page<T> extends Slice<T> {
    int getTotalPages(); //전체 페이지 수
    long getTotalElements(); //전체 데이터 수
    <U> Page<U> map(Function<? super T, ? extends U> converter); //변환기
}
```

- Slice 인터페이스

```java
public interface Slice<T> extends Streamable<T> {
    int getNumber(); //현재 페이지
    int getSize(); //페이지 크기
    int getNumberOfElements(); //현재 페이지에 나올 데이터 수
    List<T> getContent(); //조회된 데이터
    boolean hasContent(); //조회된 데이터 존재 여부
    Sort getSort(); //정렬 정보
    boolean isFirst(); //현재 페이지가 첫 페이지 인지 여부
    boolean isLast(); //현재 페이지가 마지막 페이지 인지 여부
    boolean hasNext(); //다음 페이지 여부
    boolean hasPrevious(); //이전 페이지 여부
    Pageable getPageable(); //페이지 요청 정보
    Pageable nextPageable(); //다음 페이지 객체
    Pageable previousPageable();//이전 페이지 객체
    <U> Slice<U> map(Function<? super T, ? extends U> converter); //변환기
}
```

<br>

# count 쿼리 문제점

Page를 사용할 때 count 쿼리를 다음과 같이 분리할 수 있다. 분리하는 이유는 기본으로 count 쿼리를 지원하는데 join등 최적화 되지않은 쿼리가 실행될수 있다.

```java
public interface MemberRepository extends JpaRepository<Member, Long> {

    @Query(value = "select m from Member m left join m.team t")
    Page<Member> findByAge(int age, Pageable pageable);
}

@SpringBootTest
@Transactional
@Rollback(false)
public class MemberRepositoryTest {

    @Autowired
    MemberRepository memberRepository;

    @Test
    public void paging() {
        //given
        memberRepository.save(new Member("member1", 10));
        memberRepository.save(new Member("member2", 10));
        memberRepository.save(new Member("member3", 10));
        memberRepository.save(new Member("member4", 10));
        memberRepository.save(new Member("member5", 10));

        //when
        int age = 10;

        // 첫번째 파라미터 : 현재 페이지
        // 두번째 파라미터 : 조회할 데이터 수
        // 세번째 파라미터 : 소팅 방법
        // 네번째 파라미터 : 소팅할 컬럼
        PageRequest pageRequest = PageRequest.of(0, 3, Sort.by(Sort.Direction.DESC, "username"));

        Page<Member> page = memberRepository.findByAge(age, pageRequest);

        // API 결과 반환시 Entity를 그대로 반환하면 안되기 때문에 Dto로 변경해서 반환한다.
        Page<MemberDto> toMap = page.map(member -> new MemberDto(member.getId(), member.getUsername(), null));

        //then
        List<Member> content = page.getContent();
        long totalElements = page.getTotalElements();

        Assertions.assertThat(content.size()).isEqualTo(3); //조회된 데이터 수
        Assertions.assertThat(page.getTotalElements()).isEqualTo(5); //전체 데이터 수
        Assertions.assertThat(page.getNumber()).isEqualTo(0); //페이지 번호
        Assertions.assertThat(page.getTotalPages()).isEqualTo(2); //전체 페이지 번호
        Assertions.assertThat(page.isFirst()).isTrue(); //첫번째 항목인가?
        Assertions.assertThat(page.hasNext()).isTrue(); //다음 페이지가 있는가?
    }
}
```

- 결과

```
// content 쿼리 실행 결과
select
    member0_.member_id as member_i1_0_,
    member0_.age as age2_0_,
    member0_.team_id as team_id4_0_,
    member0_.username as username3_0_ 
from
    member member0_ 
left outer join
    team team1_ 
        on member0_.team_id=team1_.team_id 
order by
    member0_.username desc limit ?

// count 쿼리 실행 결과
select
    count(member0_.member_id) as col_0_0_ 
from
    member member0_ 
left outer join
    team team1_ 
        on member0_.team_id=team1_.team_id
```

위 결과와 같이 count 쿼리 실행 결과를 보면 content 쿼리 실행 결과에서 사용한 left join이 그대로 사용되고 있다. 이렇게 되면 데이터가 많아졌을때 성능상 문제가 생길수 있다.

<br>

## count 쿼리 분리

```java
public interface MemberRepository extends JpaRepository<Member, Long> {

    @Query(value = "select m from Member m left join m.team t",
            countQuery = "select count(m) from Member m")
    Page<Member> findByAge(int age, Pageable pageable);
}

@SpringBootTest
@Transactional
@Rollback(false)
public class MemberRepositoryTest {

    @Autowired
    MemberRepository memberRepository;

    @Test
    public void paging() {
        //given
        memberRepository.save(new Member("member1", 10));
        memberRepository.save(new Member("member2", 10));
        memberRepository.save(new Member("member3", 10));
        memberRepository.save(new Member("member4", 10));
        memberRepository.save(new Member("member5", 10));

        //when
        int age = 10;

        // 첫번째 파라미터 : 현재 페이지
        // 두번째 파라미터 : 조회할 데이터 수
        // 세번째 파라미터 : 소팅 방법
        // 네번째 파라미터 : 소팅할 컬럼
        PageRequest pageRequest = PageRequest.of(0, 3, Sort.by(Sort.Direction.DESC, "username"));

        Page<Member> page = memberRepository.findByAge(age, pageRequest);

        //then
        List<Member> content = page.getContent();
        long totalElements = page.getTotalElements();

        Assertions.assertThat(content.size()).isEqualTo(3); //조회된 데이터 수
        Assertions.assertThat(page.getTotalElements()).isEqualTo(5); //전체 데이터 수
        Assertions.assertThat(page.getNumber()).isEqualTo(0); //페이지 번호
        Assertions.assertThat(page.getTotalPages()).isEqualTo(2); //전체 페이지 번호
        Assertions.assertThat(page.isFirst()).isTrue(); //첫번째 항목인가?
        Assertions.assertThat(page.hasNext()).isTrue(); //다음 페이지가 있는가?
    }
}
```

- 결과

```
// content 쿼리 실행 결과
select
    member0_.member_id as member_i1_0_,
    member0_.age as age2_0_,
    member0_.team_id as team_id4_0_,
    member0_.username as username3_0_ 
from
    member member0_ 
left outer join
    team team1_ 
        on member0_.team_id=team1_.team_id 
order by
    member0_.username desc limit ?

// count 쿼리 실행 결과
select
    count(member0_.member_id) as col_0_0_ 
from
    member member0_
```

@Query 어노테이션에 countQuery를 설정할 수 있다. 위 결과와 같이 findByAge 메소드에 설정된 countQuery가 실행 되면서, count 쿼리 문제점을 해결 할 수 있다.

<br>

# 페이지를 유지하면서 엔티티를 DTO로 변환하기

```java
public interface MemberRepository extends JpaRepository<Member, Long> {

    @Query(value = "select m from Member m left join m.team t",
            countQuery = "select count(m) from Member m")
    Page<Member> findByAge(int age, Pageable pageable);
}

@RestController
@RequiredArgsConstructor
public class HelloController {

    private final MemberRepository memberRepository;

    @GetMapping("/api/mapping")
    public Page<MemberDto> mapping() {
        memberRepository.save(new Member("member1", 10));
        memberRepository.save(new Member("member2", 10));
        memberRepository.save(new Member("member3", 10));
        memberRepository.save(new Member("member4", 10));
        memberRepository.save(new Member("member5", 10));

        //when
        int age = 10;

        // 첫번째 파라미터 : 현재 페이지
        // 두번째 파라미터 : 조회할 데이터 수
        // 세번째 파라미터 : 소팅 방법
        // 네번째 파라미터 : 소팅할 컬럼
        PageRequest pageRequest = PageRequest.of(0, 3, Sort.by(Sort.Direction.DESC, "username"));

        Page<Member> page = memberRepository.findByAge(age, pageRequest);

        // API 결과 반환시 Entity를 그대로 반환하면 안되기 때문에 Dto로 변경해서 반환한다.
        return page.map(member -> new MemberDto(member.getId(), member.getUsername(), null));
    }
}
```

- 결과

```json
{
    "content": [
        {
            "id": 5,
            "username": "member5",
            "teamName": null
        },
        {
            "id": 4,
            "username": "member4",
            "teamName": null
        },
        {
            "id": 3,
            "username": "member3",
            "teamName": null
        }
    ],
    "pageable": {
        "sort": {
            "sorted": true,
            "unsorted": false,
            "empty": false
        },
        "offset": 0,
        "pageSize": 3,
        "pageNumber": 0,
        "unpaged": false,
        "paged": true
    },
    "last": false,
    "totalPages": 2,
    "totalElements": 5,
    "size": 3,
    "number": 0,
    "sort": {
        "sorted": true,
        "unsorted": false,
        "empty": false
    },
    "first": true,
    "numberOfElements": 3,
    "empty": false
}
```

위와 같이 API 에서 데이터 반환시 map 을 통해 Dto로 변환해서 반환하는 것을 추천한다.

<br><br>

# 정리

- Page (count 쿼리 O)
- Slice (count 쿼리 X) 추가로 limit + 1을 조회한다. 그래서 다음 페이지 여부 확인(최근 모바일 리스트 생각해보면 됨)
- List (count 쿼리 X)
- 카운트 쿼리 분리(이건 복잡한 sql에서 사용, 데이터는 left join, 카운트는 left join 안해도 됨)
    - 실무에서 매우 중요!!!

> 참고: 전체 count 쿼리는 매우 무겁다.

<br><br><br>
## References 및 사진 출처

> __김영한의 실전! 스프링 데이터 JPA__