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
1. 모든 인스턴스 필드는 final(재할당 방지)
    > 💡 final의 또 다른 효과: 안전 발행(safe publication)
    > final 필드는 생성자가 끝나는 시점에 그 값이 다른 스레드에서도 완성된 상태로 보이는 것이 보장된다(JMM). 덕분에 불변 객체는 별도의 동기화(synchronized, volatile) 없이도 여러 스레드가 안전하게 공유할 수 있다.
2. 모든 인스턴스 필드는 private(캡슐화)
3. 가변 객체를 인스턴스 필드로 사용해야 한다면 방어적 복사 사용
    > 방어적 복사는 defensive-copy.md 참고.
    
    > ⚠️ final은 재할당만 막을 뿐, 가변 객체의 내부 상태 변경은 막지 못한다. 따라서 가변 객체 필드는 final + 방어적 복사가 함께 있어야 불변이 완성된다.
4. 인스턴스 필드의 setter 등 상태를 변경하는 메서드 제거
    > 🔍 "상태를 변경하는 메서드"의 여러 형태<br>
    > setter는 가장 노골적인 형태일 뿐, 이름만 다른 변경 메서드나 가변 객체의 내부 변경도 같은 부류다.
    ```java
    public final class Account {
        private final int balance;
        private final List<String> history;

        public Account(int balance, List<String> history) {
            this.balance = balance;
            this.history = new ArrayList<>(history);
        }

        // ❌ 1. 전형적인 setter — final 필드라 이건 컴파일 자체가 안 됨
        //public void setBalance(int balance) {
        //    this.balance = balance;
        //}

        // ❌ 2. 가변 객체 필드의 내부 변경 — 재할당이 아니라 final이 못 막음(가장 위험)
        public void addHistory(String record) {
            this.history.add(record);   // 컴파일 통과! 내부 상태가 바뀜
        }
    }
    ```

> 💡 record(자바 16+): 위 조건 대부분을 자동으로 충족하는 불변 데이터 클래스다.<br>
> `public record Money(int amount) { }` 한 줄로 필드는 `private final`, 값 기반 `equals`/`hashCode`/`toString`, 접근자, 암묵적 final(상속 불가)까지 제공된다. 단, 필드가 가변 객체(List 등)라면 record라도 방어적 복사를 직접 해줘야 완전한 불변이 된다.

> 참고: sealed 클래스(자바 17+)
> permits로 명시한 자식만 상속을 허용하는 방식. 단, 상속 '금지'가 아니라 '제한'이므로 불변 강제보다는 상속 계층을 통제된 범위로 한정할 때 쓴다(각 자식 클래스는 final/sealed/non-sealed 중 하나를 명시해야 함).

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
문제: Money가 **상속**이 가능한 클래스인 경우(클래스에 final 미적용)<br>
부모 클래스 자체는 불변이지만, 자식 클래스가 메서드를 오버라이딩해 불변의 약속을 깰 수 있음.

case (1): 자식이 가변 상태를 추가하고 getter를 오버라이딩 → 불변이 깨짐
```java
// 부모 — 그 자체로는 불변
class Money {
    private final int amount;

    Money(int amount) {
        this.amount = amount;
    }

    public int getAmount() {
        return this.amount;
    }
}

// 자식 — 가변 필드를 추가하고 getAmount()를 오버라이딩
class MutableMoney extends Money {
    private int fakeAmount;

    MutableMoney(int amount) {
        super(amount);
        this.fakeAmount = amount;
    }

    public void setValue(int amount) {
        this.fakeAmount = amount;   // 변경 가능
    }

    @Override
    public int getAmount() {
        return this.fakeAmount;     // 부모의 amount가 아니라 가변 필드를 반환
    }
}

// 사용하는 측
Money price = new MutableMoney(1000);   // 다형성으로 부모 타입으로 받음
((MutableMoney) price).setValue(0);
price.getAmount();  // 0 — 부모 amount(1000)는 그대로지만 오버라이딩된 동작이 불변을 깬다
```

case (2): 필드 가리기(hiding)만으로는 불변이 깨지지 않음
```java
// 부모
class Money {
    protected final int amount;   // 자식에서 접근 가능하도록 protected

    Money(int amount) {
        this.amount = amount;
    }

    public int getAmount() {
        return this.amount;   // 부모 메서드는 항상 부모의 amount를 읽음(정적 바인딩)
    }
}

// 자식 — 같은 이름의 필드로 부모 필드를 가리지만(hiding), 메서드는 오버라이딩하지 않음
class ChildMoney extends Money {
    private int amount;   // 부모의 amount를 가림(hiding)

    ChildMoney(int amount) {
        super(amount);
        this.amount = amount;
    }

    public void setValue(int amount) {
        this.amount = amount;   // 자식의 amount만 바뀜
    }
}

// 사용하는 측
Money price = new ChildMoney(1000);
((ChildMoney) price).setValue(0);
price.getAmount();  // 1000 — getAmount()는 부모의 amount를 읽으므로 불변이 유지된다
```
필드는 정적 바인딩이라 부모의 `getAmount()`는 언제나 부모의 `amount`를 읽음. 따라서 필드를 가리는 것만으로는 불변이 깨지지 않고, **메서드 오버라이딩이 동반돼야** 깨짐.
> 필드는 정적 바인딩, 메서드는 동적 바인딩 — 자세한 내용은 oop/inheritance.md 참고.

> ⚠️ case (2)에서 부모 필드를 `protected`로 둔 것은 hiding을 보이기 위한 예시일 뿐이다. 실제 불변 클래스에서는 필드를 private으로 유지한다(2장 조건 참고).

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

> 핵심: 불변을 깨는 것은 필드 가리기(case 2)가 아니라 자식의 메서드 오버라이딩(case 1)이다.<br>
> 그래서 해결책은 필드를 잠그는 게 아니라 상속 자체를 막는 것(클래스 final 등)이다.