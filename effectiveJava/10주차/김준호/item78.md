## 아이템 78. 공유 중인 가변 데이터는 동기화해 사용하라

- ~~long, double 외의 변수를 읽고 쓰는 동작은 원자적이다.~~
- read from memory -> update in cpu -> write in memory

- 동기화는 **배타적 실행**과 **스레드 사이의 안정적인 통신**에 꼭 필요하다.


```java
public class StopThread {

    private static boolean stopRequested;

    public static void main(String[] args) throws InterruptedException {

        Thread thread = new Thread(() -> {
            int i = 0;
            while (!stopRequested) {
                i++;
            }
        });

        thread.start();

        TimeUnit.SECONDS.sleep(1);
        stopRequested = true;
    }
}
```

```java
public class StopThread {
    private static boolean stopRequested;

    public StopThread() {
    }

    public static void main(String[] args) throws InterruptedException {
        Thread thread = new Thread(() -> {
            for(int var0 = 0; !stopRequested; ++var0) {
            }

        });
        thread.start();
        TimeUnit.SECONDS.sleep(1L);
        stopRequested = true;
    }
}
```

- 무한루프
- volatile로 메모리에서 읽어오거나, synchronized로 동기화.


```java
    public class SynchronizedStopThread {
    private static boolean stopRequested;

    private static synchronized void requestStop() {
        stopRequested = true;
    }

    private static synchronized boolean stopRequested() {
        return stopRequested;
    }

    public static void main(String[] args) throws InterruptedException {
        Thread backgroundThread = new Thread(() -> {
            int i = 0;
            while (!stopRequested()) {
                i++;
                
            }
        });
        backgroundThread.start();

        TimeUnit.SECONDS.sleep(1);
        requestStop();
    }
}
```

- 쓰기 메서드와 읽기 메서드를 둘 다 동기화했음을 주목하자.

- 동시 update에서는 synchronized 써야하고, volatile은 한 스레드는 write, 한 스레드는 read에서만.

- synchronized 비싸니깐(blocked) AtomicInteger 쓰자.

> 여러 스레드가 가변 데이터를 공유한다면 그 데이터를 읽고 쓰는 동작은 반드시 동기화해야 한다. 스레드간 공유되는 가변 데이터의 동기화를 하지 않거나 실패하면, 프로그램이 응답 불가 상태에 빠지거나 안전 실패(Safety failure)에 빠질 수 있다. 이러한 문제의 디버깅 난이도는 가장 어려운 축에 속한다. 배타적 실행은 필요없고, 스레드끼리의 통신만 필요하다면 volatile 한정자만으로도 동기화 할 수 있다.
