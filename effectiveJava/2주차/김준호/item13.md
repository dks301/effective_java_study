# 아이템 13. clone 재정의는 주의해서 진행해라

클래스에서 clone을 재정의 하기위해선 해당 클래스에 Cloneable 인터페이스를 상속받아 구현하여야 한다. 그런데 정작 clone 메소드는 Cloneable 인터페이스가 아닌 Object에 선언되어있다. Cloneable 인터페이스에는 아무것도 선언되어있지 않은 빈 인터페이스이다. 그렇다면 `Cloneable 인터페이스는 무슨 일을 할까?`

### Cloneable 인터페이스의 역할

- Cloneable 인터페이스는 상속받은 클래스가 복제해도 되는 클래스임을 명시하는 용도의 믹스인 인터페이스이다.  Cloneable 인터페이스의 역할은 Object의 clone의 동작 방식을 결정한다. Cloneable을 상속한 클래스의 clone 메소드를 호출하면 해당 클래스를 필드 단위로 복사하여 반환한다. 만약, Cloneable을 상속받지 않고 clone 메소드를 호출하였다면 CloneNotSupportedExcetion 을 던진다. 

- 믹스인이란 클래스가 본인의 기능 이외에 추가로 구현할 수 있는 자료형으로, 어떤 선택적 기능을 제공한다는 사실을 선언하기 위해 쓰인다.
### Object clone 메소드의 일반 규약

- x.clone() != x 는 참이다. 원본 객체와 복사 객체는 서로 다른 객체이다.
- x.clone().getClass() == x.getClass() 는 참이다. 하지만 반드시 만족해야 하는 것은 아니다.
- x.clone().equals(x) 는 참이지만 필수는 아니다.
- x.clone().getClass() == x.getClass(), super.clone()을 호출해 얻은 객체를 clone 메소드가 반환한다면, 이 식은 참이다. 관례상, 반환된 객체와 원본 객체는 독립적이어야 한다. 이를 만족하려면 super.clone으로 얻은 객체의 필드 중 하나 이상을 반환 전에 수정해야 할 수도 있다.
- clone 메소드가 super.clone 이 아닌 생성자를 호출해 얻은 인스턴스를 반환 하더라도 컴파일시에 문제가 되지않지만 해당 클래스의 하위 클래스에서 super.clone을 호출한다면 하위 클래스 타입 객체를 반환하지 않고 상위 클래스 타입 객체를 반환하여 문제가 생길 수 있다.

---

- clone 메소드를 재정의 하기 위해서 Cloneable 인터페이스를 상속한다. Object의 clone 메소드는 Checked Exception인 CloneNotSupportedException을 처리하도록 선언되어있다. clone 메소드 사용을 편하게 하기위해 throws 절을 try-catch 문으로 바꿔 clone메소드에서 예외 처리하자.

---

`새로운 인터페이스를 만들 때는 절대로 Clonable을 상속해서는 안된다. 복제 기능은 생성자와 팩토리를 이용하는 것이 최고이다. 단, 배열만은 clone 메서드 방식이 가장 깔끔하다.`

[이펙티브 자바 3]