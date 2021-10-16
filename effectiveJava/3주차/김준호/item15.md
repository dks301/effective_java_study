# 아이템 15. 클래스와 멤버의 접근 권한을 최소화하라

> 잘 설계된 컴포넌트는 클래스 내부 데이터와 내부 구현 정보를 외부 컴포넌트로부터 얼마나 잘 숨겼는가에 결정된다. 모든 내부 구현을 완벽하게 숨겨, 구현과 API를 깔끔하게 분리하는 것이다. 정보 은닉, 캡슐화는 소프트웨어 설계의 근간이 되는 원리이다.


## 정보 은닉의 장점
- 시스템 개발 속도를 높여준다.
- 여러 컴포넌트를 병렬로 개발할 수 있다.
시스템 관리 비용을 낮춰준다.
- 소프트웨어의 재사용성을 높여준다.
- 외부에 거의 의존하지 않고 독자적으로 동작하는 컴포넌트라면 낯선 환경에서도 유용하게 쓰일 가능성이 높다.
- 큰 시스템을 제작하는 난이도를 낮춰줍니다. 시스템 전체가 완성되지 않은 상태에서도 개별 컴포넌트의 동작을 검증할 수 있기 때문이다.

`모든 클래스와 멤버의 접근성을 가능한 한 좁혀야 한다.`


접근 지정자의 종류

- private : 멤버를 선언한 톱레벨 클래스에서만 접근할 수 있다.
- package-private : 멤버가 소속된 패키지 안의 모든 클래스에서 접근할 수 있다.접근 제한자를 지정하지 않았을 때 적용되는 패키지 접근 수준이다. (인터페이스의 멤버는 기본적으로 public이 적용된다.)
- protected : package-private의 접근 범위를 포함하며, 이 멤버를 선언한 클래스의 하위 클래스에서도 접근할 수 있다.
- public : 모든 곳에서 접근할 수 있다.

---

- 톱레벨 클래스를 패키지 외부에서 사용할 이유가 없다면 package-private으로 선언하자.
- 그러면 API가 아닌 내부 구현이 되어 언제든 수정할 수 있다.
- 한 클래스에서만 사용하는 package-private 톱레벨 클래스나 인터페이스는 private static 중첩 클래스를 사용하자.

---

- public 클래스의 인스턴스 필드는 되도록 public이 아니어야 한다. 불변을 보장하기 어렵고 일반적으로 스레드 안전하지 않다.
- public static final 필드는 기본 타입이나 불변 객체를 참조해야 한다.
- public static final 배열 필드를 두거나 이를 반환하는 접근자 메서드는 두면 안된다.
- 다른 객체를 참조하도록 바꿀 수는 없지만 참조된 객체 자체가 수정될 수는 있다.
- 길이가 0이 아닌 배열은 모두 변경 가능하다.

```java
public static final Long[] VALUES = {1L, 2L, 3L};
```
위험!

```java
private static final Long[] PRIVATE_VALUES = {1L, 2L, 3L};
public static final List<Long> VALUES = Collections.unmodifiableList(Arrays.asList(PRIVATE_VALUES));
```

또는

```java
private static final Long[] PRIVATE_VALUES = {1L, 2L, 3L};
public static final Integer[] values() {
    return PRIVATE_VALUES.clone();
}
```

- 클라이언트가 무엇을 원하는지에 따라 둘 중 하나의 방법을 선택하면 된다.

> 프로그램 요소의 접근성은 가능한 한 최소한으로 하라. 꼭 필요한 것만 골라 최소한의 public API를 설계하자. 그 외 클래스, 인터페이스, 멤버가 의도치 않게 API로 공개되는 일이 없도록 해야 한다. public class는 상수용 public static final 필드 외에는 어떠한 public 필드도 가져서는 안 된다. public static final 필드가 참조하는 객체가 불변인지 확인하라.

[이펙티브 자바 3판]
