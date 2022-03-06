---
layout: post
title:  "Spring 타입 컨버터 ConversionService"
subtitle:   "Spring 타입 컨버터 ConversionService"
date:   2022-03-06 22:00:27 +0900
categories: spring
tags: spring Type Converter ConversionService
comments: true
---


<br>

- 목차
    - [ConversionService](#conversionservice)
        - [ConversionServiceTest - 컨버전 서비스 테스트 코드](#conversionservicetest---컨버전-서비스-테스트-코드)
        - [등록과 사용 분리](#등록과-사용-분리)
        - [컨버전 서비스 사용](#컨버전-서비스-사용)
    - [스프링에 Converter 적용하기](#스프링에-converter-적용하기)
        - [동작 확인](#동작-확인)


# ConversionService

<br>

타입 컨버터를 하나하나 직접 찾아서 타입 변환에 사용하는 것은 매우 불편하다. 그래서 스프링은 개별 컨버터를 모아두고 그것들을 묶어서 편리하게 사용할 수 있는 기능을 제공하는데, 이것이 바로 컨버전 서비스( ConversionService )이다.

<br>

타입 컨버터 Converter와 이어지는 내용이기 때문에 보지 못했다면 보고오자! <br>
https://sehwan-choi.github.io/spring/2022/03/06/spring-Converter/

<br>

ConversionService 인터페이스
```java
package org.springframework.core.convert;
import org.springframework.lang.Nullable;
public interface ConversionService {
    boolean canConvert(@Nullable Class<?> sourceType, Class<?> targetType);
    boolean canConvert(@Nullable TypeDescriptor sourceType, TypeDescriptor targetType);
    <T> T convert(@Nullable Object source, Class<T> targetType);
    Object convert(@Nullable Object source, @Nullable TypeDescriptor sourceType, TypeDescriptor targetType);
}
```

컨버전 서비스 인터페이스는 단순히 컨버팅이 가능한가? 확인하는 기능과, 컨버팅 기능을 제공한다.

<br><br>

## ConversionServiceTest - 컨버전 서비스 테스트 코드

```java
public class ConversionServiceTest {

    @Test
    void conversionService() {
        // 등록
        DefaultConversionService conversionService = new DefaultConversionService();
        conversionService.addConverter(new StringToIntegerConverter());
        conversionService.addConverter(new IntegerToStringConverter());
        conversionService.addConverter(new StringToIpPortConverter());
        conversionService.addConverter(new IpPortToStringConverter());

        Assertions.assertThat(conversionService.convert("10", Integer.class)).isEqualTo(10);
        Assertions.assertThat(conversionService.convert(10, String.class)).isEqualTo("10");
        Assertions.assertThat(conversionService.convert("127.0.0.1:8080", IpPort.class)).isEqualTo(new IpPort("127.0.0.1",8080));
        Assertions.assertThat(conversionService.convert(new IpPort("127.0.0.1",8080), String.class)).isEqualTo("127.0.0.1:8080");
    }
}
```

DefaultConversionService 는 ConversionService 인터페이스를 구현했는데, 추가로 컨버터를 등록하는 기능도 제공한다.

<br>

## 등록과 사용 분리

<br>

컨버터를 등록할 때는 StringToIntegerConverter 같은 타입 컨버터를 명확하게 알아야 한다. 반면에 컨버터를 사용하는 입장에서는 타입 컨버터를 전혀 몰라도 된다. <br>
타입 컨버터들은 모두 컨버전 서비스 내부에 숨어서 제공된다. 따라서 타입을 변환을 원하는 사용자는 컨버전 서비스 인터페이스에만 의존하면 된다. <br>
물론 컨버전 서비스를 등록하는 부분과 사용하는 부분을 분리하고 의존관계 주입을 사용해야 한다.

<br>

## 컨버전 서비스 사용

```
Integer value = conversionService.convert("10", Integer.class)
```

<br>

## 인터페이스 분리 원칙 - ISP(Interface Segregation Principal)

<br>

인터페이스 분리 원칙은 클라이언트가 자신이 이용하지 않는 메서드에 의존하지 않아야 한다. <br>


DefaultConversionService 는 다음 두 인터페이스를 구현했다.
- ConversionService : 컨버터 사용에 초점
- ConverterRegistry : 컨버터 등록에 초점

<br>

이렇게 인터페이스를 분리하면 컨버터를 사용하는 클라이언트와 컨버터를 등록하고 관리하는 클라이언트의 관심사를 명확하게 분리할 수 있다. <br>
특히 컨버터를 사용하는 클라이언트는 ConversionService 만 의존하면 되므로, 컨버터를 어떻게 등록하고 관리하는지는 전혀 몰라도 된다. <br>
결과적으로 컨버터를 사용하는 클라이언트는 꼭 필요한 메서드만 알게된다. 이렇게 인터페이스를 분리하는 것을 ISP 라 한다. <br>

ISP 참고: https://ko.wikipedia.org/wiki/%EC%9D%B8%ED%84%B0%ED%8E%98%EC%9D%B4%EC%8A%A4_%EB%B6%84%EB%A6%AC_%EC%9B%90%EC%B9%99

<br>

스프링은 내부에서 ConversionService 를 사용해서 타입을 변환한다. 예를 들어서 앞서 살펴본 @RequestParam 같은 곳에서 이 기능을 사용해서 타입을 변환한다. <br>
이제 컨버전 서비스를 스프링에 적용해보자.

<br><br>

# 스프링에 Converter 적용하기

<br>

WebConfig - 컨버터 등록
```java
@Configuration
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void addFormatters(FormatterRegistry registry) {
        registry.addConverter(new StringToIntegerConverter());
        registry.addConverter(new IntegerToStringConverter());
        registry.addConverter(new IpPortToStringConverter());
        registry.addConverter(new StringToIpPortConverter());
    }
}
```

스프링은 내부에서 ConversionService 를 제공한다. 우리는 WebMvcConfigurer 가 제공하는 addFormatters() 를 사용해서 추가하고 싶은 컨버터를 등록하면 된다. 이렇게 하면 스프링은 내부에서 사용하는 ConversionService 에 컨버터를 추가해준다 <br>

<br><br>

## 동작 확인

```java
@RestController
public class HelloController {

    @GetMapping("/hello-v2")
    public String helloV2(@RequestParam Integer data) {
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

localhost:8080/hello-v2?data=10을 실행해보자. <br>
실행한 결과 아래와 같은 실행 로그를 볼 수 있다. <br>

```
StringToIntegerConverter : convert source=10
data = 10
```

?data=10 의 쿼리 파라미터는 문자이고 이것을 Integer data 로 변환하는 과정이 필요하다. <br>
실행해보면 직접 등록한 StringToIntegerConverter 가 작동하는 로그를 확인할 수 있다. <br>
그런데 생각해보면 StringToIntegerConverter 를 등록하기 전에도 이 코드는 잘 수행되었다. 그것은 스프링이 내부에서 수 많은 기본 컨버터들을 제공하기 때문이다. 컨버터를 추가하면 추가한 컨버터가 기본 컨버터 보다 높은 우선순위를 가진다. <br>

<br>


localhost:8080/hello-v3?data=127.0.0.1:8080을 실행해보자. <br>
실행한 결과 아래와 같은 실행 로그를 볼 수 있다. <br>

```
StringToIpPortConverter    : convert source=127.0.0.1:8080
data = IpPort(ip=127.0.0.1, port=8080)
```

?ipPort=127.0.0.1:8080 쿼리 스트링이 @RequestParam IpPort ipPort 에서 객체 타입으로 잘 변환 된 것을 확인할 수 있다. <br>


- 처리과정

@RequestParam 은 @RequestParam 을 처리하는 ArgumentResolver 인 RequestParamMethodArgumentResolver 에서 ConversionService 를 사용해서 타입을 변환한다. <br> 
부모 클래스와 다양한 외부 클래스를 호출하는 등 복잡한 내부 과정을 거치기 때문에 대략 이렇게 처리되는 것으로 이해해도 충분하다. 만약 더 깊이있게 확인하고 싶으면 IpPortConverter 에 디버그 브레이크 포인트를 걸어서 확인해보자.

<br><br><br>
## References 및 사진 출처

> __김영한의 스프링 MVC 2편 - 백엔드 웹 개발 활용 기술__