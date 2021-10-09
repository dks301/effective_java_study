# 4장. 클래스와 인터페이스 - GOAL

> 추상화의 기본 단위인 클래스와 인터페이스는 자바의 심장이다.
> 클래스와 인터페이스를 쓰기 편하고, 견고하며, 유연하게 만들어보자


## 아이템16. public 클래스에서는 public 필드가 아닌 접근자 메서드를 사용하라
이따금 인스턴스 필드들을 모아놓는 일 외에는 아무 목적도 없는 퇴보한 클래스를 작성하려 할 때가 있다.

### 퇴보한 클래스
```java
class Point {
    public double x;
    public double y;
}
```

이런 클래스는 데이터 필드에 직접 접근할 수 있으니, 캡슐화의 이점을 제공하지 못한다.
API를 수정하지 않고는 내부 표현을 바꿀 수 없고, 불변식을 보장할 수 없으며, 외부에서 필드에 접근할 때 부수 작업을 수행할 수도 없다.
철저한 객체 지향 프로그래머는 이런 클래스를 상당히 싫어해서 필드들을 모두 private으로 바꾸고, public접근자를 추가한다.

### 올바른 데이터 캡슐화
아래는 접근자와 변경자(mutator) 메서드를 활용해 데이터를 캡슐화한다.

```java
class Point {
  private double x;
  private double y;

  public Point(double x, double y) {
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

public 클래스에서라면 이 방식이 확실히 맞다.
패키지 바깥에서 접근할 수 있는 클래스라면 접근자를 제공함으로써 클래스 내부 표현 방식을 언제든 바꿀 수 있는 유연성을 얻을 수 있다.
public 클래스가 필드를 공개하면 이를 사용하는 클라이언트가 생겨날 것이므로, 내부 표현 방식을 마음대로 바꿀 수 없게 된다.


### package-private 클래스 또는 private 중첩 클래스
package-private 클래스 또는 private 중첩 클래스라면 데이터 필드를 노출한다 해도 하등의 문제가 없다.
클래스가 표현하려는 추상 개념만 올바르게 표현해주면 된다.
같은 패키지 안에서 어떤 특정 이유 때문에 사용하던가 탑 레벨 클래스에서만 접근하기 때문에 위에서 언급한 단점들이 나타날 이유가 없어 문제될 것이 없다.


참고.

package-private : 같은 패키지 및 같은 클래스에서만 접근 가능<br>
private 중첩 클래스 : 내부 클래스가 private 형태로 된 클래스

### 불변 필드를 노출한 public 클래스 - 과연 좋을까?
public 클래스의 필드가 불변이라면 직접 노출할 때의 단점이 조금은 줄어들지만, 결코 좋은 생각은 아니다.
API를 변경하지 않고는 표현 방식을 바꿀 수 없고, 필드를 읽을 때 부수 작업을 수행할 수 없다는 단점은 여전하다.
단, 불변식은 보장할 수 있게 된다.

```java
public final class Time {
    private static final int HOURS_PER_DAY = 24;
    private static final int MINUTES_PER_HOUR = 60;

    public final int hour;
    public final int minute;

    public Time(int hour, int minute) {
        if (hour < 0 || hour >= HOURS_PER_DAY) {
            throw new IllegalArgumentException("시간: " + hour);
        }
        if (minute < 0 || minute >= HOURS_PER_HOUR) {
            throw new IllegalArgumentException("분: " + minute);
        }        
        this.hour = hour;
        this.minite = minute;    
    }
}
```

## 결론
public 클래스는 절대 가변 필드를 직접 노출해서는 안된다. 
불변 필드라면 노출해도 덜 위험하지만 안심할 순 없다.
package-private 클래스나 private 중첩 클래스에서는 종종(가변이든 불변이든) 필드를 노출하는 편이 나을떄도 있다.

