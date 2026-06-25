# 목차
1. [불변 클래스가 왜 필요한가?](#1-불변-클래스가-왜-필요한가)
2. [불변 클래스의 조건](#2-불변-클래스의-조건)
3. [불변성을 위협하는 문제와 해결책](#3-불변성을-위협하는-문제와-해결책)

# 1. 불변 클래스가 왜 필요한가?
## 불변 클래스란?
객체의 상태(인스턴스 필드)가 변하지 않는 클래스.
## 불변 클래스가 없을 때 발생하는 문제
예시 상황: Money가 불변 클래스가 아닌 경우
```java
Money price = new Money(1000);  // 가격 1,000원

discount.apply(price);  // 개발자가 실수로 apply() 메서드 내부에서 price.setValue(0) 호출하도록 코드를 잘못 작성
payment.charge(price);  // 0원으로 결제됨
receipt.save(price);    // 0원으로 기록됨
```
위와 같은 코드는 컴파일 시점에 발견할 수 없고, 런타임에 가서야 버그를 발견할 수 있음.<br>
런타임 시점에 발생한 버그는 어디서 값이 바뀌었는지 추적이 불가하고, price를 참조하는 모든 곳을 뒤져야 함.
## 불변 클래스의 장점
1. 값 오염 방지
2. 변경 수단(setter 등)이 없어 잘못된 코드 작성 시 컴파일 시점에 즉시 발견 가능

# 2. 불변 클래스의 조건
1. 모든 인스턴스 필드는 final
2. 모든 인스턴스 필드는 private(캡슐화를 위함, 불변성의 핵심은 final)
3. 인스턴스 필드의 setter 제거
4. 가변 객체를 인스턴스 필드로 사용해야 한다면 방어적 복사 사용

보충설명
> static 필드는 클래스에 속하므로 불변 클래스 조건에 포함되지 않지만, static 필드가 변경되면 인스턴스 필드가 바뀌지 않아도 객체의 동작이 달라질 수 있어 불변의 약속이 깨질 수 있다. 완전한 불변을 원한다면 static 필드도 final로 관리하는 것이 좋다.
```java
class Money {
    static int exchangeRate = 1300; // final 아님
    private final int amount;

    Money(int amount) {
        this.amount = amount;
    }

    int toUSD() {
        return amount / exchangeRate;   // Money.exchangeRate = 0 으로 변경되면 ArithmeticException 발생(객체의 동작이 바뀜)
    }
}
```

# 3. 불변성을 위협하는 문제와 해결책
상속<br>

문제: Money가 상속이 가능한 클래스인 경우(클래스에 final 미적용)<br>
부모 클래스는 불변이지만 자식 클래스가 새로운 필드/메서드를 추가하면 불변의 약속이 깨짐.
```java
// 부모
class Money {
    private final int amount;

    Money(int amount) {
        this.amount = amount;
    }

    public int getAmount() {
        return this.amount;
    }
}

// 자손
class MutableMoney extends Money {
    private int amount; // 부모의 amount 필드를 자손의 amount 필드로 가림

    MutableMoney(int amount) {
        super(amount);
        this.amount = amount;
    }

    public void setValue(int amount) {
        this.amount = amount;   // 변경 가능
    }

    @Override
    public int getAmount() {
        return this.amount; // 부모의 필드가 아닌 자신의 필드를 반환
    }
}

// 클라이언트
Money price = new MutableMoney(1000);   // 다형성으로 부모 타입으로 받음
((MutableMoney)price).setValue(0);      // 불변이 깨짐
```
해결: Money가 상속이 불가한 클래스인 경우(클래스에 final 적용)<br>
```java
// 방법 (1): 상속 자체 막기
final class Money {
    private final int amount;

    Money(int amount) {
        this.amount = amount;
    }

    public int getAmount() {
        return this.amount;
    }
}

// 방법 (2): 정적 팩터리 메서드 사용
// final class보다 유연하게 객체 생성 로직을 추가 가능
// ex. 캐싱, 유효성 검사, 하위 타입 반환 등
class Money {
    private final int amount;

    // 자손 생성자의 첫 줄에 반드시 부모 생성자 호출 필요
    // 여기서 부모 생성자가 private이므로 자손에서 호출 불가(컴파일 에러) => 상속 막음
    private Money(int amount) {
        this.amount = amount;
    }

    public static Money of(int amount) {
        // 캐싱, 유효성 검사, 하위 타입 반환 등 객체 생성 로직을 추가 가능
        return new Money(amount);
    }

    public int getAmount() {
        return this.amount;
    }
}
```