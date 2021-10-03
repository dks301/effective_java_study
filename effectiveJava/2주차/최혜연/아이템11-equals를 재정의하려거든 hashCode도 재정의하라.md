# 3장. 모든 객체의 공통 메서드 - GOAL

> Object를 상속하는 클래스는 일반 규약에 맞게 재정의해야 하는데, 언제 어떻게 재정의해야 하는지 알아보자

## 아이템11. equals를 재정의하려거든 hashCode도 재정의하라

equals를 재정의한 클래스 모두에서 hashCode도 재정의해야 한다.  
그렇지 않으면 hashCode 일반 규약을 어기게 되어 해당 클래스의 인스턴스를 HashMap이나 HashSet같은 컬렉션의 원소로 사용할 때 문제를 일으킬 것이다.



**아래는 Object 명세서에 있는 규약이다.**

-   equals 비교에 사용되는 정보가 변경되지 않았다면, 애플리케이션이 실행되는 동안 그 객체의 hashCode 메서드는 몇 번을 호출해도 일관되게 항상 같은 값을 반환해야 한다. 단, 애플리케이션을 다시 실행한다면 이 값이 달라져도 상관없다.
-   equals(Object)가 두 객체를 같다고 판단했다면, 두 객체의 hashCode는 똑같은 값을 반환해야 한다.
-   equals(Object)가 두 객체를 다르다고 판단했더라도, 두 객체의 hashCode가 서로 다른 값을 반환할 필요는 없다. 단, 다른 객체에 대해서는 다른 값을 반환해야 해시테이블의 성능이 좋아진다.

## 번외. equals와 hashcode 중 하나만 재정의 하면 어떻게 될까?

-   hashCode 메소드로 해시코드 값이 같은지를 본다. 해시 코드값이 다르면 다른 객체로 판단하고, 해시 코드값이 같으면 `equals` 메소드로 다시 비교한다. 이 두 개가 모두 맞아야 동등 객체로 판단한다. 즉, 해시코드 값이 다른 엔트리끼리는 동치성 비교를 시도조차 하지 말아야 한다.

**`hashcode`를 재정의 하지 않으면**

-   같은 값 객체라도 해시값이 다를 수 있다. 따라서 `HashTable`에서 해당 객체가 저장된 버킷을 찾을 수 없다.

**`equals`를 재정의하지 않으면**

-   `hashcode`가 만든 해시값을 이용해 객체가 저장된 버킷을 찾을 수는 있지만 해당 객체가 자신과 같은 객체인지 값을 비교할 수 없기 때문에 null을 리턴하게 된다. 결론적으로, 역시 원하는 객체를 찾을 수 없다.

이러한 이유로 객체의 정확한 동등 비교를 위해서, 특히 Hash 관련 컬렉션 프레임워크를 사용할 때, `Object의 equals` 메서드만 재정의하지 말고 `hashCode`메서드도 재정의해서 논리적 동등 객체일경우 동일한 해시코드가 리턴되도록 해야한다.

## 논리적으로 같은 객체는 같은 해시코드를 반환해야 한다.

equals는 물리적으로 다른 두 객체를 논리적으로는 같다고 할 수 있다. 하지만 hashCode 메서드는 이 둘이 전혀 다르다고 판단하여, 규약과 달리 서로 다른 값을 반환한다.

```java
Map<PhoneNumber, String> m = new HashMap<>();
m.put(new PhoneNumber(707, 867, 5309), "제니");

//실행
m.get(new PhoneNumber(707, 867, 5309) // 제니 출력 x, null출력 o
```

아래 코드는 null이 출력되는데 그 이유는 논리적 동치인 두 객체가 서로 다른 해시코드를 반환하여 2번째 규약을 지키지 못했다. 그래서 get메서드는 엉뚱한 해시 버킷에 가서 객체를 찾으려 한 것이다.  
HashMap은 해시코드가 다른 엔트리끼리는 동치성 비교를 시도조차 하지 않도록 최적화 되어있다.

## 올바른 hashCode 메서드 구현

아래는 최악의 hashCode 구현이다.  
동치인 모든 객체에서 똑같은 해시코드를 반환하니 적법하지만, 모든 객체에 똑같은 값만 내어주므로, 모든 객체가 해시테이블의 버킷 하나에 담겨 마치linkedList처럼 동작한다. 객체가 많아지면 도저히 쓸수 없게 된다.

```java
@Override
public int hashCode() { return 42; }
```

좋은 해시 함수라면 서로 다른 인스턴스에 다른 해시코드를 반환한다. - 3번째 규약  
이상적인 해시 함수는 서로 다른 인스턴스들을 32비트 정수 범위에 균일하게 분배해야 한다.

### 좋은 hashCode 작성하는 요령

1.  int 변수인 result를 선언한 후 값을 c로 초기화한다.  
    이 때, c는 해당 객체의 첫번째 핵심 필드를 단계 2.1 방식으로 계산한 해시코드이다.  
    여기서 핵심 필드는 equals 비교에 사용되는 필드를 말한다.
2.  해당 객체의 나머지 핵심 필드인 f 각각에 대해 다음 작업을 수행한다.<br>

    2.1. 해당 필드의 해시코드 c 를 계산한다.<br><br>
        2.1.1. 기본 타입 필드라면, \`Type.hashCode(f)\`를 수행한다. 여기서 Type은 해당 기본타입의 박싱 클래스다.<br><br>
        2.1.2. 참조 타입 필드면서, 이 클래스의 equals 메소드가 이 필드의 equals를 재귀적으로 호출하여 비교한다면, 이 필드의 hashCode를 재귀적으로 호출한다. 필드의 값이 null이면 0을 사용한다.
        <br><br>
        2.1.3. 필드가 배열이라면, 핵심 원소 각각을 별도 필드처럼 다룬다. 이상의 규칙을 재귀적으로 적용해 각 핵심 원소의 해시코드를 계산한 다음, 단계 2.2방식으로 갱신한다. 모든 원소가 핵심 원소라면
    `Arrays.hashCode` 를 사용한다.<br><br>
    2.2. 단계 2.1에서 계산한 해시코드 c로 result를 갱신한다.<br>
    
    ```
    `result = 31 * result + c;`  
    `31`인 이유는 홀수이면서 소수이기 때문이다.
    ```

3.  result를 반환한다.

## hashCode 적용해보기

아래는 전형적인 hashCode 메서드이다.  
동치인 PhoneNumber인스턴스들은 같은 해시코드를 가질것이다. 단순하고, 충분히 빠르고, 서로 다른 전화번호들은 다른 해시 버킷들로 제법 훌륭히 분배해준다.  
해시 충돌이 더욱 적은 방법을 써야한다면 구아마의 `com.google.common.hash.Hashing`을 참고하자

```java
@Override
public int hashCode() {
    int result = Short.hashCode(areaCode);
    result = 31 * result + Short.hashCode(prefix);
    result = 31 * result + Short.hashCode(lineNum);
    return result;
}
```

Object 클래스는 임의 갯수만큼 객체를 받아 해시코드를 계산해주는 정적 메서드인 hash를 제공한다.  
한줄로 작성가능하지만, 속도는 더 느리다. 입력 인수를 담기 위한 배열이 만들어지고, 입력 중 기본타입이 있다면 박싱과 언박싱도 거쳐야하기 때문이다.

```java
@Override
public int hashCode() {
    return Objects.hash(lineNum, prefix, areaCode);
}
```

## 캐싱하는 방식

클래스가 불변이고, 해시코드를 계산하는 비용이 크다면, 캐싱방식을 고려하자  
타입의 객체가 주로 해시의 키로 사용될 것 같다면, 인스턴스가 만들어질 때 해시코드를 계산해둬야 한다.  
아래는 해시코드를 지연 초기화하는 hashCode 메서드 - 스레드 안정성까지 고려해야 한다.

```java
private int hashCode; // 자동으로 0초기화

@Override 
public int hashCode() {
    int result = hashCode;
    if( result == 0 ) {
        result = Short.hashCode(areaCode);
        result = 31 * result + Short.hashCode(prefix);
        result = 31 * result + Short.hashCode(lineNum);
        hashCode = result;
    }
    return result;
}
```

## 주의사항

성능을 높인다고, 해시코드를 계산할 때 핵심 필드를 생략해서는 안 된다. 해시테이블의 성능을 심각하게 떨어뜨릴 수도 있다. hashCode가 반환하는 값의 생성 규칙을 API사용자에게 자세히 공표하지 마라. 클래이언트가 이 값에 의지하지 않게되고, 추후에 계산 방식을 바꿀수도 있다.

## 결론

equals를 재정의할 때는 hashCode도 반드시 재정의해야 한다. 그렇지 않으면 프로그램이 제대로 동작하지 않을 것이다.  
재정의한 hashCode는 일반 규약을 따라야하며, 서로 다른인스턴스라면 해시코드도 서로 다르게 구현해야한다.  
`AutoValue` 프레임워크는 이를 자동으로 만들어준다.
