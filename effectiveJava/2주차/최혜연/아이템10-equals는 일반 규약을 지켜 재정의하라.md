# 3장. 모든 객체의 공통 메서드 - GOAL

> Object를 상속하는 클래스는 일반 규약에 맞게 재정의해야 하는데, 언제 어떻게 재정의해야 하는지 알아보자

## 아이템10. equals는 일반 규약을 지켜 재정의하라

## equals를 재정의하지 말하야 할 때

equals 메서드는 재정의하기 쉬어보이지만, 함정이 있다. 다음 아래에 해당된다면 재정의하지 말자



**1\. 각 인스턴스가 본질적으로 고유하다.**  
값을 표현하는게 아니라 동작하는 개체를 표현하는 클래스가 해당된다. ex) Thread  
Object의 equals 메서드는 이러한 클래스에 딱 맞게 구현되었다.

**2\. 인스턴스의 '논리적 동치성'을 검사할 일이 없다.**

**3\. 상위 클래스에서 재정의한 equals가 하위 클래스에도 딱 들어맞는다.**  
대부분의 Set 구현체는 abstractSet이 구현한 equals를 상속받아 쓰고, List 구현체들은 AbstractList로 부터, Map구현체들은 AbstractMap으로부터 상속받아 그대로 쓴다.

**4\. 클래스가 private이거나 package-private이고 equals메서드를 호출할 일이 없다.**  
만약 호출할 것 같으면 아래처럼 정의하자!

```java
@Override
public boolean equals(Object o) {
    throw new AssertionError(); //호출 금지!
}
```

## equals를 재정의해야할 때

**논리적 동치성을 확인해야 하는데, 상위 클래스의 equals가 논리적 동치성을 비교하도록 재정의되지 않았을 때**

주로 값 클래스들이 해당한다. (값 클래스 : Integer, String 처럼 값을 표현하는 클래스)

두 값 객체를 equals로 비교하는 프로그래머는 객체가 같은지가 아닌, 값이 같은지를 알고 싶어한다.  
equals가 논리적 동치성을 확인하도록 재정의해두면 가능!

값이 같은 인스턴스가 둘 이상 만들어지지 않음을 보장하는 인스턴스 통제 클래스라면 재정의하지 않아도 된다. (Enum)도 해당됨.  
이때는 논리적으로 같은 인스턴스가 2개 이상 만들어지지 않으니, 논리적 동치성과 객체 식별성이 똑같은 의미가 된다.  
Object의 equals가 논리적 동치성까지 확인해준다고 볼 수 있다.

## equals 메서드는 동치관계를 구현하며, 다음을 만족한다.

-   동치관계란 무엇일까?  
    집합을 서로 같은 원소들로 이뤄진 부분집합으로 나누는 연산이다. 이 부분집합을 동치류(equivalence class: 동치클래스)라 한다.

equals 메서드가 역할을 다하려면, 모든 원소가 같은 동치류에 속한 어떤 원소와도 서로 교환할 수 있어야 한다.  
아래는 동치관계를 만족시키기 위한 다섯 요건이다.

### 반사성 (reflexivity)

-   null이 아닌 모든 참조값 x에 대해, `x.equals(x)`는 true다.
-   객체는 자기 자신과 같아야한다.

### 대칭성(symmertry)

-   null이 아닌 모든 참조 값 x, y에 대해 `x.equals(y)`가 true면, `y.equals(x)`도 true다.
-   두 객체는 서로에 대한 동치 여부에 똑같이 답해야 한다.
-   규약을 어기면, 그 객체를 사용하는 다른 객체들이 어떻게 반응할지 알 수 없다.

```java
public final class CaseInsensitiveString {
    private final String s;

    public CaseInsensitiveString(String s) {
        this.s = Objects.requireNonNull(s);
    }
    // 대칭성 위배!!!
    @Override
    public boolean equals(Object o) {
        if( o instanceof CaseInsensitiveString) {
            return s.equalsIgnoreCase((CaseInsensitiveString) o).s);
        }
        if ( o instanceof String) {
            return s.equalsIgnoreCase((String) o);
        }
    }

    @Override
    public boolean equals(Object o) {
         return o instanceof CaseInsensitiveString && ((CaseInsensitiveString) o).s.equalsIgnoreCase(s);
    }
} 

// 비교 시도
CaseInsensitiveString cis = new CaseInsensitiveString("Hyeyeon");
String s = "Hyeyeon";

/** 대칭성 위반 */
cis.equals(s); // true
s.equals(cis); // false  (String의 equals는 CaseInsensitiveString의 존재를 모른다)
```

### 추이성(transitivity)

-   null이 아닌 모든 참조 값 x, y, z에 대해 `x.equals(y)`가 true이고, `y.equals(z)`가 true 이면, `x.equals(z)`도 true다.
-   리스코프 치환 원칙 : 어떤 타입에 있어 중요한 속성이라면, 그 하위 타입에서도 중요하다.

### 일관성(consistency)

-   null이 아닌 모든 참조 값 x, y에 대해 `x.equals(y)`를 반복해서 호출하면 항상 true를 반환하거나 항상 false를 반환한다.
-   두 객체가 같아면 영원히 같아야한다.
-   equals의 판단에 신뢰할 수 없는 자원이 끼어들게 해서는 안 된다.
-   equals는 항시 메모리에 존재하는 객체만을 사용한 결정적 계산만 수행해야 한다.

### null이 아니다.

-   null이 아닌 모든 참조 값 x에 대해서 `x.equals(null)`은 false다.

```java
// 묵시적 null 검사 - 이게낫다.
@Override
public boolean equals(Object o) {
    // instanceof로 입력 매개변수가 올바른 타입인지 검사
    if(!(o instance of MyType)) 
        return false;
    MyType mt = (MyType) o;
}
```

## 양질의 equals 메서드 구현 방법 - 단계별

1\. == 연산자를 사용해 입력이 자기 자신의 참조인지 확인한다.  
2\. instanceof 연산자로 입력이 올바른 타입인지 확인한다. (Set, LIst, Map...)  
3\. 입력을 올바른 타입으로 형변환한다. ( 이단계는 100% 성공)  
4\. 입력 객체와 자기 자신의 대응되는 '핵심' 필드들이 모두 일치하는지 하나씩 검사한다.  
5\. 다했다면, 자문하라. 대칭적인가? 추이성이 있나? 일관적인가?

```java
@Override
public boolean equals(Object o){
    if( o == this)  // 1번 자기자신의 참조인지 확인
        return true;
    if(!(o instanceof PhoneNUmber)) // 입력이 올바른 타입인지 확인
        return false;
    PhoneNumber pn = (PhoneNumber) o; //입력을 올바른 타입으로 형변환
    return pn.lineNum == lineNum && pn.prefix == prefix && pn.areaCode ==areaCode; // 모두 일치하는지 하나씩 검사
}
```

## 마지막 주의사항!

-   euqals를 재정의할 땐 hashCode도 반드시 재정의하자
-   너무 복잡하게 해결하려 들지 말자.
-   Object 외의 타입을 매개변수로 받는 equals 메서드는 선언하지 말자.  
    아래 예시는 Object.equals 재정의가 아니라, 다중정의(아이템52)이다.
-   `// 입력 타입은 반드시 Object 여야 한다. public boolean equals(MyClass o){ }`
-   equals 테스트 코드도 존재한다.
    -   @AutoValue

## 결론

꼭 필요한 경우가 아니면 equals를 재정의하지 말자.  
재정의를 해야할 할 땐, 핵심필드 모두를 빠짐없이, 다섯가지 규약을 확실히 지켜가며 비교하자!!
