## 아이템 83. 지연 초기화는 신중히 사용하라

다른 모든 최적화와 마찬가지로 지연 초기화에 대해 해줄 조언은 필요할때까지 하지마라.

대부분의 상황에서 일반적인 초기화가 지연 초기화 보다 낫다.

```java
private final FieldType field1 = computeFieldValue();
```
인스턴스 필드를 초기화하는 일반적인 방법

```java
private FieldType field;

private synchronized FieldType getField() {
  if (field == null)
    field = computeFieldValue();
  return field;
}
```
지연 초기화가 초기 순환성을 깨뜨릴 것 같으면 synchronized를 단 접근자를 사용하자.

```java

private static class FieldHolder {
  static final FieldType field = computeFieldValue();
}

private static FieldType getField() { return FieldHolder.field; }
```
성능 때문에 정적 필드를 지연 초기화해야 한다면 지연 초기화 홀더 클래스 관용구를 사용하자.

```java

private volatile FieldType field;

private FieldType getField() {
  FieldType result = field;
  if (result != null)    // 첫 번째 검사 (락 사용 안 함)
    return result;

  synchronized(this) {
    if (field == null) // 두 번째 검사 (락 사용)
      field = computeFieldValue();
    return field;
  }
}
```
성능 때문에 인스턴스 필드를 지연 초기화해야 한다면 이중검사 관용구를 사용하자.

```java

private volatile FieldType field;

private FieldType getField() {
  FieldType result = field;
  if (result == null)
    field = result = computeFieldValue();
  return result;
}
```
단일 검사 관용구로는 초기화가 중복해서 일어날수 있다.

