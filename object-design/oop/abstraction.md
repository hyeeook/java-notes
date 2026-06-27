# 목차
1. [추상화가 왜 필요한가?](#1-추상화가-왜-필요한가)
2. [추상화를 구현하는 방법](#2-추상화를-구현하는-방법)
3. [interface vs abstract class — 언제 무엇을 쓸까?](#3-interface-vs-abstract-class---언제-무엇을-쓸까)

# 1. 추상화가 왜 필요한가?
## 추상화란?
복잡한 내부 구현은 감추고, "무엇을 할 수 있는가"만 외부에 드러내는 것.

## 추상화가 없을 때 발생하는 문제
예시 상황: 결제 수단이 구현 클래스에 직접 의존하는 경우

```java
public class Payment {
    private KakaoPay kakaoPay;  // 구현 클래스에 직접 의존
}

public class SubscriptionService {
    private KakaoPay kakaoPay;  // 구현 클래스에 직접 의존
}
```
위와 같은 코드는 나중에 결제 수단을 네이버페이로 바꾸면 KakaoPay를 참조하는 모든 클래스를 수정해야 함.<br>
참조하는 클래스가 많을수록 변경 영향도가 커지고, 수정 중 실수가 발생할 가능성도 높아짐.

## 추상화의 목적
1. 변경 영향도 감소(구현이 바뀌어도 사용하는 측은 수정 불필요)
2. 유연성 확보(구현체를 자유롭게 교체 가능)
3. 테스트 용이성(구현체 대신 Mock 객체로 교체 가능)
> "구현이 아닌 추상(interface)에 의존하라" — SOLID 원칙 중 DIP(의존 역전 원칙)와 연결되는 개념.

# 2. 추상화를 구현하는 방법
## interface 사용
```java
// "무엇을 할 수 있는가"만 정의
public interface PaymentMethod {
    void pay(int amount);
}

// 구현체 A
public class KakaoPay implements PaymentMethod {
    @Override
    public void pay(int amount) { ... }
}

// 구현체 B
public class NaverPay implements PaymentMethod {
    @Override
    public void pay(int amount) { ... }
}

// 사용하는 측: interface에만 의존
public class Payment {
    private PaymentMethod paymentMethod;

    public Payment(PaymentMethod paymentMethod) {
        this.paymentMethod = paymentMethod;
    }
}

// 결제 수단 바꿀 때 — Payment 손댈 필요 없음
new Payment(new KakaoPay());    // 카카오페이 쓸 때
new Payment(new NaverPay());    // 네이버페이로 바꿀 때
```

## abstract class 사용
```java
// 공통 필드·구현은 부모에, 차이점만 추상으로
public abstract class Animal {
    protected String name;
    protected int age;

    public Animal(String name, int age) {
        this.name = name;
        this.age = age;
    }

    // 공통 구현
    public String getName() {
        return name;
    }

    public abstract void speak();   // 자식마다 다른 것만 추상으로
}

public class Dog extends Animal {
    public Dog(String name, int age) {
        super(name, age);
    }

    @Override
    public void speak() {
        System.out.println(name + ": 멍멍!");
    }
}

public class Cat extends Animal {
    public Cat(String name, int age) {
        super(name, age);
    }

    @Override
    public void speak() {
        System.out.println(name + ": 야옹!");
    }
}
```

# 3. interface vs abstract class — 언제 무엇을 쓸까?
interface는 "~할 수 있다"는 능력/행위의 약속이고, abstract class는 "~이다"라는 계열을 표현함.

| | `interface` | `abstract class` |
|:---:|:---:|:---:|
| 표현 | "~할 수 있다" | "~이다" |
| 필드 | ❌ | ✅ |
| 일반 메서드 | ❌ (Java 8+ default 메서드 제외) | ✅ |
| 다중 구현/상속 | ✅ (여러 개 구현 가능) | ❌ (하나만 상속 가능) |
| 용도 | 능력/행위의 약속 | 공통 필드·구현을 공유하는 계열 |
| 예시 | `Flyable`, `Printable`, `PaymentMethod` | `Animal`, `Shape`, `BaseVehicle` |

> `Flyable`(날 수 있다) → interface: 새도 날고, 비행기도 날 수 있음. 공통 계열이 아님.<br>
> `Animal`(동물이다) → abstract class: 공통 필드(name, age)와 공통 구현(getName())을 공유하는 계열.