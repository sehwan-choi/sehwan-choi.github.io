---
layout: post
title:  "Customize Exception"
subtitle:   "Google Colab Preferences and Usage"
date:   2021-03-17 12:28:27 +0900
categories: spring
tags: java CustomizeException ControllerAdvice ExceptionHandler
comments: true
---

## @ControllerAdvice @ExceptionHandler 를 이용한 Customize Exception 만들기

- 목차
	- [예외 처리 과정](#예외-처리-과정) 
	- [@ExceptionHandler](#@ExceptionHandler)
	- [@ControllerAdvice](#@ControllerAdvice) 
	- [@ExceptionHandler @ControllerAdvice 동시사용](#@ExceptionHandler-@ControllerAdvice-동시사용)

## 예외 처리 과정
프로그래밍에서 예외 처리는 아주 중요하면서도 어렵다.
좋은 API를 만든다는 것은 GET, POST, PUT, DELETE 각각의 API 마다 예외처리가 필요하며 상황에 맞는 HTTP STATUS를 반환해야한다.
예외 처리를 하는 경우와 방법은 다양하다.

- 메서드 내에서 예외 상황을 예측해서 처리하는 try-catch문을 이용하는 방법
- 요구사항에 의한 예외 처리 (ex. validation > 특정 값이 0~255범위가 아니면 유효하지않은 값으로 판단하고 예외 처리)
- 스프링 시큐리티에서 인터셉터로 잡아서 UnauthorizedException 같은 예외 처리
- 기타 여러 예외 처리들을 적용하다보면 코드가 엄청나게 복잡해진다.

if문으로 잡든 try-catch로 잡든 상위 메서드로 예외처리를 위임하든 코드는 복잡해진다

그렇게 되면 유지보수하기 아주 어려워진다.

비즈니스 로직에 집중하기 어렵고, 비즈니스 로직과 관련된 코드보다 예외 처리를 위한 코드가 더 많아지는 경우도 생긴다.

이런 문제를 조금이라도 개선하기 위해 @ExceptionHandler와 @ControllerAdvice를 사용한다고 보면 이해가 쉬워진다.

---


## @ExceptionHandler
@ExceptionHandler 경우는 @Controller, @RestController가 적용된 Bean내에서 발생하는 예외를 잡아서 하나의 메서드에서 처리해주는 기능을 한다.

```c
@RestController 
public class MyRestController { 
    ... 
    ... 
    @ExceptionHandler(NullPointerException.class) 
    public Object nullex(Exception e) { 
        System.err.println(e.getClass()); 
        return "myService"; 
    } 
}
```
위와 같이 적용하기만 하면 된다. @ExceptionHandler라는 어노테이션을 쓰고 인자로 캐치하고 싶은 예외클래스를 등록해주면 끝난다.

@ExceptionHandler({ Exception1.class, Exception2.class}) 이런식으로 두 개 이상 등록도 가능하다.

위의 예제에서 처럼하면 MyRestController에 해당하는 Bean에서 NullPointerException이 발생한다면, @ExceptionHandler(NullPointerException.class)가 적용된 메서드가 호출될 것이다.

주의사항/알아 둘 것

Controller, RestController에만 적용가능하다. (@Service같은 빈에서는 안됨.)
리턴 타입은 자유롭게 해도 된다. (Controller내부에 있는 메서드들은 여러 타입의 response를 할 것이다. 해당 타입과 전혀다른 리턴 타입이어도 상관없다.)
@ExceptionHandler를 등록한 Controller에만 적용된다. 다른 Controller에서 NullPointerException이 발생하더라도 예외를 처리할 수 없다.
메서드의 파라미터로 Exception을 받아왔는데 이것 또한 자유롭게 받아와도 된다.
예제를 보면서 테스트를 해보도록하자.

```c
@RestController 
public class MyRestController { 
    @Autowired 
    private MyService myService; 
    @GetMapping("/hello") 
    public String hello() { 
        return "hello";//문자열 반환 
    } 
    
    @GetMapping("/myData") 
    public MyData myData() { 
        return new MyData("myName");//object 반환 
    } 
    
    @GetMapping("/service") public String serviceCall() { 
        return myService.serviceMethod();//일반적인 service호출 
    } 
    
    @GetMapping("/serviceException") 
    public String serviceException() { 
        return myService.serviceExceptionMethod(); //service에서 예외발생 
    } 
    
    @GetMapping("/controllerException") 
    public void controllerException() { 
        throw new NullPointerException();//controller에서 예외발생 
    } 
    
    @GetMapping("/customException") public String custom() { 
        throw new CustomException();//custom예외 발생
    } 
    
    @ExceptionHandler(NullPointerException.class) 
    public Object nullex(Exception e) { 
        System.err.println(e.getClass()); 
        return "myService"; 
    } 
}
```
[MyRestController.class]
String타입과 MyData라는 나만의 객체타입을 리턴하는 메서드등의 존재하지만 ExceptionHandler하나로 다 처리할 수 있다.

myService.serviceExceptionMethod는 Service안에서 Exception이 발생하는데 이 메서드를 호출하면 서비스에서 예외가 발생했지만 결국 컨트롤러 내에서 발생한 것과 같으므로 ExceptionHandler가 예외를 잡아내어 "myService"가 리턴된다.

```c
public class CustomException extends RuntimeException{ 
    private static final long serialVersionUID = 1L; 
}
```
RuntimeException을 확장한 클래스로 CustomException을 만들었다.

이 예외는 NullPointerException이 아니기 때문에 발생하더라도 ExceptionHandler에 의해서 처리되지 않는다.

만약 하나로 더 많은 예외 처리를 하길 원한다면 모든 예외의 부모클래스인 Exception.class를 핸들링하게하면 된다.
```c
@ExceptionHandler(Exception.class)
```



## @ControllerAdvice
@ExceptionHandler가 하나의 클래스에 대한 것이라면, @ControllerAdvice는 모든 @Controller 즉, 전역에서 발생할 수 있는 예외를 잡아 처리해주는 annotation이다.

```c
@RestControllerAdvice 
public class MyAdvice { 
    @ExceptionHandler(CustomException.class) 
    public String custom() { 
        return "hello custom"; 
    } 
}
```
위와 같이 새로운 클래스파일을 만들어서 annotation을 붙이기만 하면 된다. 그 다음에 @ExceptionHandler로 처리하고 싶은 예외를 잡아 처리하면 된다.

별도의 속성값이 없이 사용하면 모든 패키지 전역에 있는 컨트롤러를 담당하게 된다.

@RestControllerAdvice와 @ControllerAdvice가 존재하는데 @RestControllerAdvice 어노테이션을 들여다보면 아래와 같이 되어있다.
```c
@Target(ElementType.TYPE) 
@Retention(RetentionPolicy.RUNTIME) 
@Documented 
@ControllerAdvice 
@ResponseBody 
public @interface RestControllerAdvice { 
    //... 
}
```
@ControllerAdvice와 동일한 역할 즉, 예외를 잡아 핸들링 할 수 있도록 하는 기능을 수행하면서 @ResponseBody를 통해 객체를 리턴할 수도 있다는 얘기다.

ViewResolver를 통해서 예외 처리 페이지로 리다이렉트 시키려면 @ControllerAdvice만 써도 되고, API서버여서 에러 응답으로 객체를 리턴해야한다면 @ResponseBody 어노테이션이 추가된 @RestControllerAdvice를 적용하면 되는 것이다.

@RestController에서 예외가 발생하든 @Controller에서 예외가 발생하든 @ControllerAdvice + @ExceptionHandler 조합으로 다 캐치할 수 있고 @ResponseBody의 필요 여부에 따라 적용하면 된다는 것이다.



또한, 만약에 전역의 예외를 잡긴하되 패키지 단위로 제한할 수도있다.
```c
@RestControllerAdvice("com.example.demo.login.controller")
```
login모듈에 있는 RestController에서 발생하는 예외를 잡으려면 위와 같이 하면 된다. (패키지 구성을 잘하면 유용하다)


## @ExceptionHandler @ControllerAdvice 동시사용

@ExceptionHandler 와 @ControllerAdvice 를 동시에 사용할수 있다.

```c
@Data
@AllArgsConstructor
@NoArgsConstructor
public class ExceptionResponse {
    private Date timestamp;
    private String message;
    private String details;
}
```
ExceptionResponse.java

```c
@RestController
@ControllerAdvice
public class CustomizeResponseEntityExceptionHandler extends ExceptionResponse{

    @ExceptionHandler(Exception.class)
    public final ResponseEntity<Object> handleAllExceptions(Exception ex, WebRequest request){
        ExceptionResponse exceptionResponse =
                new ExceptionResponse(new Date(),ex.getMessage(),request.getDescription(false));
        return new ResponseEntity(exceptionResponse, HttpStatus.INTERNAL_SERVER_ERROR);
    }

    @ExceptionHandler(UserNotFoundException.class)
    public final ResponseEntity<Object> handleUserNotFoundExceptions(Exception ex, WebRequest request){
        ExceptionResponse exceptionResponse =
                new ExceptionResponse(new Date(),ex.getMessage(),request.getDescription(false));
        return new ResponseEntity(exceptionResponse, HttpStatus.NOT_FOUND);
    }
}
```
CustomizeResponseEntityExceptionHandler.java

위의 소스를 보면 Controller에서 UserNotFoundException 발생시 handleUserNotFoundExceptions로 나머지 Exception 발생시 handleAllExceptions로 예외처리를 하게 된다.

> 참고 : [https://jeong-pro.tistory.com/195](https://jeong-pro.tistory.com/195)