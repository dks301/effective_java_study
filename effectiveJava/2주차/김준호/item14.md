# 아이템 14. Comparable을 구현할지 고려하라

`유일무이한 메서드인 compareTo`

- compareTo()는 단순 동치성 비교에 더해 순서까지 비교할 수 있으며 제네릭 하다.

```java
public interface Comparable<T> {
  int compareTo(T t);
}
```

### compareTo() 일반규약
- 이 객체와 주어진 객체의 순서를 비교한다.
- 이 객체가 주어진 객체보다 작으면 음수를, 같으면 0을, 크면 양수를 반환한다.
- 이 객체와 비교할 수 없는 타입의 객체가 주어지면 ClassCastException을 던진다.
- x.compareTo(y) = -y.compareTo(x)
- 예외가 발생한다면 반대의 경우도 발생해야한다.
- 추이성을 보장
- x.compareTo(y) > 0이고 y.compareTo(z)이면 x.compareTo(z) > 0도 성립
- (x.compareTo(y) == 0) == (x.equals(y))


```java
public int compareTo(PhoneNumber pn) {
  int result = Short.compare(this.areaCode, pn.areaCode);
  if(result == 0) {
      result = Short.compare(this.prefix, pn.prefix);
      if(result == 0) {
          result = Short.compare(line Num, pn.lineNum);
      }
  }
  return result;
}
```
정적 compare 메서드를 활용
```java
static Comparator<Object> hashCodeOrder = new Comparator<>() {
  public int compare(Object o1, Object o2) {
      return Integer.compare(o1.hashCode(), o2.hashCode());
  } 
}
```
비교자 생성 메서드를 활용한 비교자
```java
static Comparator<Object> hashCodeOrder = Comparator.comparingInt(o -> o.hashCode());
```

---

> 순서를 고려해야 하는 값 클래스를 작성할 경우 꼭 Comparable 인터페이스를 구현해야한다.
그 인스턴스들은 쉽게 정렬하고, 검색하고, 비교 기능을 제공하는 컬렉션과 어우러져야 한다.
compareTo()는 필드의 값을 비교할 때 '<'와 '>' 연산자는 지양하자.
대신 박싱된 기본 타입 클래스가 제공하는 정적 compare 메서드나 Comparator 인터페이스가 제공하는 비교자 생성 메서드를 사용.