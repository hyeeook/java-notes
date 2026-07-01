# 목차
1. [Entity와 Value Object가 왜 필요한가?](#1-entity와-value-object가-왜-필요한가)
2. [Entity와 Value Object를 구분하는 기준](#2-entity와-value-object를-구분하는-기준)
3. [Entity와 Value Object를 구현하는 방법](#3-entity와-value-object를-구현하는-방법)
4. [판단 기준 정리](#4-판단-기준-정리)

# 1. Entity와 Value Object가 왜 필요한가?
## Entity와 Value Object란?
Entity는 식별자(identity)로 구별되는 객체, Value Object는 속성(값) 자체로 구별되는 객체.

## 구분하지 않을 때 발생하는 문제
예시 상황: Money(값)와 Account(계좌)를 구분 없이 모두 가변으로 설계
```java
class Money {
    private int amount;

    Money(int amount) {
        this.amount = amount;
    }

    public void setAmount(int amount) {
        this.amount = amount;
    }
}

// 사용하는 측
Money price = new Money(1000);
order1.setPrice(price);
order2.setPrice(price);   // 같은 Money 인스턴스를 공유

price.setAmount(0);       // order1, order2의 가격이 동시에 0으로 바뀜
```
Money는 "1000원"이라는 값 자체가 의미를 가지는 객체인데 가변으로 설계하면, 여러 곳에서 참조를 공유할 때 한쪽의 변경이 다른 쪽에 의도치 않게 전파됨.<br>
반대로 계좌처럼 식별자를 가지고 생명주기 동안 잔액이 계속 바뀌는 객체를 불변으로 설계하면, 잔액이 바뀔 때마다 새 객체를 만들어야 해서 비효율적이고 "같은 계좌"라는 개념도 표현하기 어려움.

## Entity와 Value Object를 구분하는 목적
1. 객체의 본질(식별자 기반 vs 값 기반)에 맞는 동등성(equals/hashCode) 정의
2. 값 자체로 의미를 가지는 객체는 불변으로 만들어 공유 시 부작용 방지
3. 식별자로 추적되는 객체는 가변을 허용해 불필요한 객체 생성 방지
> Value Object를 불변으로 만드는 이유는 immutable-class.md 참고.

# 2. Entity와 Value Object를 구분하는 기준
## 기준 (1) - 식별자가 있는가?
- 있다 → Entity (예: 계좌번호, 회원 ID, 주문 번호)
- 없다 → Value Object (예: 금액, 좌표, 기간)

## 기준 (2) - "같다"의 의미가 식별자 기준인가, 속성 기준인가?
- Entity: 속성이 모두 같아도 식별자가 다르면 다른 객체 (동명이인은 다른 사람)
- Value Object: 식별자 없이 속성이 같으면 같은 객체 (만원짜리 지폐 두 장은 서로 바꿔도 무관)

| | Entity | Value Object |
|:---:|:---:|:---:|
| 동등성 비교 기준 | 식별자(id) | 모든 속성 |
| 가변/불변 | 가변 (상태가 바뀌어도 같은 객체) | 불변 (속성이 바뀌면 다른 객체) |
| 예시 | 회원, 계좌, 주문 | 금액, 주소, 좌표, 기간 |
> 💡 Entity를 정의하는 본질은 "식별자 기반 동등성"이지 가변성이 아니다. 가변은 Entity에서 흔한 특성일 뿐 필수는 아니며(불변 Entity도 가능), 반대로 Value Object는 불변이 사실상 규칙이다.

# 3. Entity와 Value Object를 구현하는 방법
## Entity - 식별자 기반 equals/hashCode, 상태는 가변 허용
```java
class Account {
    private final String accountNumber; // 식별자는 불변
    private int balance;                // 상태는 가변

    Account(String accountNumber, int balance) {
        this.accountNumber = accountNumber;
        this.balance = balance;
    }

    public void deposit(int amount) {
        this.balance += amount;
    }

    @Override
    public boolean equals(Object o) {
        if (!(o instanceof Account)) return false;
        Account other = (Account) o;
        return this.accountNumber.equals(other.accountNumber); // 식별자만 비교
    }

    @Override
    public int hashCode() {
        return accountNumber.hashCode();
    }
}
```

## Value Object - 모든 속성 기반 equals/hashCode, 불변 강제
```java
final class Money {
    private final int amount;

    Money(int amount) {
        this.amount = amount;
    }

    public Money add(Money other) {
        return new Money(this.amount + other.amount); // 상태 변경 대신 새 객체 반환
    }

    @Override
    public boolean equals(Object o) {
        if (!(o instanceof Money)) return false;
        Money other = (Money) o;
        return this.amount == other.amount; // 모든 속성 비교
    }

    @Override
    public int hashCode() {
        return Objects.hash(amount);
    }
}
```
> Value Object를 불변 클래스의 조건에 맞게 구현하는 자세한 방법은 immutable-class.md 참고.

> 💡 위 Money는 record로 한 줄로 대체할 수 있다: `public record Money(int amount) { }`.<br>
> record는 모든 필드를 `private final`로 만들고 값 기반 `equals`/`hashCode`를 자동 생성해, Value Object의 "속성 기반 동등성 + 불변" 요건에 그대로 들어맞는다. 반면 Entity는 식별자 하나만으로 동등성을 정의해야 하는데 record의 자동 equals는 모든 필드를 비교하므로, Entity를 record로 만들 때는 주의가 필요하다.

# 4. 판단 기준 정리
새로운 클래스를 설계할 때 아래 질문으로 Entity/Value Object를 판단한다.

| 질문 | Entity | Value Object |
|:---:|:---:|:---:|
| 식별자가 필요한가? | ✅ | ❌ |
| 속성이 같아도 다른 객체일 수 있는가? | ✅ | ❌ |
| 생명주기 동안 상태가 바뀌는가? | ✅ (같은 식별자 유지) | ❌ (바뀌면 새 객체) |
