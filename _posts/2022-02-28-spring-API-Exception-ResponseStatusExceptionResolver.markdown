---
layout: post
title:  "Spring API 예외 처리 - ResponseStatusExceptionResolver"
subtitle:   "Spring API 예외 처리 - ResponseStatusExceptionResolver"
date:   2022-03-02 01:00:27 +0900
categories: spring
tags: spring ResponseStatusExceptionResolver
comments: true
---


<br>

- 목차
    - [ResponseStatusExceptionResolver](#responsestatusexceptionresolver)
    - [@ResponseStatus 가 달려있는 예외처리](#responsestatus-가-달려있는-예외처리)
        - [@ResponseStatus 가 달려있는 예외처리 결과](#responsestatus-가-달려있는-예외처리-결과)
    - [ResponseStatusException 예외 처리](#responsestatusexception-예외-처리)
        - [ResponseStatusException 예외 처리](#responsestatusexception-예외-처리)


# ResponseStatusExceptionResolver

<br>

ResponseStatusExceptionResolver 는 예외에 따라서 HTTP 상태 코드를 지정해주는 역할을 한다. <br><br>

다음 두 가지 경우를 처리한다.
- @ResponseStatus 가 달려있는 예외
- ResponseStatusException 예외

<br><br>

# @ResponseStatus 가 달려있는 예외처리

<br>

```java
@ResponseStatus(code = HttpStatus.BAD_REQUEST, reason = "error.bad")
public class BadRequestException extends RuntimeException{
    
}
```

위 코드와 같이 사용자 정의 Exception 위에 @ResponseStatus사용 한경우 HTTP 상태 코드를 변경시켜준다. <br>
그리고 reason에는 아래와 같이 messages.properties에 설정한 메세지를 사용할 수 있다. <br>

messagges.properties
```properties
error.bad=잘못된 요청 입니다. 메세지 사용
```

<br>

BadRequestException 예외가 컨트롤러 밖으로 넘어가면 ResponseStatusExceptionResolver 예외가 해당 애노테이션을 확인해서 오류 코드를 HttpStatus.BAD_REQUEST (400)으로 변경하고, 메시지도
담는다. <br>
ResponseStatusExceptionResolver 코드를 확인해보면 결국 response.sendError(statusCode, resolvedReason) 를 호출하는 것을 확인할 수 있다. <br>
sendError(400) 를 호출했기 때문에 WAS에서 다시 오류 페이지( /error )를 내부 요청한다.

<br>

ApiExceptionController
```java
@Slf4j
@RestController
public class ApiExceptionController {

    @GetMapping("/api/response-status-ex1")
    public String responseStatusEx1() {
        throw new BadRequestException();
    }
}
```

/api/response-status-ex1 를 호출 하게 되면, 원래는 500에러가 발생해야 하지만 BadRequestException에 @ResponseStatus에 의해서 HttpStatus.BAD_REQUESTHttpStatus.BAD_REQUEST 에러로 변경 되어서 출력된다.

<br>

## @ResponseStatus 가 달려있는 예외처리 결과

```json
{
    "timestamp": "2022-03-01T16:00:49.617+00:00",
    "status": 400,
    "error": "Bad Request",
    "exception": "hello.exception.exception.BadRequestException",
    "message": "잘못된 요청 입니다. 메세지 사용",
    "path": "/api/response-status-ex1"
}
```

<br><br>

# ResponseStatusException 예외 처리

<br>

@ResponseStatus 는 개발자가 직접 변경할 수 없는 예외에는 적용할 수 없다. (애노테이션을 직접 넣어야 하는데, 내가 코드를 수정할 수 없는 라이브러리의 예외 코드 같은 곳에는 적용할 수 없다.) <br>
추가로 애노테이션을 사용하기 때문에 조건에 따라 동적으로 변경하는 것도 어렵다. 이때는 ResponseStatusException 예외를 사용하면 된다. <br>

<br>

ApiExceptionController
```java
@Slf4j
@RestController
public class ApiExceptionController {

    @GetMapping("/api/response-status-ex2")
    public String responseStatusEx2() {
        throw new ResponseStatusException(HttpStatus.NOT_FOUND, "error.bad", new IllegalArgumentException());
    }
}
```

위 코드와 같이 /api/response-status-ex2 호출시 실제로는 IllegalArgumentException가 발생하여 500에러가 발생해야 하지만 ResponseStatusException를 사용하여 404에러로 변경 하였고, 원하는 메세지도 출력해줄수 있다.

<br>

## ResponseStatusException 예외 처리 결과

```json
{
    "timestamp": "2022-03-01T16:05:03.996+00:00",
    "status": 404,
    "error": "Not Found",
    "exception": "org.springframework.web.server.ResponseStatusException",
    "message": "잘못된 요청 입니다. 메세지 사용",
    "path": "/api/response-status-ex2"
}
```

<br><br><br>
## References 및 사진 출처

> __김영한의 스프링 MVC 2편 - 백엔드 웹 개발 활용 기술__