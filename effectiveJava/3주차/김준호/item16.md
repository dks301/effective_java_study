# 아이템 16. public 클래스에서는 public 필드가 아닌 접근자 메서드를 사용하라

```java
class Point {
    public double x;
    public double y;
}
```

- 이런 클래스는 데이터 필드에 직접 접근할 수 있으니 캡슐화의 이점을 제공하지 못한다.
- API를 수정하지 않고는 내부 표현을 바꿀 수 없다.
- 불변식 보장 x
- 외부에서 필드에 접근할 때 부수 작업 수행 불가.

```java
public class Point {
  private double x;
  private double y;

  public Point2(double x, double y) {
    this.x = x;
    this.y = y;
  }

  public double getX() {
    return x;
  }

  public double getY() {
    return y;
  }

  public void setX(double x) {
    this.x = x;
  }
  public void setY(double y) {
    this.y = y;
  }
}
```

- public 클래스라면 이 방식이 확실히 맞다.

- package-private 클래스 혹은 private 중첩 클래스라면 데이터 필드를 노출해도 문제가 되지 않는다.

- public 클래스의 필드가 불변이라면 직접 노출할 때의 단점은 조금은 줄어들지만 좋은 생각이 아니다.

- API의 노출을 함부로 바꿀 수 없고, 필드를 접근할 때, 부수 작업을 수행할 수 없는 단점은 여전하다.

> public 클래스는 절대 가변 필드를 직접 노출해서는 안된다. 불변 필드라면 노출해도 덜 위험하지만 완전히 안심할 수는 없다. 하지만 package-private 클래스나 private 중첩 클래스에서는 종종 `불변이든 가변이든` 필드를 노출하는 편이 나을 때도 있다.


[이펙티브 자바 3판]