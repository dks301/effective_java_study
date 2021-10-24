# 아이템 11. equals를 재정의하려거든 hashCode도 재정의하라

`equals를 재정의한 클래스는 hashCode도 재정의 해야 한다.`

그렇지 않으면 hashCode 일반 규약을 어기게 되어 해당 클래스의 인스턴스를 HashMap이나 HashSet 같은 컬렉션의 원소로 사용할 때 문제를 일으킬 것이다.


- equals 비교에 사용되는 정보가 변경되지 않았다면, hashCode 메서드가 리턴하는 값도 변해서는 안된다. (애플리케이션 다시 실행하는 경우 제외)
- equals가 두 객체가 같다고 판단했다면, 두 객체의 hashCode는 똑같은 값을 반환해야한다.
- equals가 두 객체를 다르다고 판단했더라도, hashCode는 꼭 다를 필요는 없다. 하지만, 다른 객체에 대해서는 다른 값을 반환해야 해시테이블의 성능이 좋아진다.
 

## 논리적으로 같은 객체는 같은 해시코드를 반환해야 한다.
equals가 물리적으로 다른 두 객체를 논리적으로 같다고 할 때, Object의 기본 hashCode는 서로 다른 값을 반환한다.

```java
Map<PhoneNumber, String> map = new HashMap<>();
map.put(new PhoneNumber(010,1234,5678), new Person("Jenny"));
map.get(new PhoneNumber(010,1234,5678))
```
위 코드를 실행하면 "Jenny"가 아닌 null을 반환한다.

PhoneNumber 클래스는 hashCode를 재정의하지 않았기 때문에, 논리적 동치인 두 객체가 서로 다른 해시코드를 반환하여 get 메서드는 다른 해시 버킷에 가서 객체를 찾으려 한 것이다.

HashMap은 해시코드가 서로 다른 객체끼리는 동치성 비교를 시도조차 않도록 최적화 되어있다.

```java
@Override 
public int hashCode() {
	return 42;
}
```
모든 객체에게 똑같은 값만 내어주므로 모든 객체가 해시테이블의 버킷 하나에 담겨 마치 연결리스트처럼 동작한다. 그 결과 평균 수행 시간이 O(1)인 해시테이블이 O(n)으로 느려져서, 도저히 쓸 수 없게 된다.

 

## 좋은 해시 함수라면 서로 다른 인스턴스에 다른 해시코드를 반환한다.

- 좋은 hashCode를 작성하는 요령
int 변수인 result를 선언한 후 값을 c로 초기화한다.
- 이 때, c는 해당 객체의 첫번째 핵심 필드를 단계 2.1 방식으로 계산한 해시코드이다.
- 여기서 핵심 필드는 equals 비교에 사용되는 필드를 말한다.
- 해당 객체의 나머지 핵심 필드인 f 각각에 대해 다음 작업을 수행한다.
- 해당 필드의 해시코드 c 를 계산한다.
- 기본 타입 필드라면, Type.hashCode(f)를 수행한다. 여기서 Type은 해당 기본타입의 박싱 클래스다.
- 참조 타입 필드면서, 이 클래스의 equals 메소드가 이 필드의 equals를 재귀적으로 호출하여 비교한다면, 이 필드의 hashCode를 재귀적으로 호출한다.
- 필드가 배열이라면, 핵심 원소 각각을 별도 필드처럼 다룬다.
- 모든 원소가 핵심 원소라면 Arrays.hashCode를 사용한다.
단계 2.1에서 계산한 해시코드 c로 result를 갱신한다.
```
result = 31 * result + c;
```
result를 반환한다.
주의할 점
- equals비교에 사용되는 필드에 대해서만 해시코드를 계산한다.
- 성능을 높인답시고 해시코드를 계산할 때 핵심 필드를 생략해서는 안 된다.
- 만약 hash로 바꾸려는 필드가 기본 타입이 아니면 해당 필드의 hashCode를 불러 구현한다.
- 계산이 복잡한 경우는 표준형을 만들어 구현한다.
- 참조 타입 필드가 null일 경우 0을 사용.
31을 곱하는 이유는 비슷한 필드가 여러개일 때 해시효과를 크게 높여주기 위해서다.
- 비슷한 값들이 여러개 있을때 그냥 더하면 같은 값이 나올 확률이 높다.
- 그래서 31을 곱해 큰수로 만들어 해시효과를 증가시킨다.

전형적인 hashCode 메서드
```java
@Override
    public int hashCode() {
        int result = Integer.hashCode(areaCode);
        result = 31 * result + Integer.hashCode(prefix);
        result = 31 * result + Integer.hashCode(lineNum);
        return result;
    }
```
- PhoneNumber 인스턴스의 핵심 필드 3개를 사용해 간단한 계산을 수행하고, 이 과정에 비결정적 요소는 없다.
- 동치인 PhoneNumber 인스턴스는 서로 같은 해시코드를 가질 것이 확실하다.

- Objects 클래스의 hashCode 메서드 - 성능이 살짝 아쉽다.
```java
@Override
public int hashCode() {
    return Objects.hash(lineNum,prefix,areaCode);
}
```
- 입력 인수를 담기 위한 배열이 만들어지고, 입력 중 기본 타입이 있다면 박싱과 언박싱도 거친다. 속도가 느리다.

---

- 클래스가 불변이고 해시코드를 계산하는 비용이 크다면, 매번 새로 계산하기 보다 캐싱을 고려해야 한다.
- 이 타입의 객체가 주로 해시의 키로 사용될 것 같다면 인스턴스가 만들어질 때 해시코드를 계산해 둔다.
- 해시의 키로 사용되지 않는 경우라면 hashCode가 처음 불릴 때 계산하는 지연 초기화하면 좋다.
- 필드를 지연 초기화 하려면 그 클래스가 thread safe가 되도록 동기화에 신경 써야한다.
- **성능을 높인답시고 핵심필드 생략은 절대 안된다.**

---

> equals를 재정의할 때는 hashCode도 반드시 재정의해야 한다. 그렇지 않으면 프로그램이 제대로 동작하지 않을 것이다.
재정의한 hashCode는 Object의 API 문서에 기술된 일반 규약을 따라야 하며, 서로 다른 인스턴스라면 되도록 해시코드도 서로 다르게 구현해야 한다. AutoValue 프레임워크를 사용하면 멋진 equals와 hashCode를 자동으로 만들어준다.

[이펙티브 자바 3판]