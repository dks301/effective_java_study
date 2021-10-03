# 2장. 객체 생성과 파괴

> GOAL  
> 1. 객체를 만들어야 할 때와 만들지 말아야 할 때를 구분하기  
> 2. 올바른 객체 생성 방법과 불필요한 생성을 피하는 방법  
> 3. 제때 파괴됨을 보장하고, 파괴 전에 수행해야 할 정리 작업을 관리하기

# 아이템8. finalizer와 cleaner 사용을 피하라

자바는 두 가지 객체 소멸자를 제공한다.  
그 중 finalizer는 예측할 수 없고, 상황에 따라 위험할 수 있어 일반적으로 불필요하다.  
오동작, 낮은 성능, 이식성 문제 등등..

자바 9에서는 finalizer를 사용 자제로 지정하고, cleaner를 그 대안으로 소개한다.  
cleaner는 finalizer보다는 덜 위험하지만, 여전히 예측할 수 없고, 느리고, 일반적으로 불필요하다.

```java
public class FinalizeExample {
    @Override
    protected void finalize() throws Throwable {
        super.finalize(); // 이게 언제 실행될지 예측이 안된다. 
    }
 }
```

## 단점1. 즉시 수행된다는 보장이 없다.

finalizer와 cleaner로는 제때 실행되어야 하는 작업은 절대 할 수 없다.  
파일 닫기를 finalizer와 cleaner에 맡기면 중대한 오류를 일으킬 수 있다. 시스템이 동시에 열 수 있는 파일 개수에 한계가 있기 때문이다. 시스템이 finalizer와 cleaner 실행을 게을리 해서 파일을 계속 열어둔다면, 새로운 파일을 열지 못해 프로그램이 실패할 수 있다. finalizer와 cleaner가 얼마나 신속히 수행될지는 가비지 컬렉터 알고리즘에 달려있다.

## 단점 2. 인스턴스 반납을 지연시킬 수도 있다.

fianlize 스레드는 우선순위가 낮아서, 언제 실행될지 모른다. Finalize에 어떤 작업이 있고, 쓰레드가 처리를 못해서 대기하고 있다면, 해당 인스턴스는 GC가 되지않고 계속 쌓이다가 `OutOfMemoryException`이 발생할 수도 있다.

Cleaner는 자신을 수행할 스레드를 제어할 수 있다는 면에서 조금 낫다. 하지만 여전히 가비지 컬렉터의 통제하에 있으니 즉각 수행되리란 보장은 없다.

## 단점 3. 실행하지 않을 수도 있다.

접근할 수 없는 일부 객체에 딸린 종료 작업을 전혀 수행하지 못한 채 프로그램이 중단 될 수도 있다.  
상태를 영구적으로 수정하는 작업에서는 절대 finalizer나 cleaner에 의존해서는 안된다.

`System.gc`나 `System.runFinalization` 메서드에 속지말자. 실행이 보장되지 않는다.  
이를 보장해주겠다는 메서드가 2개 있었는데 `System.runFinalizersOnExit`와 `Runtime.runFinalizersOnExit`이다. 하지만 심각한 결함이 있다.

## 단점 4. 심각한 성능 문제도 있다.

finalizer 를 사용한 객체를 생성하고 파괴하니 50배나 느렸다. finalizer가 GC의 효율을 떨어뜨리기 때문이다. cleaner도 비슷하다.

## 단점 5. finalizer를 사용한 클래스는 finalizer 공격에 노출되어, 심각한 보안 이슈를 일으킬 수 있다.

생성자나 직렬화과정에서 예외가 발생하면, 생성되다 만 객체에서 악의적인 하위 클래스의 finalizer가 수행될 수 있게 된다. 해결법으로 finalizer메서드를 final로 선언하면 된다.

```java
@Override
protected final void finalize() throws Throwable {
    super.finalize(); // 이게 언제 실행될지 예측이 안된다. 
}
```

## 자원 반납하는 방법

finalizer와 cleaner를 대신해줄 묘안은 `AutoCloseable`을 구현하는 것이다.  
클라이언트에서 인스턴스를 다쓰고 나면 close 메서드를 호출하면 된다.

```java
public class SampleResource implements AutoCloseable {
    @Override
    public void close() throw RunTimeException {
        System.out.println("close");
    }

    public void hello() {
        System.out.println("hello");
    }
}

public class SampleRunner {
    public static void main(String[] args) {
        try{
            SampleResource sampleResource = new SampleResource();
            sampleResource.hello();
        } finally {
            if(sampleResouse != null)
                sampleResource.close();
        }
    }
}

// 이렇게 하면 명시적으로 close 호출 안해줘도 된다. 
public class SampleRunner {
    public static void main(String[] args) {
        try(SampleResource sampleResource = new SampleResource()) {
            sampleResource.hello();
        } 
    }
}
```

그럼 대체 cleaner와 finalizer는 대체 어디에 쓰는 물건일까.

1\. 자원의 소유자가 close 메서드를 호출하지 않는 것에 대비한 안전망 역할이다.  
즉시 호출되리라는 보장은 없지만, 클라이언트가 하지 않은 자원 회수를 늦게라도 해주는 것이 아예 안하는 것보단 낫다.  
자바에서 일부 클래스는 안전망 역할의 finalizer를 제공한다. `FileInputStream`, `FileOutputStream`, `ThreadPoolExecutor`가 대표적이다.

```java
public class SampleResource implements AutoCloseable {

    private boolean closed;

    @Override
    public void close() throw RunTimeException {
        if(this.closed) {
            throw new IlealStateException();
    }

    public void hello() {
        System.out.println("hello");
    }

    @Override
    protected void finalize() throws Throwable {
        if(!this.closed)  close(); // 이렇게 안전망 역할
    }
}
```

2\. 네이티브 피어 정리할 때 쓰기  
네이티브 피어 : 일반 자바 객체가 네이트브 메서드를 통해 기능을 위임한 네이트브 객체  
네이티브 피어는 자바 객체가 아니라서 , GC는 존재를 알지 못한다. 그 결과 자바 피어를 회수할 때 네이티브 객체까지 회수하지 못한다. cleaner나 finalizer가 나서서 처리하기에 적당한 작업이다. 하지만 중요한 리소스인 경우에는 `close`메서드를 사용하자

## Cleaner 구현

```java
public class Room implements AutoCloseable{

    private static final Cleaner cleaner = Cleaner.create();

    //수거 대상이 되면 방을 청소한다. 
    private final Cleaner.Cleanable cleanable;

    //방의 상태. cleanable과 공유한다.
    private final State state;

    public SampleResource(final int numJunkFiles) {
        this.resourceCleaner = new ResourceCleaner(numJunkFiles);
        this.cleanable = cleaner.register(this, state); // Runnable 객체를 등록
    }

    private static class State implements Runnable {
        int numJunkPiles; // 방안의 쓰레기 수

        State(final int numJunkPiles) {
            this.numJunkPiles = numJunkPiles;
        }

        //close 메서드나 cleaner가 호출한다.
        //cleanable에 의해 딱 한번 실행된다.
        @Override
        public void run() { 
            System.out.println("방청소");
            numJunkPiles = 0;
        }
    }

    @Override
    public void close() throws Exception {
        cleanable.clean();
    }
}
```

## 결론

cleaner는 안전망 역할이나, 중요하지 않은 네이티브 자원 회수용으로만 사용하자.  
물론 불확실성, 성능 저하 주의할 것!

## 참고

자료는 이펙티브 자바 책과 백기선님의 강의를 들으며 작성했다.  
[\[이펙티브자바 - 아이템8\] finalizer와 cleaner 사용을 피하라](https://youtu.be/sdPdpMYqW_k)