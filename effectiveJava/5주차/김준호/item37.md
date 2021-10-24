## 아이템 37. ordinal 인덱싱 대신 EnumMap을 이용하라

#### ordinal 메서드가 반환하는 값을 배열 첨자로 이용하는 예

```JAVA
class Herb {
  enum LifeCycle {ANNUAL, PERENNIAL, BIENNIAL}

  final String name;
  final LifeCycle lifeCycle;

  Herb(String name, LifeCycle lifeCycle) {
    this.name = name;
    this.LifeCycle = lifeCycle;
  }

  @Override public String toString() {
    return name;
  }
}

// 이렇게 사용하지 마라!
Herb[] garden = ...;

Set<Herb>[] herbsByType = //Indexed by Herb.Type.ordinal()
  (Set<Herb>[]) new Set[Herb.Type.values.length];

for(int i=0; i<herbsByType.length; i++) {
  herbsByType[i] = new HashSet<Herb>;
}

for(Herb h : garden) {
  herbsByType[h.type.ordinal()].add(h);
}

//결과 출력
for(int i=0;i < herbsByType.length; i++) {
  System.out.printf("%s : %s%n", Herb.Type.values()[i], herbsByType[i]);
}
```
  - 형 안전성을 보장하지 않는다.
  - 배열은 제네릭과 호환되지 않으므로 무점검 형변환이 일어남.

**enum 상수를 어떤 값에 대응시키려 한다면 EnumMap을 사용하자!**

```JAVA
//EnumMap을 사용해 enum 상수별 데이터를 저장하는 프로그램
Map<Herb.Type, Set<Herb>> herbsByType = new EnumMap<Herb.Type, Set<Herb>>(Herb.Type.class);

for(Herb.Type t : Herb.Type.values) {
  herbsByType.put(t, new HashSet<Herb>());
}
for(Herb h : garden) {
  herbsByType.get(h.type).add(h);
}
System.out.println(herbsByType);
```
  - EnumMap의 생성자는 키 자료형을 나타내는 class객체를 인자로 받는다. -> 한정적 자료형 정보

#### 잘못된 예2

```JAVA
// ordinal() 값을 배열의 배열 첨자로 사용

public enum Phase { SOLID, LIQUID, GAS;
  public enum Transition {
    MELT, FREEZE, BOIL, CONDENSE, SUBLIME, DEPOSIT;

    // 행은 from의 ordinal을, 열은 to의 ordinal을 인덱스로 쓴다.
    private static final Transition [][] TRANSITIONS = { // 상전이 테이블
      {null, MELT, SUBLIME},
      {FREEZE, null, BOIL},
      {DEPOSIT, CONDENSE, null}
    };

    // 한 상태에서 다른 상태로의 전이를 반환한다.
    public static Transition from(Phase src, Phase dst) {
      return TRANSITIONS[src.ordinal()][dst.ordinal()];
    }
  }
}
```
- 값이 수정되었을 때 안전하지 않다.
- EnumMap 으로 바꾼 코드

```JAVA
// 중첩 EnumMap을 이용해서 데이터와 enum쌍을 연결했다.
public enum Phase {
  SOLID, LIQUID, GAS;
   public enum Transition {
     MELT(SOLID, LIQUID), FREEZE(LIQUID, SOLID),
     BOIL(LIQUID, GAS), CONDENSE(GAS, LIQUID),
     SUBLIME(SOLID, GAS), DEPOSIT(GAS, SOLID);

     private final Phase src;
     private final Phase dst;

     Transition(Phase src, Phase dst) {
       this.src = src;
       this.dst = dst;
     }

     // 상전이 맵 초기화한다.
     private static final Map<Phase, Map<Phase, Transition>> m = new EnumMap<Phase, Map<Phase, Transition>>(Phase.class);

     static {
       for(Phase p : Phase.values()) {
         m.put(p, new EnumMap<Phase, Transition>(Phase.class));
       }
       for(Transition trans : Transition.values()) {
         m.get(trans.src).put(trans.dst, trans);
       }
     }

     public static Transition from(Phase src, Phase dst) {
       return m.get(src).get(dst);
     }
   }
}
```
  - 이걸 배열로 하면 나중에 enum 상수가 추가되었을 때, 배열을 변경해야한다.  실수하면 프로그램에 문제생긴다.

### 요약
ordinal 값을 배열 첨자로 사용하는 것은 적절하지 않으며, 대신 **EnumMap** 을 써라