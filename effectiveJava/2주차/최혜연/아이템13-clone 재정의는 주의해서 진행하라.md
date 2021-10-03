# 3장. 모든 객체의 공통 메서드 - GOAL

> Object를 상속하는 클래스는 일반 규약에 맞게 재정의해야 하는데, 언제 어떻게 재정의해야 하는지 알아보자

## 아이템13. clone 재정의는 주의해서 진행하라

Cloneable을 구현하는 것만으로는 외부 객체에서 clone 메서드를 호출할 수 없다.

## 번외. clone은 무엇인가

-   객체의 원본을 복제하는데 사용하는 메서드이다.
-   생성과정의 복잡한 과정을 반복하지 않고, 복제할 수 있다.
-   clone 메서드를 사용하면, 객체의 정보가 동일한 또 다른 인스턴스가 생성되는 것이므로, 객체지향 프로그램에서의 정보 은닉, 객체보호의 관점에서 위배될 수 있다.
-   해당 클래스의 clone 메서드의 사용을 허용한다는 의미로 cloneable 인터페이스를 명시해 준다.
-   얕은 복사는 주소값만 복사(하나 수정하면 같이 수정), 깊은 복사는 내용까지 복사)

```java
public class Student implements Cloneable {
    @Override
    protected Object clone() throws CloneNotSupportedException {
        return super.clone();
    }
}
```

## Cloneable 인터페이스의 역할

Cloneable은 복제해도 되는 클래스임을 명시하는 용도의 믹스인 인터페이스지만, 의도한 목적을 제대로 이루지 못했다. `clone`메서드가 선언된 곳이 `Cloneable`이 아닌 `Object`이고, 그마저도 `protected` 이다. 그래서 Cloneable을 구현하는 것 만으로는 외부 객체에서 `clone` 메서드를 호출할 수 없다.

하지만 이런 문제점에도 불구하고, `Cloneable` 은 널리쓰이고 있어서, 잘알아두는 것이 좋다.




**Object의 protected 메서드인 clone의 동작 방식을 결정한다.**
Cloneable을 구현한 클래스의 인스턴스에서 clone을 호출하면, 그 객체의 필드들을 하나하나 복사한 객체를 반환하며, 그렇지 않은 클래스의 인스턴스에서 호출하면 CloneNotSupportedException을 던진다.

실무에서 Clonable을 구현한 클래스는 clone 메서드를 public으로 제공하며, 사용자는 당연히 복제가 제대로 이뤄지리라 기대한다.

## clone 메서드의 일반 규약

객체의 복사본을 만들어서 반환한다.'복사'의 뜻은 그 객체를 구현한 클래스에 따라 다를 수 있다.
일반적이 의도는 다음과 같다.

> `x.clone() != x` 는 참이다.
> `x.clone().getClass() == x.getClass()` 도 참이다.
> `x.clone().equals(x)` 도 일반적으로 참이다.

하지만 위의 예시는 모두 필수가 아니다.
관례상, 이 메서드가 반환하는 객체는 `super.clone`을 호출해 얻어야 한다.

관례상, 반환된 객체와 원본 객체는 독립적이어야한다. 이를 만족하려면 `super.clone`으로 얻은 객체의 필드 중 하나 이상을 반환 전에 수정해야 할 수도 있다.

## clone 메서드의 구현

### 기본 타입이거나 불변 객체

먼저 super.clone을 호출한다. 이렇게 얻은 객체는 원본의 완벽한 복제본일 것이다. 클래스에 정의된 모든 필드는 원본 필드와 똑같은 값을 갖는다. 모드 필드가 기본 타입이거나 불변 객체를 참조한다면 이 객체는 완벽히 우리가 원하는 상태라 더 손볼 것이 없다.

```java
@Override
public PhoneNumber clone() {
    try {
        return (PhoneNumber) super.clone();
    } catch (CloneNotSupportedException e) {
        throw new AssertionError(); //일어날 수 없는 일이다.
    }
}
```

이 메서드가 동작하게 하려면 Cloneable을 구현한다고 추가해야한다. PhoneNumber를 반환하게 했는데, 재정의한 메서드의 반환 타입은 상위 클래스의 메서드가 반환하는 타입의 하위 타입일 수 있다. 이 방식으로 클라이언트가 형변환하지 않아도 되게끔 해주자.

### 클래스가 가변 객체를 참조

clone 메서드는 사실상 생성자와 같으 효과를 낸다. **clone은 원본 객체에 아무런 해를 끼치지 않는 동시에 복제된 객체의 불변식을 보장해야한다.**
Stack의 clone메서드는 제대로 동작하려면, 스택 내부 정보를 복사해야 하는데, 가장 쉬운 방법응 elements 배열의 clone을 재귀적으로 호출해주는 것이다.

```java
@Override
public Stack clone() {
    try {
        Stack result = (Stack) super.clone();
        result.elements = elements.clone();
        return result;
    } catch (CloneNotSupportedException e) {
        throw new AssertionError();
    }
}
```

clone을 재귀적으로 호출하는 것만으로는 충분하지 않을 때도 있다.
해시테이블 내부는 버킷들의 배열이고, 각 버킷은 키-값 쌍을 담는 연결 리스트의 첫번째 엔트리를 참조한다.

```java
public class HashTable implements Cloneable {
  private Entry[] buckets = ...;

  private static class Entry {
    final Object key;
    Object value;
    Entry next;

    Entry(Obejct key, Object value, Entry next) {
      this.key = key;
      this.value = value;
      this.next = next;
    }

    // 1. 이 엔트리가 가리키는 연결 리스트를 재귀적으로 복사
    // bucket의 크기가 크지 않다면 괜찮지만 너무 크다면 콜 스택 오버플로가 발생한다.
    Entry deepCopy() {
      return new Entry(key, value, next == null ? null : next.deepCopy());
    }
  }

  @Override
  public HashTable clone() {
    try {
      HashTable result = (HashTable) super.clone();
      result.buckets = new Entry[buckets.length];
      for (int i = 0; i < buckets.length; i++) {
        if (buckets[i] != null) {
          result.buckets[i] = buckets[i].deepCopy();
        }
      }
      return result;
    } catch (CloneNotSupportedException e) {
      throw new AssertionError();
    }
  }
}
```

```java
// 2. 재귀호출대신 반복자로 써서 순회한다. -> 엔트리 자신이 가리키는 연결 리스트를 반복적으로 복사한다.
Entry deepCopy() {
  Entry result = new Entry(key, value, next);
  for (Entry p = result; p.next != null; p = p.next) {
    p.next = new Entry(p.next.key, p.next.value, p.next.next);
  }
  return result;
}
```

### 참조

-   Cloneable 을 구현하는 모든 클래스는 clone을 재정의해야 한다.
-   접근 제한자는 public 으로, 반환 타입은 클래스 자신으로 변경한다.
-   기본 타입 필드와 불변 객체 참조만 갖는 클래스라면 아무 필드도 수정할 필요가 없다.
-   단, 일련번호나 고유 ID는 비록 기본 타입이나 불변일지라도 수정해줘야 한다.

이미 구현한 클래스를 확장하는게 아니라면 복사 생성자와 복사 팩토리라는 더 나은 객체 복사 방식을 제공할 수 있다.

## 복사 생성자와 복사 팩터리 - 더 나은 객체 복사 방식

복사 생성자란 단순히 **자신과 같은 클래스의 인스턴스를 인수로 받는 생성자**를 말한다.
복사 팩터리는 **복사 생성자를 모방한 정적 팩터리**다.

더 정확한 이름은 `변환 생성자(conversion constructor)`, `변환 팩터리(conversion factory)` 이다.
이들을 이용하면, 클라이언트는 원본의 구현 타입에 얽매이지 않고 복제본의 타입을 직접 선택할 수 있다.
(HashSet 객체 s를 TreeSet 타입으로 복제할 수 있다)

```java
public Yum(Yum yum) {...}; //복사 생성자

public static Yum newInstance(Yum yum) {..}; //복사 팩터리
```

복사 생성자와 그 변형인 복사 팩터리는 Cloneable/clone 방식보다 나은 면이 많다.
1\. 언어 모순적이고 위험천만한 객체 생성 매커니즘을 사용하지 않는다.
2\. 엉성하게 문서화된 규약에 기대지 않는다.
3\. 정상적인 final 필드 용법과도 충돌하지 않는다.
4\. 불필요한 검사 예외를 던지지 않고, 형변환도 필요치 않다.
5\. 복사 생성자와 복사 팩터리는 해당 클래스가 구현한 '인터페이스'타입의 인스턴스를 인수로 받을 수 있다.

## 결론

Cloneable이 몰고온 모든 문제를 되짚어 봤을 때, 새로운 인터페이스를 만들 때는 절대 Cloneable을 확장해서는 안되며, 새로운 클래스도 이를 구현해서는 안된다. final클래스라면 Cloneable을 구현해도 위험이 크지 않지만, 성능 최적화 관점에서 검토한 후, 별다른 문제가 없을 때만 드물게 허용해야 한다.
기본원칙은 '복제 기능은 생성자와 팩터리를 이요하는게 최고'라는 것이다. 단, `배열은 clone 메서드 방식`이 가장 깔끔한, 이 규칙의 합당한 예외라 할 수 있다!

## 참조

[https://blog.naver.com/seoooohs2/222462390130](https://blog.naver.com/seoooohs2/222462390130)
