# OOP 4대 특징

객체지향 프로그래밍(OOP)의 핵심 4가지 특징을 정리한다.<br>
단순히 키워드(`private`, `extends` 등)를 외우는 게 아니라, "왜 필요한가"와 "잘못 쓰면 어떤 문제가 생기는가"를 중심으로 다룬다.

## 목차

| 특징 | 핵심 한 줄 | 문서 |
|:---:|:---|:---:|
| 캡슐화 | 데이터와 행위를 묶고, 내부 구현을 외부로부터 숨긴다 | [encapsulation.md](./encapsulation.md) |
| 추상화 | 복잡한 내부는 감추고, 무엇을 할 수 있는지만 드러낸다 | [abstraction.md](./abstraction.md) |
| 상속 | 공통 부분을 부모에 두고 재사용하되, 잘못 쓰면 위험하다 | [inheritance.md](./inheritance.md) |
| 다형성 | 같은 타입으로 서로 다른 객체를 동일하게 다룬다 | [polymorphism.md](./polymorphism.md) |

## 읽는 순서

캡슐화 → 추상화 → 상속 → 다형성 순으로 보는 것을 권장한다.<br>
추상화는 다형성의 전제 조건이 되고, 상속은 다형성이 성립하는 한 가지 방법이기 때문이다.

## 관련 문서

- [immutable-class.md](../immutable-class.md) — 캡슐화·상속과 깊이 연결되는 불변 클래스 설계
- [entity-vs-value-object.md](../entity-vs-value-object.md) — 캡슐화에서 다룬 Entity/Value Object 구분