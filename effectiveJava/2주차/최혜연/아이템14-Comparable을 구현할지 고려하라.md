# 3장. 모든 객체의 공통 메서드 - GOAL

> Object를 상속하는 클래스는 일반 규약에 맞게 재정의해야 하는데, 언제 어떻게 재정의해야 하는지 알아보자

## 아이템14. Comparable을 구현할지 고려하라

Comparable 인터페이스의 유일무이한 메서드인 `compareTo`를 알아보자.



-   compareTo는 Object의 메서드가 아니다.
-   compareTo는 단순 동치성 비교에 더해 순서까지 비교할 수 있으며, 제네릭하다.
-   Comparable을 구현했다는 것은, 그 클래스의 인스턴스들에는 자연적인 순서가 있음을 뜻한다.  
    그래서 Comparable을 구현한 객체들의 배열은 다음처럼 손쉽게 정렬할 수 있다. `Arrays.sort(a)`
-   검색, 극단값 계산, 자동 정렬되는 컬렉션 관리도 쉽게할 수 있다.

아래 코드는 알파벳순으로 출력된다. String이 Comparable을 구현한 덕분이다.

```java
public class WordList {
    public static void main(String[] args){
        Set<String> s = new TreeSet<>();
        Collections.addAll(s, args);
        System.out.println(s);
    }
}
```

```java
// 구현
pulib interface Comparable<T> {
    int compareTo(T t);
}
```

## compareTo 메서드 일반 규약

이 객체와 주어진 객체의 순서를 비교한다.  
이 객체가 주어진 객체보다 작으면 음수를, 같으면 0을, 크면 양수를 반환한다.  
이 객체와 비교할 수 없는 타입의 객체가 주어지면 ClassCastException을 던진다.



1\. Comparable을 구현한 클래스는 `x.compareTo(y) == y.compareTo(x)` 이다.

-   또한 예외가 발생한다면 양쪽 표현식 모두 발생한다.
-   두 객체 참조의 순서를 바꿔 비교해도 예상한 결과가 나와야 한다.

2\. Comparable을 구현한 클래스는 추이성을 보장해야 한다.

-   `x.compareTo(y) > 0 && y.compareTo(z) > 0` 이면 `x.compareTo(z) > 0` 도 성립한다.

3\. Comparable을 구현한 클래스는 `x.compareTo(y) == 0` 이면 `x.compareTo(z) == y.compareTo(z)` 이다.

-   크기가 같은 객체들끼리는 어떤 객체와 비교하더라도 항상 같아야 한다.

4\. `(x.compareTo(y) == 0) == (x.equals(y))` 여야 한다.

-   이 권고는 필수는 아니지만, 지키는게 좋다.
-   compareTo 메서드로 수행한 동치성 테스트의 결과가 equals와 같아야 한다.
-   이를 잘 지키면 compareTo로 줄지은 순서와 equals의 결과가 일관되게 한다.
-   Comparable을 구현하고, 이 권고를 지키지 않는 모든 클래스는 그 사실을 명시해야 한다. 다음과 같이 명시하면 적당하다.  
    `이 클래스의 순서는 equals 메서드와 일관되지 않다`

1~3 규약들은 compareTo 메서드로 수행하는 동치성 검사도 equals 규약과 똑같이 반사성, 대칭성, 추이성을 충족해야 함을 뜻한다. 그래서 주의사항도 똑같다.

4번 규약을 지키지 못하면, 비교를 활용하는 클래스와 어울리지 못한다. 비교를 활용하는 클래스의 예로는 정렬된 컬렉션인 `TreeSet, TreeMap, Collections, Arrays` 가 있다. 예시로 BigDecimal 클래스가 있다.

```java
new BigDecimal("1.0")
new BigDecimal("1.00")

// equals메서드를 비교하면, 서로 다르기 때문에 HashSet에서는 원소를 2개
// compareTo 메서드로 비교하면, TreeSet 메서드에서는 원소 1개. 두 BigDecimal 인스턴스가 똑같다.
```

## compareTo 메서드 작성 요령 (3가지)

-   `compareTo` 메서드의 인수 타입은 컴파일 타임에 정해지기 때문에 입력 인수의 타입을 확인하거나 형변환할 필요가 없다.
-   `compareTo` 메서드는 각 필드가 동치인지를 비교하는게 아니라 그 순서를 비교한다.
-   `Comparable` 을 구현하지 않은 필드나 표준이 아닌 순서로 비교해야 한다면, `Comparator`를 대신 사용한다.
-   비교자는 직접 만들거나 자바가 제공하는 것중에 골라쓰면 된다.
-   `compareTo` 메서드에서 관계 연산자 `<`, `>` 를 사용하는 이전 방식은 거추장스럽고 오류를 유발하니, 추천하지 않는다.

### 1\. 객체 참조 필드가 하나뿐인 비교자

```java
public final class CaseInsensitiveString implements Comparable<CaseInsensitiveString> {
    public int compareTo(CaseInsensitiveString cis) {
        return String.CASE_INSENSITIVE_ORDER.compare(s, cis.s);
    }
```

### 2\. 기본 타입 필드가 여럿일 때의 비교자

```java
public int compareTo(PhoneNumber pn) {
  int result = Short.compare(this.areaCode, pn.areaCode); //가장 중요한 필드
  if(result == 0) {
      result = Short.compare(this.prefix, pn.prefix); //두번째로 중요한 필드
      if(result == 0) {
          result = Short.compare(line Num, pn.lineNum); //세번째로 중요한 필드
      }
  }
  return result;
}
```

### 3\. 비교자 생성 메서드를 활용한 비교자

-   Comparator 인터페이스가 일련의 비교자 생성 메서드와 팀을 꾸려 메서드 연쇄 방식으로 비교자를 생성할 수 있게 되었다.
-   더 멋지게 활용할 수 있지만, 성능 저하가 뒤따른다.

```java
private static final Comparator<PhoneNumber> COMPARATOR = 
    comparingInt((PhoneNumber pn) -> pn.areaCode)
        .thenComparingInt(pn -> pn.prefix)
        .thenComparingInt(pn -> pn.lineNUm);

public int compareTo(PhoneNumber pn) {
    return COMPARATOR.compare(this, pn);
}
```

## 예시로, 해시코드 값을 기준으로 비교하는 비교자에는 2가지 방법이 있다.

`o1.hashCode - o2.hashCode` 이런식으로 하면 안된다.

### 1\. 정적 compare 메서드를 활용한 비교자

```java
static Comparator<Object> hashCodeOrder = new Comparator<>() {
  public int compare(Object o1, Object o2) {
      return Integer.compare(o1.hashCode(), o2.hashCode());
  } 
}
```

### 2\. 비교자 생성 메서드를 활용한 비교자

```java
static Comparator<Object> hashCodeOrder = Comparator.comparingInt(o -> o.hashCode());
```

## 결론

순서를 고려해야 하는 값 클래스를 작성해야 한다면 꼭 `Comparable` 인터페이스를 구현하여, 인스턴스들을 쉽게 정렬, 검색, 비교 기능을 제공하는 컬렉션과 어우러지도록 해야한다.

compareTo 메서드에서 필드의 값을 비교할때 `<`, `>` 연산자는 쓰지 말아야 한다.  
그 대신 박싱된 기본 타입 클래스가 제공하는 정적 `compare` 메서드나 `Comparator` 인터페이스가 제공하는 비교자 생성 메서드를 사용하자.