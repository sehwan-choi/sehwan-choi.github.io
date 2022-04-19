---
layout: post
title:  "Review Effective Java page3 - private 생성자나 열거 타입으로 싱글턴임을 보증하라"
subtitle:   "Review Effective Java page3 - private 생성자나 열거 타입으로 싱글턴임을 보증하라"
date:   2022-04-20 01:10:27 +0900
categories: review
tags: review effectivejava
comments: true
---

<br>

- 목차
    - [private 생성자나 열거 타입으로 생글턴임을 보증하라](#private-생성자나-열거-타입으로-싱글턴임을-보증하라)
    - [public static final 필드 방식의 싱글턴](#public-static-final-필드-방식의-싱글턴)
    - [정적 팩터리 방식의 싱글턴](#정적-팩터리-방식의-싱글턴)
    - [싱글턴 클래스의 직렬화](#싱글턴-클래스의-직렬화)
    - [열거 타입 방식의 싱글턴 - 바람직한 방법](#열거-타입-방식의-싱글턴---바람직한-방법)

<br>

# private 생성자나 열거 타입으로 싱글턴임을 보증하라

<br>

싱글턴이란 인스턴스를 오직 하나만 생성할 수 있는 클래스를 말한다.<br>
싱글턴의 전형적인 예로는 함수와 같은 무상태 객체나 설계상 유일해야 하는 시스템 컴포넌트를 들 수 있다.<br>
그런데 클래스를 싱글턴으로 만들면 이를 사용하는 클라이언트를 테스트하기가 어려워질 수 있다.<br>
타입을 인터페이스로 정의한 다음 그 인터페이스를 구현해서 만든 싱글턴이 아니라면 싱글턴 인스턴스를 가짜(mock) 구현으로 대체할 수 없기 때문이다.<br>
싱글턴을 만드는 방식은 보통 둘 중 하나다. 두 방식 모두 생성자는 private으로 감춰두고, 유일한 인스턴스에 접근할 수 있는 수단으로 public static 멤버를 하나 마련해둔다.<br><br>

## public static final 필드 방식의 싱글턴

<br>

```java
public class Elvis {

    public static final Elvis INSTANCE = new Elvis();

    private Elvis(){}
}

public class main {
    public static void main(String[] args) {
        Elvis elvis = Elvis.INSTANCE;
    }
}
```

private 생성자는 public static final 필드인 Elvis.INSTANCE를 초기화할 때 딱 한번 호출된다.<br>
public이나 protected생성자가 없으므로 Elvis클래스가 초기화될 때 만들어진 인스턴스가 전체 시스템에서 하나뿐임이 보장된다.<br>
예외로 리플렉션 API인 AccessibleObject.setAccessible을 사용해 private생성자를 호출할 수 있다.<br>
이러한 공격을 방어하려면 생성자를 수정하여 두 번째 객체가 생성되려 할 때 예외를 던지게 하면 된다.<br>
<br>

## 정적 팩터리 방식의 싱글턴

<br>

```java
public class Elvis2 {

    private static final Elvis2 INSTANCE = new Elvis2();

    private Elvis2() {}

    public static Elvis2 getInstance() {return INSTANCE;}
}

public class main {
    public static void main(String[] args) {
        Elvis2 elvis2 = Elvis2.getInstance();
    }
}
```

Elvis2.getInstance()는 항상 같은 객체의 참조를 반환하므로 제2의 Elvis2인스턴스란 결코 만들어지지 않는다.(하지만 리플렉션 API를 통한 예외를 똑같이 적용된다.)<br>
정적 팩터리 방식의 장점은 다음과 같다.<br>
1. 간결하다.<br>
2. API를 바꾸지 않고도 싱글턴이 아니게 변결할 수 있다. INSTANCE대신 다른 인스턴스를 반환 한다거나, 타입에 따라서 다른 인스턴스를 반환하게 할 수 있다.<br>
3. 정적 팩터리를 제네릭 싱글턴 팩터리로 만들 수 있다.<br>
4. 정적 팩터리의 메서드 참조를 공급자(supplier)로 사용할 수 있다는 점이다. getInstance를 Supplier<Elvis2>로 사용하는 방식이다.<br>

<br>

이러한 장점이 굳이 필요하지 않다면 public static final방식이 좋다.<br><br>

## 싱글턴 클래스의 직렬화

<br>

위의 둘 중 하나의 방식으로 만든 싱글턴 클래스를 직렬화하려면 단순히 Serializable을 구현한다고 선언하는 것만으로는 부족하다.<br>
모든 인스턴스 필드를 transient라고 선언하고 readResolve메서드를 제공해야 한다.<br>
이렇게 하지 않으면 직렬화된 인스턴스를 역직렬화할 때 마다 새로운 인스턴스가 만들어진다.<br>

<br>


## 열거 타입 방식의 싱글턴 - 바람직한 방법

<br>

```java
public enum EnumElvis {
    INSTANCE;

    public void leaveTheBuilding() {}
}

public class main {
    public static void main(String[] args) {
        EnumElvis enumElvis = EnumElvis.INSTANCE;
    }
}
```

public 필드 방식과 비슷하지만, 더 간결하고, 추가 노력 없이 직렬화할 수 있고, 심지어 아주 복잡한 직렬화 상황이나 리플렉션 공격에서도 제2의 인스턴스가 생기는 일을 완벽히 막아준다.<br>
조금 부자연스러워 보일수 있으나 대부분 상황에서는 원소가 하나뿐인 열거 타입이 싱글턴을 만드는 가장 좋은 방법이다.<br>

<br>

>단!<br>
만들려는 싱글턴이 Enum 외의 클래스를 상속해야 한다면 이 방법은 사용할 수 없다.<br>
하지만, 열거 타입이 다른 인터페이스를 구현하도록 선언할 수는 있다.<br>