---
layout: post
title:  "Spring 타입 컨버터 FormattingConversionService"
subtitle:   "Spring 타입 컨버터 FormattingConversionService"
date:   2022-03-06 23:20:27 +0900
categories: spring
tags: spring Type Formatter FormattingConversionService
comments: true
---


<br>

- 목차
    - [FormattingConversionService](#formattingconversionservice)
        - [테스트 코드 확인](#테스트-코드-확인)
    - [DefaultFormattingConversionService 상속 관계](#defaultformattingconversionservice-상속-관계)
    - [Formatter 적용하기](#formatter-적용하기)
        - [Formatter 테스트](#formatter-테스트)
    - [스프링이 제공하는 기본 Formatter](#스프링이-제공하는-기본-formatter)
    - [정리](#정리)
        - [주의!](#주의)


# FormattingConversionService

<br>

컨버전 서비스에는 컨버터만 등록할 수 있고, 포맷터를 등록할 수 는 없다. 그런데 생각해보면 포맷터는 객체 문자, 문자 객체로 변환하는 특별한 컨버터일 뿐이다. <br>
포맷터를 지원하는 컨버전 서비스를 사용하면 컨버전 서비스에 포맷터를 추가할 수 있다. 내부에서 어댑터 패턴을 사용해서 Formatter 가 Converter 처럼 동작하도록 지원한다. <br>
FormattingConversionService 는 포맷터를 지원하는 컨버전 서비스이다. <br>
DefaultFormattingConversionService 는 FormattingConversionService 에 기본적인 통화, 숫자 관련 몇가지 기본 포맷터를 추가해서 제공한다. <br>

<br>
타입 컨버터 Formatter와 이어지는 내용이기 때문에 보지 못했다면 보고오자! <br>
https://sehwan-choi.github.io/spring/2022/03/06/spring-Formatter/

<br><br>

## 테스트 코드 확인

<br>

```java
class MyNumberFormatterTest {

    MyNumberFormatter formatter = new MyNumberFormatter();

    @Test
    void parse() throws ParseException {
        Number result = formatter.parse("1,000", Locale.KOREA);
        Assertions.assertThat(result).isEqualTo(1000L);
    }

    @Test
    void print() {
        String result = formatter.print(1000, Locale.KOREA);
        Assertions.assertThat(result).isEqualTo("1,000");
    }
}
```

위 코드ㅡ와 같이 MyNumberFormatter를 통해서 "1,000" -> 1000으로 1000 -> "1,000"으로 변경할 수 있다.

<br><br>

# DefaultFormattingConversionService 상속 관계

<br>

FormattingConversionService 는 ConversionService 관련 기능을 상속받기 때문에 결과적으로 컨버터도 포맷터도 모두 등록할 수 있다. 그리고 사용할 때는 ConversionService 가 제공하는 convert 를 사용하면 된다. <br>
추가로 스프링 부트는 DefaultFormattingConversionService 를 상속 받은 WebConversionService 를 내부에서 사용한다.<br>

<br><br>

# Formatter 적용하기

<br>

```java
@Configuration
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void addFormatters(FormatterRegistry registry) {
        // 우선순위 주의!
//        registry.addConverter(new StringToIntegerConverter());
//        registry.addConverter(new IntegerToStringConverter());
        registry.addConverter(new IpPortToStringConverter());
        registry.addConverter(new StringToIpPortConverter());

        registry.addFormatter(new MyNumberFormatter());
    }
}
```

위 코드와 같이 AddFormatters를 Override한후 registry에 addConverter로 등록하자. <br>

<br>

> 주의! <br>
> 만일 문자 -> 숫자, 숫자 -> 문자로 변경하는 컨버터를 만들어서 등록했다면 주석처리 해야한다. <br>
> 그렇지 않으면 MyNumberFormatter는 동작하지 않을 것이다.<br>
> <br>
> 왜냐하면 <br>
> Converter가 Formatter보다 우선순위가 높기 때문에 같은 기능이 등록되어 있으면 Converter가 더 높은 우선순위로 인해 먼저 사용된다.

<br><br>

##  Formatter 테스트

<br>

```java
@RestController
public class HelloController {

    @GetMapping("/hello-v2")
    public String helloV2(@RequestParam Integer data) {
        System.out.println("data = " + data);
        return "ok";
    }
}
```

localhost:8080/hello-v2/data=10,000 으로 테스트를 해보자. <br>


```xml
MyNumberFormatter          : text=10000, locale=ko
data = 10000
```

"10,000" 이라는 포맷팅 된 문자가 Integer 타입의 숫자 10000으로 정상 변환 된 것을 확인할 수 있다.
 <br><br>

 # 스프링이 제공하는 기본 Formatter

 <br>

 스프링은 자바에서 기본으로 제공하는 타입들에 대해 수 많은 포맷터를 기본으로 제공한다. <br>
IDE에서 Formatter 인터페이스의 구현 클래스를 찾아보면 수 많은 날짜나 시간 관련 포맷터가 제공되는 것을 확인할 수 있다. <br>
그런데 포맷터는 기본 형식이 지정되어 있기 때문에, 객체의 각 필드마다 다른 형식으로 포맷을 지정하기는 어렵다. <br>
스프링은 이런 문제를 해결하기 위해 애노테이션 기반으로 원하는 형식을 지정해서 사용할 수 있는 매우 유용한 포맷터 두 가지를 기본으로 제공한다.
<br><br>

- @NumberFormat : 숫자 관련 형식 지정 포맷터 사용, NumberFormatAnnotationFormatterFactory
- @DateTimeFormat : 날짜 관련 형식 지정 포맷터 사용,
Jsr310DateTimeFormatAnnotationFormatterFactory

<br><br>

> 참고 <br>
> @NumberFormat , @DateTimeFormat 의 자세한 사용법이 궁금한 분들은 다음 링크를 참고하거나 관련 애노테이션을 검색해보자. <br>
> https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#format-CustomFormatAnnotation

<br><br>

# 정리

컨버터를 사용하든, 포맷터를 사용하든 등록 방법은 다르지만, 사용할 때는 컨버전 서비스를 통해서 일관성 있게 사용할 수 있다.

<br><br>

## 주의!


메시지 컨버터( HttpMessageConverter )에는 컨버전 서비스가 적용되지 않는다. <br>
특히 객체를 JSON으로 변환할 때 메시지 컨버터를 사용하면서 이 부분을 많이 오해하는데, HttpMessageConverter 의 역할은 HTTP 메시지 바디의 내용을 객체로 변환하거나 객체를 HTTP 메시지 바디에 입력하는 것이다. 예를 들어서 JSON을 객체로 변환하는 메시지 컨버터는 내부에서 Jackson 같은
라이브러리를 사용한다. 객체를 JSON으로 변환한다면 그 결과는 이 라이브러리에 달린 것이다. 따라서 JSON 결과로 만들어지는 숫자나 날짜 포맷을 변경하고 싶으면 해당 라이브러리가 제공하는 설정을 통해서
포맷을 지정해야 한다. 결과적으로 이것은 컨버전 서비스와 전혀 관계가 없다.
컨버전 서비스는 @RequestParam , @ModelAttribute , @PathVariable , 뷰 템플릿 등에서 사용할 수 있다.

<br><br><br>
## References 및 사진 출처

> __김영한의 스프링 MVC 2편 - 백엔드 웹 개발 활용 기술__