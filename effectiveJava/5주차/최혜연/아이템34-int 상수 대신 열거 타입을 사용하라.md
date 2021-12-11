# 6장. 열거 타입과 애너테이션 - GOAL

> 자바에는 특수한 목적의 참조 타입이 두가지가 있다.  
> 하나는 클래스의 일종인 열거 타입(enum), 다른 하나는 인터페이스 일종인 애너테이션(annotation)이다.
> 이 타입들을 올바르게 사용하는 방법을 알아보자!

## 아이템34. int 상수 대신 열거 타입을 사용하라
열거 타입은 일정 개수의 상수 값을 정의한 다음, 그 외의 값은 허용하지 않는 타입이다.
자바에서 열거 타입을 지원하기 전에는 아래 처럼 정수 상수를 한 묶음 선언해서 사용하곤 했다.

```java
public static final int APPLE_FUJI = 0;
public static final int APPLE_PIPPIN = 1;
public static final int APPLE_GRANNY_SMITH = 2;

public static final int ORANGE_NAVEL = 0;
public static final int ORANGE_TEMPLE = 1;
public static final int ORANGE_BLOOD = 2;
```

위의 방식은 `정수 열거 패턴`이라 불리는데, 단점이 많다. 
- 타입 안정을 보장할 수 없고 표현력도 좋지 않다.
- 오렌지로 건네야할 메서드에 사과를 보내고, 서로 비교해도 컴파일러는 경고 메시지를 출력하지 못한다.
  `APPLE_FUJI == ORANGE_NAVEL;`
- 문자열로 출력하기도 까다롭다 (의미가 아닌 숫자로만 보인다.)
- 같은 정수 열거 그룹에 속한 모든 상수를 한 바퀴 순회하는 방법도 마땅치 않다.
- 심지어 그 안에 상수가 몇 개인지도 알 수 없다.

## 열거 타입
```java
public enum Apple { FUJI, PIPPIN, GRANNY_SMITH }
public enum Orange { NAVEL, TEMPLE, BLOOD }
```
### 열거 타입 특징
- 열거 타입 자체는 클래스이다. 
- 상수 하나당 자신의 인스턴스를 하나씩 만들어 public static final 필드로 공개한다.
- 열거 타입은 밖에서 접근할 수 있는 생성자를 제공하지 않으므로 사실상 final이다.
- 클라이언트가 인스턴스를 직접 생성하거나, 확장할 수 없으니, 열거 타입 선언으로 만들어진 인스턴스들은 딱 하나씩만 존재함이 보장된다.
- 열거 타입은 인스턴스 통제된다. (싱글턴을 일반화한 형태)
- 열거 타입은 컴파일타임 타입 안정성을 제공한다.
   - Apple을 매개변수로 받는 메서드로 선언했다면, 건네받은 참조는 Apple의 세 가지 값 중 하나임이 확실하다.
   - 다른 값을 넘기려하면 컴파일 오류가 난다.
- 열거 타입에는 각자의 이름 공간이 있어서 이름이 같은 상수도 평화롭게 공존한다.
- 열거 타입에 새로운 상수를 추가하거나, 순서를 바꿔도 다시 컴파일하지 않아도 된다.
   - 공개되는 것이 오직 필드의 이름뿐이라, 정수 열거 패턴과 달리 상수 값이 클라이언트로 컴파일되어 각인되지 않기 때문이다.
- 열거 타입의 toString 메서드는 출력하기에 적합한 문자열을 내어준다.
- 임의의 메서드나 필드를 추가할 수 있고, 임의의 인터페이스를 구현할 수도 있다.

### 데이터와 메서드를 갖는 열거 타입

```java

@Getter
public enum Planet {
  MERCURY(3.302e+23, 2.439e6),
  VENUS(4.869e+24, 6.052e6),
  EARTH(5.975e+24, 6.378e6),
  MARS(6.419e+23, 3.393e6),
  JUPITER(1.899e+27, 7.149e7),
  SATURN(5.685e+26, 6.027e7),
  URANUS(8.683e+25, 2.556e7),
  NEPTUNE(1.024e+26, 2.447e7);

  private final double mass;            // 질량(단위: 킬로그램)
  private final double radius;          // 반지름(단위: 미터)
  private final double surfaceGravity;  // 표면중력(단위: m / s^2)

  // 중력상수 (단위: m^3 / kg s^2)
  private static final double G = 6.67300E-11;

  // 생성자
  Planet(double mass, double radius) {
    this.mass = mass;
    this.radius = radius;
    this.surfaceGravity = G * mass / (radius * radius);
  }

  public double getMass() {
    return mass;
  }

  public double getRadius() {
    return radius;
  }

  public double getSurfaceGravity() {
    return surfaceGravity;
  }

  public double surfaceWeight(double mass) {
    return mass * surfaceGravity;
  }
}


public class WeightTable {
  public static void main(String[] args) {
    double earthWeight = Double.parseDouble(args[0]);
    double mass = earthWeight / Planet.EARTH.getSurfaceGravity();
    for(Planet p : Planet.values()){
        System.out.println(p + "에서의 무게는 " + p.surfaceWeight(mass) + "이다.\n");
    }
  }
  // NEPTUNE 에서의 무게는 69.912739이다.
  // ...
}
```
## 열거 타입 쓸 때 조언
- 열거 타입 상수 각각을 특정 데이터와 연결지으려면, 생성자에서 데이터를 받아 인스턴스 필드에 저장하며 된다.
- 열거 타입은 근본적으로 불변이라 모든 필드는 `final` 이어야 한다. 필드를 public으로 선언해도 되지만, private으로 두고, public접근자 메서드를 두는 게 낫다.
- 널리 쓰이는 열거 타입은 톱레벨 클래스로!
- 특정 톱레벨 클래스에서만 쓰인다면, 해당 클래스의 멤버 클래스로 만든다.

## 값에 따라 분기하는 열거 타입 - 만족하는가?
```java
public double apply(double x, double y) {
    switch(this) {
        case PLUS: return x + y;
        case MINUS: return x - y;
        case TIMES: return x * y;
        case DIVIDE: return x / y;
    }
    throw new AssertionError("알 수 없는 연산: " + this);
}
```
## 상수별 메서드 구현을 활용한 열거 타입
아래 처럼 하면, apply 메서드가 상수 선언 바로 옆에 붙어 있으니, 새로운 상수를 추가할 때 apply도 재정의해야 한다는 사실을 깜빡하기는 어렵다.
apply가 추상메서드 이므로, 재정의하지 않았다면 컴파일 오류로 알려준다.
```java
public enum Operation {
    PLUS{
        public double apply(double x, double y) {
            return x + y;
        }
    }, 
    MINUS{
        public double apply(double x, double y) {
            return x - y;
        }
    }, 
    TIMES{
        public double apply(double x, double y) {
            return x * y;
        }
    }, 
    DIVIDE{
        public double apply(double x, double y) {
            return x / y;
        }
    };
    
    public abstract double apply(double x, double y);    
}
```

## 전략 열거 타입 패턴
```java
enum PayrollDay {
    MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY, SATURDAY, SUNDAY;
    
    private static final int MINS_PER_SHIFT = 8 * 60;
    
    int pay(int minutesWorked, int payRate) {
        int basePay = minutesWorked * payRate;
        int overtimePay;
        switch(this) {
            case SATURDAY:
            case SUNDAY:
                overtimePay = basePay / 2;
                break;
            default:
                overtimePay = minutesWorked <= MINS_PER_SHIFT ? 0 : 
                        (minutesWorked - MINS_PER_SHIFT) * payRate / 2;
        }
        return basePay + overtimePay;
    }
}
```
위 코드는 관리 관점에서 위험한 코드다.
- 새로운 열거 타입을 추가할 때 case 문을 잊지않고, 쌍으로 넣어줘야한다.
- 가독성도 떨어지고, 오류 발생 가능성도 높아진다.

```java
enum PayrollDay {
    MONDAY(WEEKDAY),
    TUESDAY(WEEKDAY),
    WEDNESDAY(WEEKDAY),
    THURSDAY(WEEKDAY),
    FRIDAY(WEEKDAY),
    SATURDAY(WEEKEND),
    SUNDAY(WEEKEND);
    
    private final PayType payType;
    
    PayrollDay(PayType payType) {
        this.payType = payType;
    }
    
    int pay(int minutesWorked, int payRate) {
        return payType.pay(minutesWorked, payRate);
    }
    
    enum PayType {
        WEEKDAY {
            int overtimePay(int minutesWorked, int payRate) {
                return minutesWorked <= MINS_PER_SHIFT ? 0 : (minutesWorked - MINS_PER_SHIFT) * payRate / 2;
            }
        },
        
        WEEKEND {
            int overtimePay(int minutesWorked, int payRate) {
                return minutesWorked * payRate / 2
            }
        };
        
        abstract int overtimePay(int minutesWorked, int payRate);
        private static final int MINS_PER_SHIFT = 8 * 60;
        
        int pay(int minutesWorked, int payRate) {
            int basePay = minutesWorked & payRate;
            return basePay + overtimePay(minutesWorked, payRate);
        }
    }
}
```
위 코드는 **전략 열거 타입 패턴**이다. switch 문이나 상수별 메서드 구현이 필요없게 되었다.
switch문보다 복잡하지만, 더 안전하고 유연하다.
switch문은 열거 타입의 상수별 동작을 구현하는데 적합하지 않다. 하지만 기존의 열거 타입의 상수별 동작을 혼합해 넣을 때에는 switch문이 좋은 선택이 될 수 있다. 최소한으로 코드를 변경할 수 있기 때문이다.

## 열거 타입은 언제쓰지?
필요한 원소를 컴파일 타임에 다 알 수 있는 상수 집합이라면 항상 열거 타입을 사용하자.
EX) 태양계 행성, 한 주의 요일, 체스 말, 메뉴 아이템, 명령 줄 플래그 등

열거 타입에 정의된 상수 개수가 영원히 고정 불변일 필요는 없다.

## 결론
열거 타입은 확실히 정수 상수보다 뛰어나다.
대대수 열거 타입이 명시적 생성자나 메서드 없이 쓰이지만, 각 상수를 특정 데이터와 연결짓거나 상수마다 다르게 동작하게 할 때는 필요하다.
드물게는 하나의 메서드가 상수별로 다르게 동작해야 할 때도 있다. 이때는 `switch`대신 상수별 메서드 구현을 사용하자.
열거 타입 상수 일부가 같은 동작을 공유한다면, 전략 열거 타입 패턴을 사용하자
