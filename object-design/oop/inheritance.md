# 목차
1. [상속이 왜 필요한가?](#1-상속이-왜-필요한가)
2. [상속이 위험한 이유](#2-상속이-위험한-이유)
3. [잘못된 상속 — is-a 위반](#3-잘못된-상속--is-a-위반)
4. [잘못된 상속 — 취약한 기반 클래스 문제](#4-잘못된-상속--취약한-기반-클래스-문제)
5. [메서드 오버라이딩 vs 필드 섀도잉](#5-메서드-오버라이딩-vs-필드-섀도잉)
6. [상속을 안전하게 설계하는 방법](#6-상속을-안전하게-설계하는-방법)

# 1. 상속이 왜 필요한가?
## 상속이란?
기존 클래스(부모)의 필드/메서드를 새 클래스(자식)가 물려받는 것. "is-a" 관계일 때 사용.

## 상속이 없을 때 발생하는 문제
예시 상황: Apple과 Pear가 각자 color 필드와 getColor()를 중복으로 가지는 경우
```java
public class Apple {
    private String color;

    public Apple() {
        this.color = "빨강";
    }

    public String getColor() {
        return color;
    }
}

public class Pear {
    private String color;

    public Pear() {
        this.color = "초록";
    }

    public String getColor() {
        return color;
    }
}
```
공통 속성(color)과 공통 메서드(getColor())가 클래스마다 중복됨. 과일 종류가 늘어날수록 동일한 코드가 계속 반복됨.

해결: 공통 부분을 부모 클래스로 추출
```java
public abstract class Fruit {
    protected String color;

    public Fruit(String color) {
        this.color = color;
    }

    public String getColor() {
        return color;
    }
}

public class Apple extends Fruit {
    public Apple() {
        super("빨강");
    }
}

public class Pear extends Fruit {
    public Pear() {
        super("초록");
    }
}
```

## 상속이 안전한 조건
1. 순수한 is-a 관계일 것 (Apple은 진짜로 Fruit다)
2. 부모 메서드끼리 서로 호출하며 얽혀 있지 않을 것 (self-use 최소화)
3. 부모가 상속을 염두에 두고 설계했을 것 (어떤 메서드를 오버라이딩해도 되는지 문서화)

Fruit의 getColor()는 다른 메서드를 호출하지도, 호출당하지도 않는 단순한 구조라 자식이 무엇을 해도 부모의 내부 흐름이 깨질 일이 없음.

# 2. 상속이 위험한 이유
상속은 부모의 "공개된 동작"뿐 아니라 "내부 구현 방식"에도 암묵적으로 의존하게 만듦.<br>
단순히 코드를 재사용하기 위한 목적으로 상속을 쓰면, 부모의 사소한 변경에도 자식이 예고 없이 깨질 수 있음.

> Effective Java Item 18: "상속보다는 컴포지션을 사용하라"<br>
> Effective Java Item 19: "상속을 고려해 설계하고 문서화하라. 그러지 않았다면 상속을 금지하라."

# 3. 잘못된 상속 — is-a 위반
문제: Car가 Engine을 상속받는 경우
```java
// ❌ is-a 위반: Car는 Engine이 아님
public class Car extends Engine {
}
```
"Car는 Engine이다"는 의미적으로 어색함. 또한 Car가 Engine의 모든 public 메서드를 그대로 물려받아 불필요한 책임이 새어 들어옴(캡슐화 위반).

해결: has-a 관계는 컴포지션(필드로 포함) 사용
```java
// ✅ has-a: Car는 Engine을 가짐
public class Car {
    private Engine engine;

    public void start() {
        engine.ignite();   // 필요한 기능만 선택적으로 위임
    }
}
```

# 4. 잘못된 상속 — 취약한 기반 클래스 문제
문제: 부모의 내부 구현에 의존하는 자식
```java
public class Counter {
    protected int count = 0;

    public void increment() {
        count++;
    }

    public void incrementTwice() {
        increment();
        increment();
    }
}

public class LoggingCounter extends Counter {
    private int callCount = 0;

    @Override
    public void increment() {
        callCount++;
        super.increment();
    }

    public int getCallCount() {
        return callCount;
    }
}
```
```java
LoggingCounter counter = new LoggingCounter();
counter.incrementTwice();
System.out.println(counter.getCallCount());   // 2
```
incrementTwice()는 부모에 정의되어 있지만, 내부에서 호출하는 increment()는 동적 바인딩에 의해 자식의 오버라이딩된 버전이 호출됨. 따라서 callCount도 2가 됨.

문제: 부모가 incrementTwice()를 리팩토링하면?
```java
public void incrementTwice() {
    count += 2;   // increment() 호출 없이 직접 처리
}
```
```java
counter.incrementTwice();
System.out.println(counter.getCallCount());   // 0 (의도와 다르게 동작)
```
LoggingCounter는 코드를 건드리지 않았지만, count는 정확히 증가해도 callCount는 더 이상 의도대로 동작하지 않음. 컴파일 에러도 없이 조용히 깨짐.

# 5. 메서드 오버라이딩 vs 필드 섀도잉
메서드는 동적 바인딩(실제 객체 타입 기준), 필드는 정적 바인딩(변수의 선언된 타입 기준)으로 동작함.

```java
public class Animal {
    protected String name = "동물";

    public void makeSound() {
        System.out.println("동물 소리");
    }
}

public class Dog extends Animal {
    protected String name = "강아지";   // 섀도잉: 부모와 별개의 필드

    @Override
    public void makeSound() {
        System.out.println("멍멍");   // 오버라이딩: 부모 메서드를 덮어씀
    }
}
```
```java
Animal a = new Dog();
System.out.println(a.name);   // "동물" — 필드는 변수 타입(Animal) 기준
a.makeSound();                 // "멍멍" — 메서드는 실제 객체 타입(Dog) 기준
```

`super.method()`는 오버라이딩을 무시하고 부모의 원본 메서드를 직접 호출함.<br>
`super.field`는 동일한 이름이라도 부모의 별개 필드에 접근함(필드는 섀도잉되므로 자식 안에 두 개의 필드가 동시에 존재).

# 6. 상속을 안전하게 설계하는 방법

## (1) 생성자에서 재정의 가능한 메서드 호출 금지
문제: 생성자에서 오버라이딩 가능한 메서드를 호출하는 경우
```java
public class Parent {
    public Parent() {
        init();   // 생성자에서 오버라이딩 가능한 메서드 호출
    }

    protected void init() {
        System.out.println("Parent init");
    }
}

public class Child extends Parent {
    private String value = "초기값";

    @Override
    protected void init() {
        System.out.println("Child init: " + value);
    }
}
```
```java
new Child();   // 출력: "Child init: null"
```
객체 생성 순서는 `super()` 실행 → 자식 필드 초기화 → 자식 생성자 본문 순서임.<br>
`Parent()`가 `init()`을 호출하는 시점엔 동적 바인딩에 의해 `Child.init()`이 실행되지만, 이때 `Child.value`는 아직 초기화 전이라 `null`이 출력됨.<br>
컴파일 에러도, 런타임 예외도 없이 조용히 발생하는 버그라 디버깅이 까다로움.

> Effective Java: "생성자에서는 재정의 가능한 메서드를 호출하지 마라"

## (2) protected 필드 최소화
`protected` 필드는 자식이 부모의 내부 구현에 직접 접근하게 만드는 강한 결합임.<br>
나중에 필드명을 바꾸거나 제거하면 자식 클래스가 컴파일조차 되지 않을 수 있음.

```java
// ❌ protected 필드 직접 노출
public class Counter {
    protected int count = 0;
}

// ✅ private + protected 메서드로 노출
public class Counter {
    private int count = 0;

    protected int getCount() {
        return count;
    }

    protected void setCount(int count) {
        this.count = count;
    }
}
```
메서드로 노출하면 내부 자료형이나 검증 로직이 바뀌어도 시그니처만 유지되면 자식에 영향이 가지 않음.

## (3) @Override의 역할
`@Override`는 없어도 컴파일되지만, 부모 메서드 시그니처를 잘못 베껴 쓰는 실수를 컴파일 타임에 잡아주는 안전장치 역할을 함.

문제: @Override 없이 시그니처를 잘못 적은 경우
```java
public class Parent {
    public void process(String input) { ... }
}

public class Child extends Parent {
    public void process(int input) { ... }   // @Override 없음, 오버로딩이 되어버림
}
```
```java
Parent p = new Child();
p.process("hello");   // Child.process(int)가 아니라 Parent.process(String) 호출됨
```
의도는 재정의였지만, 자바는 오버로딩으로 인식하여 새로운 메서드가 하나 추가된 것뿐임.<br>
`@Override`를 붙였다면 "부모에 해당 시그니처가 없다"는 컴파일 에러로 즉시 발견 가능함.