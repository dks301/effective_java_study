## 아이템 80. 스레드보다는 실행자, 태스크, 스트림을 애용하라

- 실행자 프레임워크

java.util.concurrent 패키지에는 인터페이스 기반의 유연한 태스크 실행 기능을 담은 실행자 프레임워크(Executor Framework)가 있다. 과거에는 단순한 작업 큐(work queue)를 만들기 위해서 수많은 코드를 작성해야 했는데, 이제는 아래와 같이 간단하게 작업 큐를 생성할 수 있다.

```java
ExecutorService executorService = Executors.newSingleThreadExecutor();

exec.execute(runnable);

exec.shutdown();
```
실행자 프레임워크는 아래와 같은 주요 기능을 가지고 있다.

- 특정 태스크가 완료되기를 기다릴 수 있다.

- 태스크 모음 중에서 어느 하나(invokeAny) 혹은 모든 태스크(invokeAll)가 완료되는 것을 기다릴 수 있다.

- 실행자 서비스가 종료하기를 기다린다. 

- 완료된 태스크들의 결과를 차례로 받는다.  (ExecutorCompletionService)

- 태스크를 특정 시간에 혹은 주기적으로 실행하게 한다.  (ScheduledThreadPoolExecutor)

> 스레드를 직접 다루지 말자
작업 큐를 직접 만들거나 스레드를 직접 다루는 것도 일반적으로 삼가야 한다. 스레드를 직접 다루지말고 실행자 프레임워크를 이용하자. 그러면 작업 단위와 실행 매커니즘을 분리할 수 있는다. 작업 단위는 Runnable과 Callable로 나눌 수 있다. Callable은 Runnable과 비슷하지만 값을 반환하고 임의의 예외를 던질 수 있다. 자바 7부터 실행자 프레임워크는 포크-조인(fork-join) 태스크를 지원하도록 확장됐다. ForkJoinTask의 인스턴스는 작은 하위 태스크로 나뉠 수 있고 ForkJoinPool을 구성하는 스레드들이 이 태스크들을 처리하며, 일을 먼저 끝낸 스레드가 다른 스레드의 남은 태스크를 가져와 대신 처리할 수도 있다.

