# 목차
1. [불변 클래스가 왜 필요한가?](#1-불변-클래스가-왜-필요한가)
2. [불변 클래스의 조건](#2-불변-클래스의-조건)
3. [불변성을 위협하는 문제와 해결책](#3-불변성을-위협하는-문제와-해결책)

# 1. 불변 클래스가 왜 필요한가?
## 불변 클래스란?
객체의 상태가 변하지 않는 클래스.
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

# 3. 불변성을 위협하는 문제와 해결책
1. 상속
예시 문제 상황: Money가 상속이 가능한 클래스인 경우(클래스에 final 미적용)<br>
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