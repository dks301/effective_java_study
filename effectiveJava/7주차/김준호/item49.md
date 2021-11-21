## 아이템 49. 매개변수가 유효한지 검사하라

- 메서드와 생성자 대부분은 입력 매개변수의 값이 특정 조건을 만족하기를 바란다.
- 예컨대 인덱스 값은 음수가 되면 안되며, 객체 참조는 null이 아니어야 한다.
- 이런 제약은 메서드 몸체가 시작되기 전에 검사해야 한다.
- 오류는 가능한 빨리 잡아야 한다는 원칙이기도 하다.
- 매개변수 검사를 제대로 하지 않았을 경우 생길 수 있는 문제들.
    - 메서드 중간에 애매모호한 예외를 던지며 실패할 수 있다.
    - 메서드가 잘못된 값을 반환한다.
    - 메서드가 어떤 객체를 이상한 상태로 바꾸어 두어, 미래에 알 수 없는 문제가 발생한다.
- 즉, 매개변수 검사에 실패하면 실패 원자성(failure atomicity, 아이템 76)을 어기는 결과를 낳을 수 있다.

---

- public과 protected 메서드는 매개변수 값이 잘못됐을 때 던지는 예외를 문서화해야 한다.
- @throw 자바 doc 태그 사용!
- 즉, 매개변수의 제약을 문서화한다면 그 제약을 어겼을 때 발생하는 예외를 함께 기술해야 한다.
- 이러한 간단한 방법으로 API 사용자가 제약을 지킬 가능성을 크게 높일 수 있다.

```java
/**
 * (현재 값 mod m) 값을 반환한다. 이 메서드는
 *  항상 음이 아닌 BigInteger를 반환한다는 점에서 remainder 메서드와 다르다.
 *
 * @param m 계수 (양수여야 한다.)
 * @return 현재 값 mod m
 * @throws ArithmaticException m이 0보다 작거나 같으면 발생한다.
 */ 
 public BigInteger mod(BigInteger m) {
     if (m.signum() <= 0) {
         throw new ArithmaticException("m은 양수여야 합니다." + m);
     }
     // ...
 }
```
- 여기서 m이 null이면 m.signum() 호출에서 NullPointerException을 던지지만 메서드에 설명이 적혀있지 않다.
- 이유는 BigInteger 클래스 수준에서 기술했기 때문이다.

---

- null 검사는 자바7에 추가된 java.util.Objects.requireNonNull 메서드를 사용하는 것을 추천한다.
```java
public class Context {
    private Strategy strategy;

    public Context(Strategy strategy) {
        this.strategy = Objects.requireNonNull(strategy, "전략은 null일수 없습니다.");
    }
}
```

---

- 공개되지 않은 메서드라면 패키지 제작자인 여러분이 메서드가 호출되는 상황을 통제할 수 있다.
- assert(단언문)을 사용해 매개변수 유효성을 검증할 수 있다.
```java
private static void sort(long a[], int offset, int length) {
    assert a != null;
    assert offset >= 0 && offset <= a.length;
    assert length >= 0 && length <= a.length - offset;
    // ...
}
```
- 실패하면 AssertionError를 던진다.
- -ea 또는 --enableassertions 을 파라미터로 주면 런타임에 동작한다.
- 찾아보니 assert를 사용해야 할지 말지는 의견이 많이 갈리는 부분인 것 같다.
- 실제 운영하는 코드에서 -ea 옵션을 주지 않으면 잘못된 데이터가 들어오는 경우, 문제가 생길 수 있다.

---

> 메서드나 생성자를 작성할 때면 그 매개변수들에 어떤 제약이 있을지 생각하며 해야한다. 그 제약들을 문서화하고 메서드 코드 시작 부분에서 명시적으로 검사해야 한다. 이러한 습관을 반드시 기르도록 하자. 그 노력은 유효성 검사가 실제 예외를 걸러낼 때, 충분히 보상받을 것이다.