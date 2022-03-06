---
layout: post
title:  "Spring 타입 컨버터 Converter"
subtitle:   "Spring 타입 컨버터 Converter"
date:   2022-03-06 21:17:27 +0900
categories: spring
tags: spring Type Converter
comments: true
---


<br>

- 목차
    - [Converter](#converter)
        - [스프링의 타입 변환 적용 예](#스프링의-타입-변환-적용-예)
        - [스프링과 타입 변환](#스프링과-타입-변환)
    - [타입 컨버터 - Converter](#타입-컨버터---converter)
        - [StringToIntegerConverter - 문자를 숫자로 변환하는 타입 컨버터](#stringtointegerconverter---문자를-숫자로-변환하는-타입-컨버터)
        - [IntegerToStringConverter - 숫자를 문자로 변환하는 타입 컨버터](#integertostringconverter---숫자를-문자로-변환하는-타입-컨버터)
        - [테스트 코드 검증](#테스트-코드-검증)
    - [사용자 정의 타입 컨버터](#사용자-정의-타입-컨버터)
        - [StringToIpPortConverter - 컨버터](#stringtoipportconverter---컨버터)
        - [테스트 코드 검증](#테스트-코드-검증)


# Converter

```java
@RestController
public class HelloController {

    @GetMapping("/hello-v1")
    public String helloV1(@RequestParam Integer data) {
        System.out.println("data = " + data);
        return "ok";
    }

    @GetMapping("/hello-v3")
    public String helloV3(@RequestParam IpPort data) {
        System.out.println("data = " + data);
        return "ok";
    }
}
```

위 소스에서 /hello-v1/data=10 으로 보낸다면 data = 10 로 나올 것이다. <br>
스프링이 제공하는 @RequestParam 을 사용하면 이 문자 10을 Integer 타입의 숫자 10으로 편리하게 받을 수 있다. <br>
또한 객체 형태인 IpPort로 받을 수 있다. 이것은 스프링이 중간에서 타입을 변환해주었기 때문이다.

<br><br>

```java
@RestController
public class HelloController {
     @GetMapping("/hello-v1")
    public String helloV1(HttpServletRequest request) {
        String data = request.getParameter("data"); //문자 타입 조회
        Integer intValue = Integer.valueOf(data); //숫자 타입으로 변경
        System.out.println("intValue = " + intValue);
        return "ok";
    }
}
```

원래는 위와 같이 getParameter로 받은후에 타입을 변경해야 한다. 
<br>
하지만 위와 같은 일을 스프링이 자동으로 타입변환을 해준다. 

<br><br>

## 스프링의 타입 변환 적용 예

- 스프링 MVC 요청 파라미터
    -  @RequestParam , @ModelAttribute , @PathVariable
- @Value 등으로 YML 정보 읽기
- XML에 넣은 스프링 빈 정보를 변환
- 뷰를 렌더링 할 때

<br><br>

## 스프링과 타입 변환

이렇게 타입을 변환해야 하는 경우는 상당히 많다. 개발자가 직접 하나하나 타입 변환을 해야 한다면, 생각만 해도 괴로울 것이다. <br>
스프링이 중간에 타입 변환기를 사용해서 타입을 String Integer 로 변환해주었기 때문에 개발자는 편리하게 해당 타입을 바로 받을 수 있다. 앞에서는 문자를 숫자로 변경하는 예시를 들었지만, 반대로 숫자를 문자로 변경하는 것도 가능하고, Boolean 타입을 숫자로 변경하는 것도 가능하다. 만약 개발자가 새로운 타입을 만들어서 변환하고 싶으면 어떻게 하면 될까?

<br><br>

컨버터 인터페이스
```java
package org.springframework.core.convert.converter;
public interface Converter<S, T> {
    T convert(S source);
}
```

스프링은 확장 가능한 컨버터 인터페이스를 제공한다. <br>
개발자는 스프링에 추가적인 타입 변환이 필요하면 이 컨버터 인터페이스를 구현해서 등록하면 된다. <br>
이 컨버터 인터페이스는 모든 타입에 적용할 수 있다. 필요하면 X Y 타입으로 변환하는 컨버터 인터페이스를 만들고, 또 Y X 타입으로 변환하는 컨버터 인터페이스를 만들어서 등록하면 된다. <br>
예를 들어서 문자로 "true" 가 오면 Boolean 타입으로 받고 싶으면 String Boolean 타입으로 변환되도록 컨버터 인터페이스를 만들어서 등록하고, 반대로 적용하고 싶으면 Boolean String 타입으로 변환되도록 컨버터를 추가로 만들어서 등록하면 된다. 

<br><br>

> 참고 <br>
> 과거에는 PropertyEditor 라는 것으로 타입을 변환했다. PropertyEditor 는 동시성 문제가 있어서
타입을 변환할 때 마다 객체를 계속 생성해야 하는 단점이 있다. 지금은 Converter 의 등장으로 해당
문제들이 해결되었고, 기능 확장이 필요하면 Converter 를 사용하면 된다.

<br><br>

# 타입 컨버터 - Converter

타입 컨버터를 어떻게 사용하는지 코드로 알아보자. <br>
타입 컨버터를 사용하려면 org.springframework.core.convert.converter.Converter
인터페이스를 구현하면 된다. 

<br><br>

> 주의 <br>
> Converter 라는 이름의 인터페이스가 많으니 조심해야 한다. <br>
org.springframework.core.convert.converter.Converter 를 사용해야 한다.

<br><br>

## StringToIntegerConverter - 문자를 숫자로 변환하는 타입 컨버터

<br>

```java
@Slf4j
public class StringToIntegerConverter implements Converter<String, Integer> {
    @Override
    public Integer convert(String source) {
        log.info("convert source={}" ,source);

        return Integer.valueOf(source);
    }
}
```

String -> Integer 로 변환하기 때문에 소스가 String 이 된다. 이 문자를
Integer.valueOf(source) 를 사용해서 숫자로 변경한 다음에 변경된 숫자를 반환하면 된다. 

<br><br>

## IntegerToStringConverter - 숫자를 문자로 변환하는 타입 컨버터

<br>

```java
@Slf4j
public class IntegerToStringConverter implements Converter<Integer, String> {
    @Override
    public String convert(Integer source) {
        log.info("sonvert source={}",source);
        return String.valueOf(source);
    }
}
```

이번에는 숫자를 문자로 변환하는 타입 컨버터이다. 앞의 컨버터와 반대의  일을 한다. 이번에는 숫자가 입력되기 때문에 소스가 Integer 가 된다. <br>
String.valueOf(source) 를 사용해서 문자로 변경한 다음 변경된 문자를 반환하면 된다.

<br><br>

## 테스트 코드 검증

<br>

```java
class ConverterTest {

    @Test
    void stringToInteger() {
        StringToIntegerConverter converter = new StringToIntegerConverter();
        Integer result = converter.convert("10");
        Assertions.assertThat(result).isEqualTo(10);
    }

    @Test
    void IntegerToString() {
        IntegerToStringConverter converter = new IntegerToStringConverter();
        String result = converter.convert(10);
        Assertions.assertThat(result).isEqualTo("10");
    }
}
```

<br><br>

# 사용자 정의 타입 컨버터

<br><br>

```java
@Getter
@ToString
@EqualsAndHashCode
public class IpPort {
    private String ip;
    private int port;

    public IpPort(String ip, int port) {
        this.ip = ip;
        this.port = port;
    }
}

```

롬복의 @EqualsAndHashCode 를 넣으면 모든 필드를 사용해서 equals() , hashcode() 를 생성한다.  <br>
따라서 모든 필드의 값이 같다면 a.equals(b) 의 결과가 참이 된다.

<br>

## StringToIpPortConverter - 컨버터

<br>

```java
@Slf4j
public class StringToIpPortConverter implements Converter<String, IpPort> {
    @Override
    public IpPort convert(String source) {
        log.info("convert source={}", source);
        String[] split = source.split(":");
        String ip = split[0];
        int port = Integer.parseInt(split[1]);
        return new IpPort(ip,port);
    }
}
```

127.0.0.1:8080 같은 문자를 입력하면 IpPort 객체를 만들어 반환한다.

<br>

```java
@Slf4j
public class IpPortToStringConverter implements Converter<IpPort, String> {
    @Override
    public String convert(IpPort source) {
        log.info("convert source={}", source);

        return source.getIp() + ":" + source.getPort();
    }
}
```
IpPort 객체를 입력하면 127.0.0.1:8080 같은 문자를 반환한다.

<br>

## 테스트 코드 검증

<br>

```java
class ConverterTest {
    @Test
    void stringToIpPort() {
        IpPortToStringConverter converter = new IpPortToStringConverter();
        IpPort ipPort = new IpPort("127.0.0.1",8080);
        String result = converter.convert(ipPort);
        Assertions.assertThat(result).isEqualTo("127.0.0.1:8080");
    }

    @Test
    void ipPortToString() {
        StringToIpPortConverter converter = new StringToIpPortConverter();
        String source = "127.0.0.1:8080";
        IpPort result = converter.convert(source);
        Assertions.assertThat(result).isEqualTo(new IpPort("127.0.0.1",8080));
    }
}
```

타입 컨버터 인터페이스가 단순해서 이해하기 어렵지 않을 것이다. <br>
그런데 이렇게 타입 컨버터를 하나하나 직접 사용하면, 개발자가 직접 컨버팅 하는 것과 큰 차이가 없다. <br>
타입 컨버터를 등록하고 관리하면서 편리하게 변환 기능을 제공하는 역할을 하는 무언가가 필요하다. <br>

> 참고 <br>
> 스프링은 용도에 따라 다양한 방식의 타입 컨버터를 제공한다.  <br>
><br>
> Converter -> 기본 타입 컨버터 <br>
> ConverterFactory -> 전체 클래스 계층 구조가 필요할 때 <br>
> GenericConverter -> 정교한 구현, 대상 필드의 애노테이션 정보 사용 가능 <br>
> ConditionalGenericConverter -> 특정 조건이 참인 경우에만 실행 <br>
> <br>
> 자세한 내용은 공식 문서를 참고하자. <br>
> https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#core-convert <br>

<br>

> 참고 <br>
> 스프링은 문자, 숫자, 불린, Enum등 일반적인 타입에 대한 대부분의 컨버터를 기본으로 제공한다. IDE에서 Converter , ConverterFactory , GenericConverter 의 구현체를 찾아보면 수 많은 컨버터를 확인할 수 있다.


<br><br><br>
## References 및 사진 출처

> __김영한의 스프링 MVC 2편 - 백엔드 웹 개발 활용 기술__