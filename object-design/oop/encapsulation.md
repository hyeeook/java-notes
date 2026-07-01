# 목차
1. [캡슐화가 왜 필요한가?](#1-캡슐화가-왜-필요한가)
2. [캡슐화를 구현하는 방법](#2-캡슐화를-구현하는-방법)
3. [주의점 (1) - setter가 있어도 캡슐화인가?](#3-주의점-1---setter가-있어도-캡슐화인가)
4. [주의점 (2) - 가변 객체 필드가 캡슐화를 깨는 경우](#4-주의점-2---가변-객체-필드가-캡슐화를-깨는-경우)

# 1. 캡슐화가 왜 필요한가?
## 캡슐화란?
데이터(필드)와 행위(메서드)를 하나로 묶고, 내부 구현을 외부에서 직접 접근하지 못하도록 숨기는 것.
> 캡슐화 ⊃ 정보 은닉: private으로 숨기는 것은 캡슐화를 구현하는 수단 중 하나이지, 캡슐화 자체가 아니다.

## 캡슐화가 없을 때 발생하는 문제
예시 상황: BankAccount의 balance 필드가 public인 경우

```java
public class BankAccount {
    public int balance; // 잔액: 외부에서 직접 접근 가능
}

// 사용하는 측
BankAccount account = new BankAccount();
account.balance = -99999;   // 아무도 막을 수 없음
account.balance = 0;        // 말도 안 되는 값도 허용됨
```
위와 같은 코드는 잘못된 값이 들어와도 컴파일 시점에 발견할 수 없고, 런타임에 가서야 버그를 발견할 수 있음.<br>
또한 balance 필드명이나 타입을 바꾸면 참조하는 모든 곳을 수정해야 하므로 변경 영향도가 커짐.

## 캡슐화의 목적
1. 값 오염 방지(외부에서 잘못된 값 주입 차단)
2. 변경 영향도 감소(내부 구현이 바뀌어도 외부 코드는 영향 없음)
3. 유효성 검사 가능(setter/메서드 내부에서 검증 로직 추가 가능)

# 2. 캡슐화를 구현하는 방법
1. 모든 인스턴스 필드는 private으로 선언
2. 외부에서 필요한 접근은 메서드를 통해서만 허용
3. 가변 객체를 인스턴스 필드로 사용할 경우 방어적 복사 적용

```java
public class BankAccount {
    private int balance;

    public BankAccount(int balance) {
        if(balance < 0) {
            throw new IllegalArgumentException("잔액은 0 이상이어야 합니다.");
        }

        this.balance = balance;
    }

    public void deposit(int amount) {
        if(amount <= 0) {
            throw new IllegalArgumentException("입금액은 0보다 커야 합니다.");
        }

        this.balance += amount;
    }

    public int getBalance() {
        return balance;
    }
}
```

# 3. 주의점 (1) - setter가 있어도 캡슐화인가?
setter를 추가해도 캡슐화의 형태는 유지됨. 필드를 직접 노출하는 것보다 낫고, 메서드 내부에서 유효성 검사도 가능함.<br>
단, setter가 있으면 외부에서 값을 자유롭게 바꿀 수 있으므로 불변성은 포기한 것.

| | `public` 필드 | `private` + setter | `private` + 불변 |
|:---:|:---:|:---:|:---:|
| 캡슐화? | ❌ | ✅ | ✅ |
| 값 변경 가능? | ✅ | ✅ | ❌ |
| 유효성 검사? | ❌ | ✅ | ✅ (생성자에서) |
| 언제 쓰나? | 쓰지 않는다 | Entity (상태가 바뀌는 객체) | Value Object (불변 객체) |
> Entity와 Value Object의 구분은 entity-vs-value-object.md 참고.

# 4. 주의점 (2) - 가변 객체 필드가 캡슐화를 깨는 경우
필드를 private으로 선언하고 setter도 없더라도, 가변 객체는 참조가 오가는 경로에서 방어적 복사를 하지 않으면 외부에서 내부 상태를 바꿀 수 있음. getter로 내보낼 때(출력)와 생성자·setter로 받을 때(입력) 모두 해당됨.

문제: getter가 List의 참조를 그대로 반환하는 경우 (출력 방향)
```java
public class Team {
    private List<String> members;

    // ❌ 참조 그대로 반환 — 캡슐화 깨짐
    public List<String> getMembers() {
        return members;
    }
}

// 사용하는 측
Team team = new Team();
team.getMembers().add("David"); // private인데 외부에서 내부 상태 변경 가능
team.getMembers().clear();      // 멤버 전체 삭제도 가능
```

해결: 방어적 복사로 반환
```java
public class Team {
    private List<String> members;

    // ✅ 방어적 복사 — 내부와 독립된 새 리스트를 반환
    public List<String> getMembers() {
        return new ArrayList<>(members);
    }
}
```
> ⚠️ `Collections.unmodifiableList(members)`는 방어적 복사가 아니라 불변 뷰(읽기 전용 래퍼)다.<br>
> 호출자의 수정은 막아주지만 내부 members를 그대로 들여다보는 래퍼라, 이후 내부 상태가 바뀌면 반환한 뷰에도 그대로 반영된다. 호출자의 변경만 막으면 되는 경우엔 충분하지만, 내부와 완전히 분리하려면 방어적 복사를 써야 한다.

문제: 생성자가 외부 List를 참조 그대로 저장하는 경우 (입력 방향)
```java
public class Team {
    private final List<String> members;

    // ❌ 외부 리스트 참조를 그대로 저장 — 캡슐화 깨짐
    public Team(List<String> members) {
        this.members = members;
    }
}

// 사용하는 측
List<String> input = new ArrayList<>(List.of("Alice"));
Team team = new Team(input);
input.add("David");   // 외부에서 input을 바꾸면 team 내부도 함께 바뀜
```

해결: 받을 때도 방어적 복사
```java
public Team(List<String> members) {
    this.members = new ArrayList<>(members);   // ✅ 입력 시점에 방어적 복사
}
```
> 방어적 복사는 가변 객체가 오가는 입력(생성자·setter)과 출력(getter) 양쪽 모두에 필요하다.

> 방어적 복사는 불변 클래스의 조건이기도 하다. 자세한 내용은 immutable-class.md 참고.