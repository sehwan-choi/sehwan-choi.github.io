---
layout: post
title:  "Review Effective Java page2 - 생성자에 매개변수가 많다면 빌더를 고려하라"
subtitle:   "Review Effective Java page2 - 생성자에 매개변수가 많다면 빌더를 고려하라"
date:   2022-04-19 23:49:27 +0900
categories: review
tags: review effectivejava
comments: true
---

<br>

- 목차
    - [생성자에 매개변수가 많다면 빌더를 고려하라.](#생성자에-매개변수가-많다면-빌더를-고려하라)
        - [점층적 생성자 패턴](#점층적-생성자-패턴)
        - [자바빈즈 패턴](#자바빈즈-패턴)
        - [빌더 패턴](#빌더-패턴)
    - [핵심 정리](#핵심-정리)

<br>

# 생성자에 매개변수가 많다면 빌더를 고려하라.

<br><br>

## 점층적 생성자 패턴

<br>

정적 팩터리와 생성자에는 똑같은 제약이 하나 있다.<br>
선택적 매개변수가 많을때 적절히 대응하기 어렵다는 점이다.<br>
식품 포장의 영양정보를 표현하는 클래스를 생각해보자.<br>
영양정보는 1회 내용량, 총 n회 제공량, 1회 제공량당 칼로리 같은 필수 항목 몇개와 총 지방, 트랜스지방, 포화지방, 콜레스테롤, 나트륨등 총 20개가 넘는 선택 항목으로 이뤄진다.<br>
그런데 대부분 제품은 이 선택 항목중 대다수의 값이 0이다.<br>

```java
public class NutritionFacts {
    private final int servingSize;  // 필수
    private final int servings;     // 필수
    private final int calories;     // 선택
    private final int fat;          // 선택
    private final int sodium;       // 선택
    private final int carbohydrate; // 선택

    public NutritionFacts(int servingSize, int servings) {
        this(servingSize, servings, 0);
    }

    public NutritionFacts(int servingSize, int servings, int calories) {
        this(servingSize,servings,calories,0);
    }

    public NutritionFacts(int servingSize, int servings, int calories, int fat) {
        this(servingSize,servings,calories,fat,0);
    }

    public NutritionFacts(int servingSize, int servings, int calories, int fat, int sodium) {
        this(servingSize,servings,calories,fat,sodium,0);
    }

    public NutritionFacts(int servingSize, int servings, int calories, int fat, int sodium, int carbohydrate) {
        this.servingSize = servingSize;
        this.servings = servings;
        this.calories = calories;
        this.fat = fat;
        this.sodium = sodium;
        this.carbohydrate = carbohydrate;
    }
}
```

위와 같이 점층적 생성자 패턴, 필수 매개변수만 받는 생성자, 필수 매개변수와 선택 매개변수 1개를 받는 생성자... 형태로 선택 매개변수를 전부 다 받는 생성자까지 늘려가는 방식이다.<br>
이 클래스의 인스턴스를 만들려면 원하는 매개변수를 모두 포함한 생성자 중 가장 짧은 것을 골라 호출하면 된다.

```java
NutritionFacts cocaCola = new NutritionFacts(240,8,100,0,35,27);
```

<br>
위와같은 생성자는 사용자가 설정하길 원치 않는 매개변수까지 포함하기 쉬운데, 어쩔 수 없이, 그런 매개변수에도 값을 지정해줘야 한다.<br>
즉, 매개변수가 많아지면 클라이언트 코드를 작성하거나 읽기 어려워진다.<br>
클라이언트가 실수로 매개변수의 순서를 바꿔 건네줘도 컴파일러는 알아채지 못하고, 결국 런타임에 엉뚱한 동작을 하게 된다. 이는 곧 버그로 이어진다.<br>

<br><br>

## 자바빈즈 패턴

<br>

```java
public class NutritionFacts2 {
    private int servingSize;  // 필수
    private int servings;     // 필수
    private int calories;     // 선택
    private int fat;          // 선택
    private int sodium;       // 선택
    private int carbohydrate; // 선택

    public void setServingSize(int servingSize) {
        this.servingSize = servingSize;
    }

    public void setServings(int servings) {
        this.servings = servings;
    }

    public void setCalories(int calories) {
        this.calories = calories;
    }

    public void setFat(int fat) {
        this.fat = fat;
    }

    public void setSodium(int sodium) {
        this.sodium = sodium;
    }

    public void setCarbohydrate(int carbohydrate) {
        this.carbohydrate = carbohydrate;
    }
}

public class main {

    public static void main(String[] args) {
        NutritionFacts2 facts2 = new NutritionFacts2();
        facts2.setServingSize(240);
        facts2.setServings(8);
        facts2.setCalories(100);
        facts2.setSodium(35);
        facts2.setCarbohydrate(27);
    }
}

```

위와 같이 자바빈즈 패턴이랑 매개변수가 없는 생성자로 객체를 만든 후, 세터 메서드들을 통해 원하는 매개변수의 값을 설정하는 방식이다.<br>
하지만 자바빈즈 패턴도 심각한 단점을 가지고 있다.<br>
자바빈즈 패턴에서는 객체 하나를 만들려면 메서드를 여러 개 호출해야 하고, 객체가 완전히 생성되기 전까지는 일관성이 무너진 상태에 놓이게 된다.<br>
즉, 전부 set되기 전까지는 사용할 수 없는 상태가 된다.<br>
점층적 생성자 패턴에서는 매개변수들이 유효한지를 생성자에서만 확인하면 일관성을 유지할 수 있었는데, 그 장치가 완전히 사라진 것이다.<br>

<br><br>

## 빌더 패턴

<br>

빌더 패턴이란 클라이언트는 필요한 객체를 직접 만드는 대신, 필수 매개변수만으로 생성자를 호출해 빌더 객체를 얻는다. <br>
그런 다음 빌더 객체가 제공하는 일종의 세터 메서드들로 원하는 선택 매개변수들을 설정한다.<br>
마지막으로 매개변수가 없는 build 메서드를 호출해 드디어 우리에게 필요한 객체를 얻는다.<br>
빌더의 세터 메서드들은 빌더 자신을 반환하기 때문에 연쇄적으로 호출 할 수있다.<br>
이런 방식을 메서드 호출이 흐르듯 연결된다는 뜻으로 플루언트 API 혹은 메서드 연쇄 라고 한다.<br>

```java
public class UserInfo {

    private String name;
    private int age;
    private int zipCode;
    private String address;
    private char sex;
    private String birthDay;

    public static class Builder {

        private String name;
        private int age;
        private int zipCode;
        private String address;
        private char sex;
        private String birthDay;

        public Builder() {}

        public Builder name(String name) {this.name = name; return this;}
        public Builder age(int age) {this.age = age; return this;}
        public Builder zipCode(int zipCode) {this.zipCode = zipCode; return this;}
        public Builder address(String address) {this.address = address; return this;}
        public Builder sex(char sex) {this.sex = sex; return this;}
        public Builder birthDay(String birthDay) {this.birthDay = birthDay; return this;}

        public UserInfo build() {return new UserInfo(this);}
    }

    private UserInfo(Builder builder) {
        this.name = builder.name;
        this.age = builder.age;
        this.zipCode = builder.zipCode;
        this.address = builder.address;
        this.sex = builder.sex;
        this.birthDay = builder.birthDay;
    }
}

public class main {

    public static void main(String[] args) {
        UserInfo userInfo = new UserInfo.Builder()
                .name("hong")
                .address("seoul")
                .age(20)
                .birthDay("2000-01-01")
                .sex('M')
                .zipCode(11111)
                .build();
    }
}
```

위와 같이 빌드패턴을 사용하면 코드는 쓰기 쉽고, 무엇보다도 읽기 쉽다.<br><br>

또한 빌더 패턴은 계층적으로 설계된 클래스와 함께 쓰기 좋다.<br>

```java
public abstract class Person {

    private String name;
    private int age;
    private String birthDay;

    abstract static class Builder<T extends Builder<T>> {

        private String name;
        private int age;
        private String birthDay;

        public Builder() {}

        public T name(String name) {this.name = name; return self();}   //  T형으로 반환하여 하위 클래스의 빌더를 반환 하도록 한다.
        public T age(int age) {this.age = age; return self();}
        public T birthDay(String birthDay) {this.birthDay = birthDay; return self();}

        abstract public Person build(); //  하위 클래스에서 재정의 하여 실제 객체를 반환하도록 한다.
        protected abstract T self();    //  하위 클래스에서 재정의 하여 하위 클래스의 빌더를 반환 하도록 한다.
    }

    protected Person(Builder<?> builder) {
        this.name = builder.name;
        this.age = builder.age;
        this.birthDay = builder.birthDay;
    }
}

public class Student extends Person{

    private String name;
    private int age;
    private String birthDay;
    private String curriculum;
    private int grade;

    public static class Builder extends Person.Builder<Builder> {

        private String curriculum;
        private int grade;

        public Builder() {}

        public Builder curriculum(String curriculum) {this.curriculum = curriculum; return this;}
        public Builder grade(int grade) {this.grade = grade; return this;}

        @Override
        public Student build() {return new Student(this);}

        @Override
        protected Builder self() {return this;}
    }

    private Student(Builder builder) {
        super(builder);
        this.curriculum = builder.curriculum;
        this.grade = builder.grade;
    }
}

public class Teacher extends Person {

    private int zipCode;
    private String address;

    static class Builder extends Person.Builder<Builder> {

        private int zipCode;
        private String address;

        public Builder() {}

        public Builder zipCode(int zipCode) {this.zipCode = zipCode; return self();}
        public Builder address(String address) {this.address = address; return self();}

        @Override
        public Teacher build() { return new Teacher(this);};

        @Override
        protected Builder self(){return this;}
    }

    private Teacher(Builder builder) {
        super(builder);
        this.zipCode = builder.zipCode;
        this.address = builder.address;
    }
}

public class main {

    public static void main(String[] args) {

        Student student = new Student.Builder()
                .curriculum("high school")
                .grade(1)
                .age(17)
                .birthDay("2000-01-01")
                .name("choi")
                .build();

        Teacher teacher = new Teacher.Builder()
                .address("seoul")
                .age(29)
                .birthDay("1990-01-01")
                .name("park")
                .build();
    }
}
```

위와 같이 빌더 패턴은 상당히 유연한다. 빌더 하나로 여러 객체를 순회하면서 만들 수있고, 빌더에 넘기는 매개변수에 따라 다른 객체를 만들 수도 있다.<br>
하지만 빌더 패턴도 장점만 있는 것은 아니다. 객체를 만들려면, 그에 앞서 빌더부터 만들어야 한다. 빌더 생성 비용이 크지는 않지만 성능에 민감한 상황에서는 문제가 될 수 있다.<br><br>


> 참고<br>
현재의 Spring에서는 Lombok의 어노테이션중 @Builder를 통해서 쉽게 빌더패턴을 사용할수 있다.

<br><br>

# 핵심 정리

<br>

생성자나 정적 팩터리가 처리해야 할 매개변수가 많다면 빌더 패턴을 선택하는 게 더 낫다.<br>
매개변수 중 다수가 필수가 아니거나 같은 타입이면 특히 더 그렇다.<br>
빌더는 점층적 생성자보다 클라이언트 코드를 읽고 쓰기가 훨씬 간결하고, 자바빈즈보다 훨씬 안전하다.<br>
