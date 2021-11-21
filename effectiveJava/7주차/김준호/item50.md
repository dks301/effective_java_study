## 아이템 50. 적시에 방어적 복사본을 만들라

> 자바는 안전한 언어이다. 즉, 포인터를 직접 다루는 언어에서 발생하는 메모리 충돌에서 안전한다.

- 하지만 다른 클래스로 부터의 공격을 막아야한다.
- 즉, 클라이언트가 내가 작성한 코드의 불변식을 깨뜨리려 혈안이 되어 있다고 가정하고 방어적으로 프로그래밍 해야한다.

```java
public final class Period {
    private final Date start;
    private final Date end;

    public Period(Date start, Date end) {
        if (start.compareTo(end) > 0) {
            throw new IllegalArgumentException("start가 end보다 나중이다.");
        }
        this.start = start;
        this.end = end;
    }

    public Date getStart() {
        return start;
    }

    public Date getEnd() {
        return end;
    }
}
```
- 위의 코드는 final을 사용해서 불변으로 착갈할 수 있지만 Date는 가변이기 때문에 쉽게 불변식을 깰 수 있다.

```java
Date start = new Date();
Date end = new Date();
Period period = new Period(start, end);
end.setYear(99);
```

- period 내부를 수정했다.
- 자바8 부터는 Date 대신 불변인 Instant를 사용하면 된다.
- 하지만 레거시 코드에는 Date가 넘쳐날 것이다.

---

- 위의 공격을 막으려면 생성자에서 매개변수의 복사본을 만들어야 한다.

```java
public Period(Date start, Date end) {
    this.start = new Date(start.getTime());
    this.end = new Date(end.getTime());

    if (start.compareTo(end) > 0) {
        throw new IllegalArgumentException("start가 end보다 나중이다.");
        }
}
```

- 위의 코드에서 복사본을 만들고 매개변수의 유효성을 검사한 것에 주목해야한다.
- 매개변수의 유효성을 먼저 검사하고 복사본을 만들면 복사본을 만들기 전에 다른 스레드가 상태를 변화시킬 수 있다.
- 실제로 TOCTOU 공격이라고 한다.
- 그리고 clone 메서드는 사용하지 않도록한다.
- 왜냐하면 Date는 가변이기 때문에 clone을 악의를 가지고 재정의 했을 수 있다.

```java
Date start = new Date();
Date end = new Date();
Period period = new Period(start, end);
period.getEnd().setYear(99);
```

- 생성자에서의 복사 만으로는 부족하다.
- 위와같이 getter로 받아서 불변 식을 공격할 수 있다.

---

```java
public Date getStart() {
    return new Date(start.getTime());
}

public Date getEnd() {
    return new Date(end.getTime());
}
```

- 위와 같이 getter에서 복사본을 반환하면 된다.
- 마침내 모든 가변 필드가 완벽하게 캡슐화 되었다.
- ~~방어적 복사를 사용하지 않으려면 확실하게 문서화를 해야한다.~~

> 클래스가 클라이언트로부터 받는 혹은 클라이언트로 반환하는 구성요소가 가변이라면 그 요소는 반드시 방어적으로 복사해야 한다. 복사 비용이 너무 크거나 클라이언트가 그 요소를 잘못 수정할 일이 없음을 신뢰한다면 방어적 복사를 수행하는 대신 해당 구성요소를 수정했을 때의 책임이 클라이언트에 있음을 문서에 명시하도록 하자.