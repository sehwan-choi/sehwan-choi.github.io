---
layout: post
title:  "spring Data JPA Paging and Sort"
subtitle:   "spring Data JPA Paging and Sort"
date:   2021-09-22 00:29:27 +0900
categories: spring
tags: spring JPA ORM Mapping String-Data-JPA Paging Sort
comments: true
---


<br>

- 목차
	- [페이징과 정렬](#페이징과-정렬)
	    - [요청 파라미터](#요청-파라미터)
	- [글로벌 기본값 설정](#글로벌-기본값-설정)
	- [개별 기본값 설정](#개별-기본값-설정)
	- [페이징 정보가 둘 이상인 경우](#페이징-정보가-둘-이상인-경우)
	- [Page 내용을 DTO로 변환하기](#page-내용을-dto로-변환하기)
	- [Page를 1부터 시작하기](#page를-1부터-시작하기)
    
<br>

# 페이징과 정렬

스프링 데이터가 제공하는 페이징과 정렬 기능을 스프링 MVC에서 편리하게 사용할 수 있다.

<br><br>

```java
@RestController
@RequiredArgsConstructor
public class MemberController {

    private final MemberRepository memberRepository;
    }

    @GetMapping("/members")
    public Page<Member> list(Pageable pageable) {
        return memberRepository.findAll(pageable);
    }

    @PostConstruct
    public void init() {
        for (int i = 0 ; i < 100 ; i ++) {
            Member member = new Member("user"+i,i);
            memberRepository.save(member);
        }
    }
}
```

- 요청 파라미터에 size, page 사용

```
GET 127.0.0.1:8080/members?page=0&size=10

{
    "content": [
        {
            "createdDate": "2021-09-22T00:33:07.306276",
            "lastModifiedDate": "2021-09-22T00:33:07.306276",
            "createdBy": "33b42005-2ccf-4ead-bcf8-478871c0939d",
            "lastModifiedBy": "33b42005-2ccf-4ead-bcf8-478871c0939d",
            "id": 1,
            "username": "user0",
            "age": 0,
            "team": null
        },
        {
            "createdDate": "2021-09-22T00:33:07.352319",
            "lastModifiedDate": "2021-09-22T00:33:07.352319",
            "createdBy": "d6d7fe74-f386-40cd-85c9-53951cf89c4b",
            "lastModifiedBy": "d6d7fe74-f386-40cd-85c9-53951cf89c4b",
            "id": 2,
            "username": "user1",
            "age": 1,
            "team": null
        },
        {
            "createdDate": "2021-09-22T00:33:07.355321",
            "lastModifiedDate": "2021-09-22T00:33:07.355321",
            "createdBy": "09ec8125-778e-47b3-a102-63d674c60beb",
            "lastModifiedBy": "09ec8125-778e-47b3-a102-63d674c60beb",
            "id": 3,
            "username": "user2",
            "age": 2,
            "team": null
        }
    ],
    "pageable": {
        "sort": {
            "sorted": false,
            "unsorted": true,
            "empty": true
        },
        "offset": 0,
        "pageSize": 3,
        "pageNumber": 0,
        "unpaged": false,
        "paged": true
    },
    "last": false,
    "totalElements": 100,
    "totalPages": 34,
    "size": 3,
    "number": 0,
    "sort": {
        "sorted": false,
        "unsorted": true,
        "empty": true
    },
    "first": true,
    "numberOfElements": 3,
    "empty": false
}
```

```
GET 127.0.0.1:8080/members?page=2&size=3

{
    "content": [
        {
            "createdDate": "2021-09-22T00:33:07.367333",
            "lastModifiedDate": "2021-09-22T00:33:07.367333",
            "createdBy": "a156c584-5ed0-4fe9-a01f-141d8423287d",
            "lastModifiedBy": "a156c584-5ed0-4fe9-a01f-141d8423287d",
            "id": 7,
            "username": "user6",
            "age": 6,
            "team": null
        },
        {
            "createdDate": "2021-09-22T00:33:07.369334",
            "lastModifiedDate": "2021-09-22T00:33:07.369334",
            "createdBy": "fbd4cb80-c310-4648-9ccd-3e9b4014d217",
            "lastModifiedBy": "fbd4cb80-c310-4648-9ccd-3e9b4014d217",
            "id": 8,
            "username": "user7",
            "age": 7,
            "team": null
        },
        {
            "createdDate": "2021-09-22T00:33:07.373338",
            "lastModifiedDate": "2021-09-22T00:33:07.373338",
            "createdBy": "01ad5c56-63ae-4ff7-9c9f-6896da15891d",
            "lastModifiedBy": "01ad5c56-63ae-4ff7-9c9f-6896da15891d",
            "id": 9,
            "username": "user8",
            "age": 8,
            "team": null
        }
    ],
    "pageable": {
        "sort": {
            "sorted": false,
            "unsorted": true,
            "empty": true
        },
        "offset": 6,
        "pageSize": 3,
        "pageNumber": 2,
        "unpaged": false,
        "paged": true
    },
    "last": false,
    "totalElements": 100,
    "totalPages": 34,
    "size": 3,
    "number": 2,
    "sort": {
        "sorted": false,
        "unsorted": true,
        "empty": true
    },
    "first": false,
    "numberOfElements": 3,
    "empty": false
}
```

- 요청 파라미터에 sort 사용

```
127.0.0.1:8080/members?page=2&size=3&sort=username,desc

{
    "content": [
        {
            "createdDate": "2021-09-22T00:33:07.599547",
            "lastModifiedDate": "2021-09-22T00:33:07.599547",
            "createdBy": "cd03d55c-fa84-48ed-8f3b-868a5004c50d",
            "lastModifiedBy": "cd03d55c-fa84-48ed-8f3b-868a5004c50d",
            "id": 94,
            "username": "user93",
            "age": 93,
            "team": null
        },
        {
            "createdDate": "2021-09-22T00:33:07.59554",
            "lastModifiedDate": "2021-09-22T00:33:07.59554",
            "createdBy": "e18fbb9d-d05a-42ef-af53-3cb71619551e",
            "lastModifiedBy": "e18fbb9d-d05a-42ef-af53-3cb71619551e",
            "id": 93,
            "username": "user92",
            "age": 92,
            "team": null
        },
        {
            "createdDate": "2021-09-22T00:33:07.592538",
            "lastModifiedDate": "2021-09-22T00:33:07.592538",
            "createdBy": "924b8f44-5eb3-4afe-8620-afb87ab6ef8f",
            "lastModifiedBy": "924b8f44-5eb3-4afe-8620-afb87ab6ef8f",
            "id": 92,
            "username": "user91",
            "age": 91,
            "team": null
        }
    ],
    "pageable": {
        "sort": {
            "sorted": true,
            "unsorted": false,
            "empty": false
        },
        "offset": 6,
        "pageSize": 3,
        "pageNumber": 2,
        "unpaged": false,
        "paged": true
    },
    "last": false,
    "totalElements": 100,
    "totalPages": 34,
    "size": 3,
    "number": 2,
    "sort": {
        "sorted": true,
        "unsorted": false,
        "empty": false
    },
    "first": false,
    "numberOfElements": 3,
    "empty": false
}
```

- 파라미터로 Pageable 을 받을 수 있다. 
- Pageable 은 인터페이스, 실제는 org.springframework.data.domain.PageRequest 객체 생성한다.

## 요청 파라미터
- 예) /members?page=0&size=3&sort=id,desc&sort=username,desc
- page: 현재 페이지, 0부터 시작한다.
- size: 한 페이지에 노출할 데이터 건수
- sort: 정렬 조건을 정의한다.
    - 예) 정렬 속성,정렬 속성...(ASC | DESC), 정렬 방향을 변경하고 싶으면 sort 파라미터 추가 (asc 생략 가능)

<br><br>

# 글로벌 기본값 설정

application.yml

```
...
spring:
  data:
    web:
      pageable:
        default-page-size: 10   /# 기본 페이지 사이즈/
        max-page-size: 100      /# 최대 페이지 사이즈/
...
```

위와 같이 기본 페이지 사이즈와 최대 페이지 사이즈를 글로벌로 설정할 수 있다.

<br><br>

# 개별 기본값 설정

@PageableDefault 어노테이션을 사용

```java
@RestController
@RequiredArgsConstructor
public class MemberController {

    private final MemberRepository memberRepository;

    @GetMapping("/members")
    public Page<Member> list(@PageableDefault(size = 5, sort = "username", direction = Sort.Direction.DESC) Pageable pageable) {
        return memberRepository.findAll(pageable);
    }

    @PostConstruct
    public void init() {
        for (int i = 0 ; i < 100 ; i ++) {
            Member member = new Member("user"+i,i);
            memberRepository.save(member);
        }
    }
}
```

- 결과

```
GET 127.0.0.1:8080/members

{
    "content": [
        {
            "createdDate": "2021-09-22T00:53:45.670441",
            "lastModifiedDate": "2021-09-22T00:53:45.670441",
            "createdBy": "72392d41-b320-4d3d-adb3-673a3323ddb2",
            "lastModifiedBy": "72392d41-b320-4d3d-adb3-673a3323ddb2",
            "id": 100,
            "username": "user99",
            "age": 99,
            "team": null
        },
        {
            "createdDate": "2021-09-22T00:53:45.66844",
            "lastModifiedDate": "2021-09-22T00:53:45.66844",
            "createdBy": "1dae39ff-0b85-486d-a51e-6368cf294c92",
            "lastModifiedBy": "1dae39ff-0b85-486d-a51e-6368cf294c92",
            "id": 99,
            "username": "user98",
            "age": 98,
            "team": null
        },
        {
            "createdDate": "2021-09-22T00:53:45.666438",
            "lastModifiedDate": "2021-09-22T00:53:45.666438",
            "createdBy": "9a51c3c1-6f6c-463b-b7fd-9a3fcaee701f",
            "lastModifiedBy": "9a51c3c1-6f6c-463b-b7fd-9a3fcaee701f",
            "id": 98,
            "username": "user97",
            "age": 97,
            "team": null
        },
        {
            "createdDate": "2021-09-22T00:53:45.663436",
            "lastModifiedDate": "2021-09-22T00:53:45.663436",
            "createdBy": "ab8785fe-a4f4-4799-b20e-b646bd0a72d3",
            "lastModifiedBy": "ab8785fe-a4f4-4799-b20e-b646bd0a72d3",
            "id": 97,
            "username": "user96",
            "age": 96,
            "team": null
        },
        {
            "createdDate": "2021-09-22T00:53:45.660432",
            "lastModifiedDate": "2021-09-22T00:53:45.660432",
            "createdBy": "7345541f-aabf-4333-9efe-030cf133f496",
            "lastModifiedBy": "7345541f-aabf-4333-9efe-030cf133f496",
            "id": 96,
            "username": "user95",
            "age": 95,
            "team": null
        }
    ],
    "pageable": {
        "sort": {
            "sorted": true,
            "unsorted": false,
            "empty": false
        },
        "offset": 0,
        "pageNumber": 0,
        "pageSize": 5,
        "paged": true,
        "unpaged": false
    },
    "last": false,
    "totalElements": 100,
    "totalPages": 20,
    "size": 5,
    "number": 0,
    "sort": {
        "sorted": true,
        "unsorted": false,
        "empty": false
    },
    "first": true,
    "numberOfElements": 5,
    "empty": false
}
```

위 결과와 같이 요청파라미터 없이 보내면 기본값(size = 5, sort = "username", direction = Sort.Direction.DESC)으로 반환한다.

<br><br>

# 페이징 정보가 둘 이상인 경우

- 페이징 정보가 둘 이상이면 접두사로 구분한다
- @Qualifier 에 접두사명 추가 "{접두사명}_xxx”
- 예제: /members?member_page=0&order_page=1

```java
@RestController
@RequiredArgsConstructor
public class MemberController {

    private final MemberRepository memberRepository;
    private final TeamRepository teamRepository;

    @GetMapping("/members3")
    public Result list3(
            @Qualifier("member") Pageable memberPageable,
            @Qualifier("team") Pageable teamPageable) {
        Page<Member> memberPage = memberRepository.findAll(memberPageable);
        Page<Team> teamPage = teamRepository.findAll(teamPageable);

        return new Result(memberPage,teamPage);
    }

    @PostConstruct
    public void init() {
        for (int i = 0 ; i < 100 ; i ++) {
            Member member = new Member("user"+i,i);
            memberRepository.save(member);
        }

        for (int j = 0 ; j < 100 ; j ++) {
            Team team = new Team("team" + j);
            teamRepository.save(team);
        }
    }

    @Data
    static class Result {
        private Page<Member> memberPage;
        private Page<Team> teamPage;

        public Result(Page<Member> memberPage, Page<Team> teamPage) {
            this.memberPage = memberPage;
            this.teamPage = teamPage;
        }
    }
}
```

- 결과

```
GET 127.0.0.1:8080/members3?member_page=3&member_size=3&team_page=0&team_size=5

{
    "memberPage": {
        "content": [
            {
                "createdDate": "2021-09-22T01:16:11.181422",
                "lastModifiedDate": "2021-09-22T01:16:11.181422",
                "createdBy": "1193c500-15cf-433c-b57e-52f09af6656b",
                "lastModifiedBy": "1193c500-15cf-433c-b57e-52f09af6656b",
                "id": 10,
                "username": "user9",
                "age": 9,
                "team": null
            },
            {
                "createdDate": "2021-09-22T01:16:11.184424",
                "lastModifiedDate": "2021-09-22T01:16:11.184424",
                "createdBy": "bcd896fc-df68-417c-838b-3f0be29a967e",
                "lastModifiedBy": "bcd896fc-df68-417c-838b-3f0be29a967e",
                "id": 11,
                "username": "user10",
                "age": 10,
                "team": null
            },
            {
                "createdDate": "2021-09-22T01:16:11.188428",
                "lastModifiedDate": "2021-09-22T01:16:11.188428",
                "createdBy": "adeb0067-196b-4bd4-bee4-14451ecf7004",
                "lastModifiedBy": "adeb0067-196b-4bd4-bee4-14451ecf7004",
                "id": 12,
                "username": "user11",
                "age": 11,
                "team": null
            }
        ],
        "pageable": {
            "sort": {
                "sorted": false,
                "unsorted": true,
                "empty": true
            },
            "offset": 9,
            "pageSize": 3,
            "pageNumber": 3,
            "paged": true,
            "unpaged": false
        },
        "last": false,
        "totalElements": 100,
        "totalPages": 34,
        "size": 3,
        "number": 3,
        "sort": {
            "sorted": false,
            "unsorted": true,
            "empty": true
        },
        "first": false,
        "numberOfElements": 3,
        "empty": false
    },
    "teamPage": {
        "content": [
            {
                "createdDate": "2021-09-22T01:16:11.482695",
                "lastModifiedDate": "2021-09-22T01:16:11.482695",
                "id": 101,
                "name": "team0",
                "members": []
            },
            {
                "createdDate": "2021-09-22T01:16:11.497709",
                "lastModifiedDate": "2021-09-22T01:16:11.497709",
                "id": 102,
                "name": "team1",
                "members": []
            },
            {
                "createdDate": "2021-09-22T01:16:11.500712",
                "lastModifiedDate": "2021-09-22T01:16:11.500712",
                "id": 103,
                "name": "team2",
                "members": []
            },
            {
                "createdDate": "2021-09-22T01:16:11.503715",
                "lastModifiedDate": "2021-09-22T01:16:11.503715",
                "id": 104,
                "name": "team3",
                "members": []
            },
            {
                "createdDate": "2021-09-22T01:16:11.506718",
                "lastModifiedDate": "2021-09-22T01:16:11.506718",
                "id": 105,
                "name": "team4",
                "members": []
            }
        ],
        "pageable": {
            "sort": {
                "sorted": false,
                "unsorted": true,
                "empty": true
            },
            "offset": 0,
            "pageSize": 5,
            "pageNumber": 0,
            "paged": true,
            "unpaged": false
        },
        "last": false,
        "totalElements": 100,
        "totalPages": 20,
        "size": 5,
        "number": 0,
        "sort": {
            "sorted": false,
            "unsorted": true,
            "empty": true
        },
        "first": true,
        "numberOfElements": 5,
        "empty": false
    }
}
```

위 결과와 같이 페이징 정보가 둘 이상이면 @Qualifier("접두사명") 어노테이션을 이용하여 접두사명을 설정한다. 그리고 접두사명_xxx (member_size, member_page, team_size, team_page)로 구분하여 사용한다.

<br><br>

# Page 내용을 DTO로 변환하기

- 엔티티를 API로 노출하면 다양한 문제가 발생한다. 그래서 엔티티를 꼭 DTO로 변환해서 반환해야 한다.
    - 엔티티를 변경하면 API 스펙이 변경된다.
    - 외부에 내부 구조를 노출시킨다. 등등..
- Page는 map() 을 지원해서 내부 데이터를 다른 것으로 변경할 수 있다.

```java
@Data
public class MemberDto {

    private Long id;
    private String username;
    private String teamName;

    public MemberDto(Long id, String username, String teamName) {
        this.id = id;
        this.username = username;
        this.teamName = teamName;
    }

    public MemberDto(Member member) {
        this.id = member.getId();
        this.username = member.getUsername();

        if(member.getTeam() != null)
            this.teamName = member.getTeam().getName();
    }
}

@RestController
@RequiredArgsConstructor
public class MemberController {

    private final MemberRepository memberRepository;

    @GetMapping("/members2")
    public Page<MemberDto> list2(@PageableDefault(size = 5) Pageable pageable) {
        Page<Member> page = memberRepository.findAll(pageable);
        Page<MemberDto> dto = page.map(m -> new MemberDto(m));
        return dto;
    }

    @PostConstruct
    public void init() {
        for (int i = 0 ; i < 100 ; i ++) {
            Member member = new Member("user"+i,i);
            memberRepository.save(member);
        }
    }
}
```

- 결과

```
{
    "content": [
        {
            "id": 26,
            "username": "user25",
            "teamName": null
        },
        {
            "id": 27,
            "username": "user26",
            "teamName": null
        },
        {
            "id": 28,
            "username": "user27",
            "teamName": null
        },
        {
            "id": 29,
            "username": "user28",
            "teamName": null
        },
        {
            "id": 30,
            "username": "user29",
            "teamName": null
        }
    ],
    "pageable": {
        "sort": {
            "sorted": false,
            "unsorted": true,
            "empty": true
        },
        "offset": 25,
        "pageNumber": 5,
        "pageSize": 5,
        "unpaged": false,
        "paged": true
    },
    "totalPages": 20,
    "totalElements": 100,
    "last": false,
    "size": 5,
    "number": 5,
    "sort": {
        "sorted": false,
        "unsorted": true,
        "empty": true
    },
    "first": false,
    "numberOfElements": 5,
    "empty": false
}
```

위 결과와 같이 DTO로 변환해서 반환하는 것이 바람직하다.

<br><br>

# Page를 1부터 시작하기

스프링 데이터는 Page를 0부터 시작한다. <br>
만약 1부터 시작하려면?

1. Pageable, Page를 파리미터와 응답 값으로 사용히지 않고, 직접 클래스를 만들어서 처리한다. 그리고 직접 PageRequest(Pageable 구현체)를 생성해서 리포지토리에 넘긴다. 물론 응답값도 Page 대신에 직접 만들어서 제공해야 한다.

2. spring.data.web.pageable.one-indexed-parameters 를 true 로 설정한다. 그런데 이 방법은 web에서 page 파라미터를 -1 처리 할 뿐이다. 따라서 응답값인 Page 에 모두 0 페이지 인덱스를 사용하는 한계가 있다.

application.yml

```
spring:
  data:
    web:
      pageable:
        default-page-size: 10
        max-page-size: 100
        one-indexed-parameters: true
```

```java
@RestController
@RequiredArgsConstructor
public class MemberController {

    private final MemberRepository memberRepository;
    
    @GetMapping("/members")
    public Page<Member> list(@PageableDefault(size = 5, sort = "username", direction = Sort.Direction.DESC) Pageable pageable) {
        return memberRepository.findAll(pageable);
    }

    @PostConstruct
    public void init() {
        for (int i = 0 ; i < 100 ; i ++) {
            Member member = new Member("user"+i,i);
            memberRepository.save(member);
        }
    }
}
```

- 결과

```
GET 127.0.0.1:8080/members?page=0

{
    "content": [
        ...
    ],
    "pageable": {
        ...
        "pageNumber": 0,    // 페이지 인덱스
        ...
    },
    ...
    "number": 0,    // 페이지 인덱스
    ...
}


GET 127.0.0.1:8080/members?page=1

{
    "content": [
        ...
    ],
    "pageable": {
        ...
        "pageNumber": 0,    // 페이지 인덱스
        ...
    },
    ...
    "number": 0,    // 페이지 인덱스
    ...
}


GET 127.0.0.1:8080/members?page=2

{
    "content": [
        ...
    ],
    "pageable": {
        ...
        "pageNumber": 1,    // 페이지 인덱스
        ...
    },
    ...
    "number": 1,    // 페이지 인덱스
    ...
}
```

위 결과와 같이 spring.data.web.pageable.one-indexed-parameters = true로 한다면, 페이지 인덱스를 기존 페이지 값에서 -1 한 상태로 처리할 뿐이기 때문에 한계가 있다.

<br><br><br>
## References 및 사진 출처

> __김영한의 실전! 스프링 데이터 JPA__