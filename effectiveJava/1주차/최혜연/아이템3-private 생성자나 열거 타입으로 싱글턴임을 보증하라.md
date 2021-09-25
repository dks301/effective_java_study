# 2장. 객체 생성과 파괴

> GOAL  
> 1. 객체를 만들어야 할 때와 만들지 말아야 할 때를 구분하기  
> 2. 올바른 객체 생성 방법과 불필요한 생성을 피하는 방법  
> 3. 제때 파괴됨을 보장하고, 파괴 전에 수행해야 할 정리 작업을 관리하기

# 아이템3. private 생성자나 열거 타입으로 싱글턴임을 보증하라

싱글턴(singleton)이란 인스턴스를 오직 하나만 생성할 수 있는 클래스를 말한다.  
예시로, 함수와 같은 무상태(stateless) 객체나 설계상 유일해야 하는 시스템 컴포넌트를 들 수 있다.  
단점으로는, 클래스를 싱글턴으로 만들면 이를 사용하는 클라이언트를 테스트하기가 어렵다는 점이다. 싱글톤이 인터페이스를 구현하게 아니라면, mock으로 교체하는게 어렵기 때문이다.

아래는 싱글톤 패턴 예시 코드다.

```java
public final class Singleton {
 private static volatile Singleton instance = null;

 private Singleton() { }

 public static Singleton getInstane() {
     if(instance == null){
        syschronized(Singleton.class) {
            if( instance == null) {
                instance = new Singleton();
            }
        }
    }
 }
}
```

싱글턴을 만드는 방식은 3가지가 있다.

## 1\. public static final 필드 방식

private 생성자는 public static final 필드인 Elvis.INSTANCE 를 초기화할 때 딱 한번만 호출된다.

```java
public clas Elvis {
    public static final Elvis INSTANCE = new Elvis();
    private Elvis() { }

    public void leaveTheBuilding{ }
}
```

### 장점

1.  해당 클래스가 싱글턴임이 API에 명백히 드러난다.  
    public static 필드가 final이니 절대로 다른 객체를 참조할 수 없다.
2.  간결함

## 2\. 정적 팩토리 메서드 방식

Elvis.INSTANCE 는 항상 같은 객체의 참조를 반환하므로 제 2의 인스턴스는 절대 만들어지지 않는다.

```java
public clas Elvis {
    public static final Elvis INSTANCE = new Elvis();
    private Elvis() { }

    public staic Elvis getInstance() { return INSTANCE; }
    public void leaveTheBuilding{ }
}
```

### 장점

1.  API를 바꾸지 않고도 싱글턴이 아니게 변경할 수 있다.
2.  원한다면 정적 팩터리를 제네릭 싱글턴 팩터리로 만들 수 있다.
3.  정적 팩터리의 메서드 참조를 공급자로 사용할 수 있다.

위 방법 모두 직렬화에 사용한다면, 역직렬화 할 때 같은 타입의 인스턴스가 여러개 생길 수 있다.

그 문제를 해결하려면 모든 인스턴스 필드에 trasient를 추가하고 readReslove메서드를 구현해야 한다.

```java
private static final transient Singleton instance = new Singleton();
```

자바 직렬화 : implements Serializable

```java
// 싱글턴임을 보장해주는 readResolve 메서드
private Object readResolve() {
     // 진짜 Elvis를 반환하고, 가짜 Elvis는 가비지 컬렉터에 맡긴다.
    return INSTANCE;
}
```

### 3\. 열거 타입 방식

더 간결하고, 추가 노력 없이 직렬화 할 수 있다.  
리플렉션 공격에서도 제2의 인스턴스가 생기는 일을 완벽히 막아준다.  
원소가 하나뿐인 열거타입이 싱글턴을 만드는 가장 좋은 방법이다.

```java
public enum Elvis {
    INSTANCE;
}
```

### 참고

자료는 이펙티브 자바 책과 백기선님의 강의를 들으며 작성했다.  
[\[이펙티브자바 - 아이템3\] private 생성자나 열거 타입으로 싱글턴임을 보증하라](https://youtu.be/xBVPChbtUhM)