# 2장. 객체 생성과 파괴

> GOAL  
> 1. 객체를 만들어야 할 때와 만들지 말아야 할 때를 구분하기  
> 2. 올바른 객체 생성 방법과 불필요한 생성을 피하는 방법  
> 3. 제때 파괴됨을 보장하고, 파괴 전에 수행해야 할 정리 작업을 관리하기

# 아이템4. 인스턴스화를 막으려거든 private 생성자를 사용하라

정적 메서드와 정적 필드만을 담은 클래스를 만들고 싶을 때가 있을 것이다.

추상 클래스로 만드는 것으로는 인스턴스화를 막을 수 없다.

`UtilClass utilClass = new UtilClass()` 이것을 막기 위해 `public abstact class UtilClass`로 선언하면 1차적으로는 막을 수 있다.

```java
public class UtilClass {
    public static String getName() {
        reutrn "hyeyeon";
    }
    public static void main(String[] args) {
        UtilClass utilClass = new UtilClass(); 
    }
}
```

하지만 어떤 클래스가 UtilCLass를 상속받으면, 인스턴스를 만들 수 있다.

```java
static class AnotherClass extends UtilClass {

}
 public static void main(String[] args) {
     AnotherClass anotherCLass = new AnotherClass(); 
 }
```

## 인스턴스화를 막는 방법

컴파일러가 기본생성자를 만드는 경우는 오직 명시된 생성자가 없을 때 뿐이니,`private 생성자` 를 추가하면 된다.

```java
public class UtilityClass {
    //기본 생성자가 만들어지는 것을 막는다 (인스턴스화 방지용)
    private UtilityClass() {
        throw new AssertionError();
    }
}
```

위 코드는, 명시적 생성자가 private 이니 클래스 바깥에서 접근할 수 없다.  
상속을 불가능하게 하는 효과도 있다.  
모든 생성자는 명시적이든 묵시적이든 상위 클래스의 생성자를 호출하게 되는데, 이를 private로 선언했으니 하위 클래스가 상위 클래스의 생성자에 접근할 길이 막혀버린다.

### 참고

자료는 이펙티브 자바 책과 백기선님의 강의를 들으며 작성했다.  
[\[이펙티브자바 - 아이템4\] 인스턴스화를 막으려거든 private 생성자를 사용하라](https://youtu.be/A-t1T3_m15M)