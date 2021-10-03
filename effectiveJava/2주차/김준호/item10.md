# 아이템 10. equals는 일반 규약을 지켜 재정의하라

- equals 메서드 재정의는 곳곳에 함정이 도사리고 있어 끔찍한 결과를 초래한다.
- 다음의 상황 중 하나라도 해당된다면 재정의하지 않는 것이 좋다.

## equals 재정의 하지 않는 경우

### 각 인스턴스가 본질적으로 고유한 경우

- 값을 표현하는 것이 아니라 동작하는 개체를 표현하는 클래스일 경우를 말한다.
- Thread... (Object의 equals는 이러한 클래스에 딱 맞게 구현 되어 있으므로 재정의 x)

### 인스턴스의 논리적 동치성을 검사할 일이 없을 경우

### 상위 클래스에서 재정의한 equals가 하위 클래스에도 딱 들어맞는 경우

ex. List, Map, Set 모두 AbstractList, AbstractMap, AbstractSet의 equals를 그대로 사용한다.

### ~~클래스가 private이거나 package-private이고~~ equals 메서드를 호출할 일이 없을 경우

equals 자체를 호출하는 걸 막고 싶다면?

```java
@Override
public boolean equals (Object object) {
	throw new AssertionError();
}
```

## 그렇다면 언제 equals 메서드를 재정의해야 하냐?

- 객체 식별성이 아니라 논리적 동치성을 확인해야 하는데, 상위 클래스의 equals가 논리적 동치성을 비교하도록 재정의되지 않았을 때다.
ex. Integer, String...
- 즉, 같은 객체인지가 아니라 내부 value가 같은지 확인해야 하는 객체들!
- 이러한 점을 이용해 Map의 키와 Set의 원소로 사용할 수 있게 된다.

- 그리고 인스턴스 통제 클래스(싱글톤, Enum)는  equals를 재정의 할 필요가 없다.

## equals 메서드 일반 규약 (Object에 명세되어 있음)
`홀로 존재하는 클래스는 없고, 한 클래스의 인스턴스는 다른 곳으로 빈번히 전달 된다. 컬렉션과 같은 수많은 클래스는 equals 규약을 지킨다고 가정하고 동작한다.`

- equals 메서드는 동치 관계를 구현하며, 다음을 만족한다.
- 모든 참조값은 null이 아니다.

### 반사성(reflexivity)

```java
x.equlas(x) == true
```

- 객체는 자기 자신과 같아야 한다.
- 컬렉션에 contains 메서드를 호출해 true가 나오면 된다.

### 대칭성(symmetry)

```java
x.equals(y) == y.equals(x)
```

- 서로에 대한 동치 여부에 똑같이 답해야 한다.

```java
public final class CaseInsensitiveString {
    private final String s;

    public CaseInsensitiveString(String s) {
        this.s = Objects.requireNonNull(s);
    }

    @Override
    public boolean equals(Object o) {
        if(o instanceof CaseInsensitiveString) {
            return s.equalsIgnoreCase((CaseInsensitiveString) o).s);
        }
        if (o instanceof String) {
            return s.equalsIgnoreCase((String) o);
        }
    }
} 
```
```java
CaseInsensitiveString cis = new CaseInsensitiveString("Data");
String s = "Data";

cis.equals(s);  // true
s.equals(cis);  // false
```

`CaseInsensitiveString의 equals를 String과도 연동 시키려는 허황된 꿈을 버리면 된다.`

### 추이성(transitivity)

```java
x.equals(y) == true
y.equals(z) == true
x.equals(z) == true
```

- 상속 관계에서 필드를 추가한 경우, equals 규약을 지킬 수 있는 방법은 존재하지 않는다! `추상 클래스는 인스턴스화 불가능이므로 제외.`
- java.util.Date를 상속한 java.sql.Timestamp는 대칭성을 위배한다.
- **구성을 사용**하여 각 클래스의 필드만을 equals로 검사한다면 자연스럽게 해결된다.
### 일관성(consistency)

```java
x.equals(y) == true;
x.equals(y) == true;
x.equals(y) == true;
```

- 가장 좋은 방법은 불변이다.
- 하지만 불변을 사용할지라도 **equals의 판단에 신뢰할 수 없는 자원이 끼어들게 해서는 안 된다.**
- java.net.URL의 equals는 호스트의 IP주소를 사용하므로 결과가 변할 수 있다.
- 즉, equals는 항시 메모리에 존재하는 객체만을 사용하여 결정적인 계산을 해야한다.

### non-null

```java
x.equals(null) == false;
```

```java
@Override
public boolean equals(Object o) {
    if (o == null) {
        return false;
    }
}
```
- 위와 같이 명시적으로 null을 검사할 필요는 없다.
```java
@Override
public boolean equals(Object o) {
    if (!(o isinstanceof MyType)) {
        return false;
    }
    MyType myType = (MyType) o;
}
```
- 이와같이 묵시적으로 null을 검사하면된다.

## equals 메서드 구현 방법 단계별 정리

1. == 연산자를 사용해 입력이 자기 자신의 참조인지 확인한다.
    - 단순한 성능 최적화용으로 비교 작업이 복잡한 상황일 때 값어치를 한다.
2. instanceof 연산자로 입력이 올바른 타입인지 확인한다.
    - 묵시적 null 체크를 할 수 있다.
    - 또한 Collection interface처럼 특정 인터페이스의 타입 체크를 할 수도 있다.
    - List, Set, Map, Map.Entry
3. 입력을 올바른 타입으로 형변환한다.
    - instanceof 검사를 했기 때문에 100% 성공한다.
4. 입력 객체와 자기 자신의 대응되는 핵심 필드들이 모두 일치하는지 하나씩 검사한다.

---

- 기본 타입은 `==` 으로 비교하고 그 중 double, float는 Double.compare(), Float.compare()을 이용해 검사해야 한다. Float.Nan, -0.0f와 같은 특수한 부동소수 값을 다뤄야 하기 때문이다.
- Float.equals와 Double.equals는 오토박싱을 수반할 수 있으므로 성능을 저하시킬 수 있다.
- 배열의 모든 원소가 핵심 필드이면 Arrays.equals 메서드들 중 하나를 사용하자.
- null을 정상 필드로 취급하는 객체는 Objects.equals()를 이용해 NullPointerException 발생을 예방하자.
- 최상의 성능을 바란다면 다를 가능성이 더 크거나 비교하는 비용이 싼 필드를 먼저 비교하자.
- 동기화용 락 필드 같이 객체의 논리적 상태와 관련 없는 필드는 비교하면 안된다.
- equals를 재정의할 땐 hashCode도 반드시 재정의 하자
- Object 타입이 아닌 타입을 매개변수로 받는 equals 메서드는 선언하지 말자. @Override 어노테이션으로 확인하자.

---

`꼭 필요한 경우가 아니면 equals를 재정의 하지 말자.
equals를 구현해야한다면 그 클래스의 핵심 필드 모두를 빠짐없이, 다섯 가지 규약을 확실히 지켜가며 비교해야 한다.`

[이펙티브 자바 3판]