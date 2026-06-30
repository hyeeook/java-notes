# 목차
1. [다형성이란?](#1-다형성이란)
2. [다형성이 없을 때 발생하는 문제](#2-다형성이-없을-때-발생하는-문제)
3. [오버라이딩 vs 오버로딩](#3-오버라이딩-vs-오버로딩)
4. [오버로딩의 함정 — null 전달](#4-오버로딩의-함정--null-전달)
5. [다운캐스팅과 instanceof](#5-다운캐스팅과-instanceof)

# 1. 다형성이란?
## 개념
같은 타입(인터페이스/부모 클래스)으로 다양한 객체를 다루고, 실제 동작은 각 객체가 결정하는 것.

다형성은 두 가지가 합쳐져서 성립함.
1. 업캐스팅: 부모 타입 변수로 자식 객체를 다룰 수 있음
2. 동적 바인딩: 실제로 호출되는 메서드는 객체의 실제 타입 기준으로 결정됨

```java
Animal a = new Dog();   // 업캐스팅
a.makeSound();           // 동적 바인딩: 선언 타입(Animal)이 아닌 실제 타입(Dog) 기준으로 호출됨 → "멍멍"
```

> 추상화가 "공통 인터페이스/계약"을 만들고, 다형성은 그 계약을 통해 서로 다른 구현체를 동일하게 다루는 것.

# 2. 다형성이 없을 때 발생하는 문제
예시 상황: instanceof로 타입을 분기하는 경우
```java
public void makeSound(Object animal) {
    if (animal instanceof Dog) {
        System.out.println("멍멍");
    } else if (animal instanceof Cat) {
        System.out.println("야옹");
    }
    // 새로운 동물이 추가될 때마다 if-else 추가 필요
}
```
새로운 타입이 추가될 때마다 관련된 모든 메서드를 찾아 수정해야 함. OCP(개방-폐쇄 원칙) 위반.

해결: 다형성 적용
```java
public abstract class Animal {
    public abstract void makeSound();
}

public class Dog extends Animal {
    @Override
    public void makeSound() { System.out.println("멍멍"); }
}

public class Cat extends Animal {
    @Override
    public void makeSound() { System.out.println("야옹"); }
}

public void printSound(Animal animal) {
    animal.makeSound();   // 새 동물이 추가돼도 이 코드는 수정 불필요
}
```

# 3. 오버라이딩 vs 오버로딩
| | 오버로딩 (Overloading) | 오버라이딩 (Overriding) |
|:---:|:---:|:---:|
| 결정 시점 | 컴파일 타임 | 런타임 |
| 결정 기준 | 매개변수의 타입/개수 | 실제 객체의 타입 |
| 다른 이름 | 정적 다형성 | 동적 다형성 |
| 관계 | 같은 클래스 내 여러 메서드 | 부모-자식 관계 |

```java
public class Calculator {
    public int add(int a, int b) { return a + b; }
    public double add(double a, double b) { return a + b; }
    public int add(int a, int b, int c) { return a + b + c; }
}

calculator.add(1, 2);   // 컴파일러가 매개변수 타입/개수를 보고 컴파일 타임에 결정
```

오버로딩의 구별 기준은 메서드 시그니처(이름 + 매개변수의 개수·타입·순서)이며, 반환 타입은 시그니처에 포함되지 않음.
```java
public int add(int a, int b) { return a + b; }
public double add(int a, int b) { return a + b; }   // ❌ 컴파일 에러: 매개변수가 동일, 반환 타입만 다름
```

# 4. 오버로딩의 함정 — null 전달
형제 관계가 아닌 타입끼리는 더 구체적인 타입이 우선 선택됨.
```java
public class Printer {
    public void print(String s) { System.out.println("String: " + s); }
    public void print(Object o) { System.out.println("Object: " + o); }
}

printer.print(null);   // "String" 호출됨 — String이 Object보다 더 구체적인 타입
```

형제 관계의 타입이 후보로 있으면 컴파일러가 우열을 가리지 못해 모호성(Ambiguity) 컴파일 에러가 발생함.
```java
public void print(String s) { ... }
public void print(Integer i) { ... }

printer.print(null);   // 컴파일 에러: reference to print is ambiguous
```

해결: 캐스팅으로 의도를 명확히 함
```java
printer.print((String) null);
printer.print((Integer) null);
```

# 5. 다운캐스팅과 instanceof
부모 타입으로 다루던 객체에서 자식 타입만의 메서드를 호출하려면 다운캐스팅이 필요함.
```java
List<Animal> animals = new ArrayList<>();
animals.add(new Dog());
animals.add(new Cat());

for (Animal a : animals) {
    if (a instanceof Dog) {
        Dog dog = (Dog) a;   // 다운캐스팅
        dog.fetch();
    }
}
```

문제: instanceof 체크 없이 강제 캐스팅하는 경우
```java
Animal a = new Cat();
Dog dog = (Dog) a;   // ClassCastException 발생 (런타임 예외)
```
컴파일러는 문법적으로 가능한 캐스팅인지만 확인하고, 실제 객체 타입은 런타임에만 알 수 있어 컴파일 에러로는 잡히지 않음.

> 다운캐스팅 전엔 항상 instanceof로 먼저 확인해야 함.

```java
// Java 16+ 패턴 매칭 (instanceof)
if (a instanceof Dog dog) {
    dog.fetch();   // 캐스팅 없이 바로 사용 가능
}
```

## 다운캐스팅 남발은 나쁜 설계 신호
다운캐스팅과 instanceof를 자주 쓰게 되면, 다형성이 해결해준 "새 타입 추가 시 if-else 반복" 문제가 그대로 돌아온 것임. 이는 클래스 설계(추상화)가 잘못됐다는 신호임.

```java
// ❌ 다운캐스팅 남발
for (Animal a : animals) {
    if (a instanceof Dog) {
        ((Dog) a).fetch();
    } else if (a instanceof Cat) {
        ((Cat) a).scratch();
    }
}
```

해결: 공통 추상 메서드로 끌어올리기
```java
public abstract class Animal {
    public abstract void makeSound();
    public abstract void doSpecialAction();   // 공통 추상 메서드로 끌어올림
}

public class Dog extends Animal {
    @Override
    public void doSpecialAction() { fetch(); }
    private void fetch() { ... }
}

public class Cat extends Animal {
    @Override
    public void doSpecialAction() { scratch(); }
    private void scratch() { ... }
}

// 사용하는 쪽 — 다운캐스팅 불필요
for (Animal a : animals) {
    a.doSpecialAction();   // 다형성 그대로 유지
}
```