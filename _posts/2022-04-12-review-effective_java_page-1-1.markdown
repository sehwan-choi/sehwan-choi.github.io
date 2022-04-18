---
layout: post
title:  "Review Effective Java page1 - 생성자 대신 정적 팩터리 메서드를 고려하라"
subtitle:   "Review Effective Java page1 - 생성자 대신 정적 팩터리 메서드를 고려하라"
date:   2022-04-18 23:49:27 +0900
categories: review
tags: review effectivejava
comments: true
---

<br>

- 목차
    - [Effective Java - 생성자 대신 정적 팩터리 메서드를 고려하라](#effective-java---생성자-대신-정적-팩터리-메서드를-고려하라)
    - [정적 팩터리 메서드의 장점](#정적-팩터리-메서드의-장점)
    - [서비스 제공자 프레임워크](#서비스-제공자-프레임워크)
    - [정적 팩터리 메서드의 단점](#정적-팩터리-메서드의-단점)
    - [정적 팩터리 명명 방식](#정적-팩터리-명명-방식)
    - [실무 사용경험](#실무-사용경험)

<br>

# Effective Java - 생성자 대신 정적 팩터리 메서드를 고려하라

<br>

클래스의 인스턴스를 얻는 전통적인 수단은 public 생성자이다. <br>
하지만 모든 프로그래머가 꼭 알아둬야 할 기법이 하나 더 있다. <br>
클래스는 생성자와 별도로 정적 팩터리 메서드를 제공할 수 있다. <br>
그 클래스의 인스턴스를 반환하는 단순한 정적 메서드 이다.<br>

<br>

```java
public static Boolean valueOf(boolean b) {
    return b ? Boolean.TRUE : Boolean.FALSE;
}
```

위와 같이 기본 타입인 boolean 값을 받아 Boolean 객체 참조로 변환하는 메서드를 말한다. <br><br>

> 주의! <br>
지금 얘기하는 정적 팩터리 메서드는 디자인 패턴에서의 팩터리 매서드와는 다르다.<br>

<br>

클래스는 클라이언트에 public 생성자 대신 정적 팩터리 메서드를 제공할 수 있다. 이 방식에는 장점과 단점이 모두 존재한다.<br>

<br><br>

# 정적 팩터리 메서드의 장점

<br>

## 1. 이름을 가질 수 있다.

<br>

생성자에 넘기는 매개변수와 생겅자 자체만으로는 반환될 객체의 특성을 제대로 설명하지 못한다. <br>
반면 정적 팩터리는 이름만 잘 지으면 반환될 객체의 특성을 쉽게 묘사할 수 있다.<br>

```java
new LocalDateTime(LocalDate, LocalTime);

LocalDateTime.now();
```

위의 둘중에 어떤것이 현재 시간을 반환해 줄것 같은지 잘 생각해보라<br>

<br><br>

## 2. 호출될 때마다 인스턴스를 새로 생성하지는 않아도 된다.

<br>

이 덕분에 불변 클래스는 인스턴스를 미리 만들어 놓거나 새로 생성한 인스턴스를 캐싱하여 재활용하는 식으로 불필요한 객체 생성을 피할 수 있다. <br>
대표적인 예로, 위에서 설명한 Boolean.valueOf(boolean) 메서드는 객체를 아예 생성하지 않는다. 따라서 생성 비용이 큰 객체가 자주 요청되는 상황이라면 성능을 상당히 끌어올려준다. <br>

<br><br>

## 3. 반환 타입의 하위 타입 객체를 반환할 수 있는 능력이 있다.

<br>

```java
public interface Foo {

    static FooImpl createFooImpl() {
        return new FooImpl();
    }
}

public class FooImpl implements Foo{
}

public class main {

    public static void main(String[] args) {
        Foo.createFooImpl();

        FooImpl fooImpl = Foo.createFooImpl();
    }
}

```
위와같이 이 능력은 반환할 객체의 클래스를 자유롭게 선택할 수 있게 하는 '엄청난 유연성'을 선물한다.<br>
이는 인터페이스를 정적 팩터리 메서드의 반환 타입으로 사용하는 인터페이스 기반 프레임워크를 만드는 핵심 기술이기도 하다.<br>
API를 만들 때 이 유연성을 응용하면 구현 클래스를 공개하지 않고도 그 객체를 반환할 수 있어 API를 작게 유지할 수 있다.<br>

<br><br>

## 4. 입력 매개변수에 따라 매번 다른 클래스의 객체를 반환할 수 있다.

<br>

![그림1](https://sehwan-choi.github.io/assets/img/review/effectiveJava/page1/subtitle/image.JPG)

위 사진과 같이 반환 타입의 하위 타입이기만 하면 어떤 클래스의 객체를 반환하든 상관없다. 클라이언트는 팩터리가 건네주는 객체가 어느 클래스의 인스턴스인지 알 수도 없고 알 필요도 없다.<br>
심지어 또 다들 클래스의 객체를 반환해도 된다.<br>
위의 EnumSet클래스는 public 생성자 없이 오직 정적 팩터리만 제공하는데 원소의 수에 따라 두 가지 하위 클래스 중 하나의 인스턴스를 반환한다.

<br><br>

## 5. 정적 팩터리 메서드를 작성하는 시점에는 반환할 객체의 클래스가 존재하지 않아도 된다.

<br>

```java
public interface TicketSeller {
}

public class TicketStore {

    /**
     * TicketSeller는 인터페이스이고 구현체가 없음에도 아래와 같은 메서드 작성이 가능하다.
     */

    public static List<TicketSeller> getSellers() {
        return new ArrayList<>();
    }
}
```

인터페이스나 클래스가 만들어지는 시점에서 하위 타입의 클래스가 존재 하지 않아도 나중에 만들 클래스가 기존의 인터페이스나 클래스를 상속 받으면 언제든지 의존성을 주입 받아서 사용이 가능한다.<br>
위와 같이 반환값이 인터페이스가 되며 정적 팩터리 메서드의 변경 없이 구현체를 바꿔 끼울수 있다.<br>

<br><br>

## 서비스 제공자 프레임워크

<br>

다양한 서비스 제공자들이 하나의 서비스를 구성하는 시스템으로 클라이언트가 실제 구현된 서비스를 이용할 수 있도록 하는데, 클라이언트는 세부적인 구현은 몰라도 서비스를 이용할 수 있다. JDBC의 경우 mysql, postgreSql, oracle 등의 서비스 제공자들이 JDBC라는 하나의 서비스를 구성한다.<br>

## 서비스 제공자 프레임워크 구성

<br>

JDBC(Java Database Connectivity)의 경우 서비스 제공자 프레임워크에서의 제공자는 서비스 구현체이다.<br>
이 구현체들을 클라이언트에 제공하는 역할을 프레임워크가 통제하여, 클라이언트를 구현체로부터 분리해준다.<br>

<br>

- 서비스 인터페이스 : Connection <br>
- 제공자 등록 API : DriverManager.registerDrive() <br>
- 서비스 접근 API : DriverManager.getConnetion() <br>
- 서비스 제공자 인터페이스 : Driver

<br>

이를 활용하여 JDBC라는 서비스의 규칙을 지킨(Connection 인터페이스를 구현한 DB회사의 라이브러리등) 여러 DB를 사용 가능 하게된다. <br>

<br><br>


# 정적 팩터리 메서드의 단점

<br>

## 상속을 하려면 public이나 protected 생성자가 필요하니 정적 패터리 매서드만 제공하면 하위 클래스를 만들 수 없다.

<br>

앞서 이야기한 컬렉션 프레임워크의 유틸리티 구현 클래스들은 상속할 수 없다는 이야기다.<br>
어찌 보면 이 제약은 상속보다 컴포지션을 사용하도록 유도하고 불변 타입으로 만들려면 이 제약을 지켜야 한다는 점에서 오히려 장점으로 받아들일 수도 있다.<br>

<br>

## 정적 팩터리 메서드는 프로그래머가 찾기 어렵다.

<br>
생성자처럼 API설명에 명확히 드러나지 않으니 사용자는 정적 팩터리 메서드 방식 클래스를 인스턴스화할 방법을 알아내야 한다.<br>

<br><br>

# 정적 팩터리 명명 방식

![그림1](https://sehwan-choi.github.io/assets/img/review/effectiveJava/page1/subtitle/image2.jpg)

<br><br>

# 핵심정리

<br>

정적 팩터리 메서드와 public 생성자는 각자의 쓰임새가 있으니 상대적인 장단점을 이해하고 사용하는 것이 좋다.<br>
그렇다고 하더라도 정적 팩터리를 사용하는 게 유리한 경우가 더 많으므로 무작정 public 생성자를 제공하던 습관이 있다면 고치자.<br>

<br><br>

# 실무 사용경험

<br>

```java

public class main {

    public static void main(String[] args) {

        ProductRequest request1 = new ProductRequest(1L, "ITEM_1", "M-A-T-01CA", "POINT_PRODUCT");
        ProductRequest request2 = new ProductRequest(1L, "ITEM_1", "M-A-T-01CA", "CASH_PRODUCT");
    }
}

public class ProductRequest {

    private Long id;

    private String item_id;

    private String code;

    private String extraParam;

    public ProductRequest(Long id, String item_id, String code, String extraParam) {
        this.id = id;
        this.item_id = item_id;
        this.code = code;
        this.extraParam = extraParam;
    }
}
```

나는 상품을 가져오는 API호출 전에 DTO를 만들어야 했다. <br>
API 명세에 맞게 DTO를 만들고 새로운 ProductRequest객체를 만들어 파라미터를 넣어서 API호출을 한다.<br>

하지만 여기서 extraParam이 무엇을 뜻하는지 알겠는가? <br>

그냥 봐서는 모른다, 주석도 없기때문에 더욱이 알기 어렵다.<br>

사실 extraParam은 Point성 상품 요청인지, 현금성 상품 요청인지를 구분하는 값이다.(Static한 값이다)<br>

그러기 때문에 클라이언트가 실수로 잘못 넣게 된다면 자칫하면 큰 버그를 만들가능성이 있는 코드가 된다.<br>

그래서 아래와 같이 변경하였다.<br>

<br>

```java
public class main {

    public static void main(String[] args) {

        ProductRequest cashRequest = ProductRequest.createCashProductRequest(1L, "ITEM_1", "M-A-T-01CA");
        ProductRequest pointRequest = ProductRequest.createPointProductRequest(1L, "ITEM_1", "M-A-T-01CA");
    }
}

public class ProductRequest {

    private Long id;

    private String item_id;

    private String code;

    private String extraParam;
    
    private ProductRequest() {
    }

    public static ProductRequest createPointProductRequest(Long id, String item_id, String code) {
        ProductRequest productRequest = new ProductRequest();
        productRequest.id = id;
        productRequest.item_id = item_id;
        productRequest.code = code;
        productRequest.extraParam = "POINT_PRODUCT";
        return productRequest;
    }

    public static ProductRequest createCashProductRequest(Long id, String item_id, String code) {
        ProductRequest productRequest = new ProductRequest();
        productRequest.id = id;
        productRequest.item_id = item_id;
        productRequest.code = code;
        productRequest.extraParam = "CASH_PRODUCT";
        return productRequest;
    }
}
```

위와같이 수정하였다.(실제로는 Builder를 통해서 만들었다.)<br>

정적 팩터리 메소드를 이용하여 명확하게 Point성 상품 요청을 만들것인지, 현금성 상품 요청을 만들것인지 구분 하였다.<br>

이렇게 하면 클라이언트도 명확하게 사용할 수 있고, 버그도 발생하지 않을 것이다.