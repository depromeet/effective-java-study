# Item 81. wait와 notify보다는 동시성 유틸리티를 애용하라

이 책의 초판 아이템 50에서는 wait와 notify를 올바르게 사용하는 방법을 안내함

> 책에는 없지만...

-   자바에서 wait()와 notify(), notifyAll() 메서드들은 Object 클래스의 일부로 모든 자바 객체에 존재
-   이들은 공유된 자원에 대한 동시 접근을 조절하는데 사용되는 중요한 도구들
-   이런 메서드들은 주로 동기화 블록(즉, synchronized 키워드를 사용하여 정의된 블록) 내에서 사용되며, 특히 여러 스레드가 동일한 객체에 동시 접근을 시도할 때 그들 사이의 조정을 담당

<br/>

**`wait()`:**

-   이 메서드는 현재의 스레드를 대기 상태로 만듦
-   이 스레드는 다른 스레드가 notify() 또는 notifyAll() 메서드를 호출하여 해당 스레드를 깨울 때까지 계속 대기함

**`notify()`:**

-   이 메서드는 wait()에 의해 대기 상태로 만들어진 스레드 중 하나를 임의로 선택하여 실행 상태로 만듦

**`notifyAll()`:**

-   이 메서드는 wait()에 의해 대기 상태로 만들어진 모든 스레드를 실행 상태로 만듦

<br/>

_올바른 사용 예제_

```java
class SharedResource {
    private int content;
    private boolean available = false;

    public synchronized int get() {
        while (!available) {
            try {
                wait();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        available = false;
        notifyAll();
        return content;
    }

    public synchronized void put(int value) {
        while (available) {
            try {
                wait();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        content = value;
        available = true;
        notifyAll();
    }
}

```

-   wait()와 notifyAll()은 동기화 블록 내에서 사용됨
-   만약 get()이나 put() 메서드를 호출하는 스레드가 자원을 사용할 수 없는 경우, 해당 스레드는 wait()를 통해 대기 상태가 되고, 자원이 사용 가능해지면 notifyAll()에 의해 깨어나게 됨

<br/>

_올바르지 못한 사용 예시_

```java
class SharedResource {
    private int content;
    private boolean available = false;

    public int get() {
        if (!available) {
            try {
                wait(); // Incorrect usage
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        available = false;
        notifyAll(); // Incorrect usage
        return content;
    }

    public void put(int value) {
        if (available) {
            try {
                wait(); // Incorrect usage
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        content = value;
        available = true;
        notifyAll(); // Incorrect usage
    }
}
```

-   `wait()`, `notifyAll()` 메서드가 동기화 블록(synchronized 블록) 내에서 호출되지 않고 있다는 점에서 잘못된 예
-   항상 반복문 안에서 사용해야 한다
-   이러한 메서드들은 동기화 블록 내에서 호출되어야 하는데, 그렇지 않을 경우 `IllegalMonitorStateException`이 발생

<br/>

### 중간 결론

-   지금은 중요도가 예전만 못한 것 또한 사실이고, wait와 notify를 사용해야 할 이유가 많이 줄었음
-   wait와 notify는 고수준 동시성 유틸리티를 구현할 때만 사용해야 함
-   하지만 올바르게 사용하기가 아주 까다로우니 고수준 동시성 유틸리티를 사용하는 것이 바람직함
-   만약 사용해야한다면 wait는 while문 안에서 호출해야 함
-   일반적으로 notify()보다는 notifyAll()을 사용
-   혹시라도 notify를 사용하다면 응답 불가 상태에 빠지지 않도록 조심해야 함

<br/>

### 동시성 유틸리티

java.util.concurrent 패키지가 제공하는 고수준 동시성 유틸리티는 다음과 같은 범주로 나눌 수 있다.

**1. 실행자 프레임워크 (item 80)**: Executor Framework, ExecutorService, Executors 등을 지원하는 실행자 프레임워크

-   `Executor`: 이 인터페이스는 실행 명령의 개념을 캡슐화한다. 즉, 클래스에서 작업을 처리하는 방법을 특정화하는 대신, 작업이 무엇인지 정의하고 Executor에 제출하여 실행할 수 있다.
-   `ExecutorService`: Executor 인터페이스를 확장한 하위 인터페이스로, 수명 주기를 가진 Executor이다. 즉, 시작과 종료를 관리할 수 있다. 주로 스레드 풀 구현에 사용된다.
    -   Executors.newFixedThreadPool(n)를 호출하여 고정된 크기의 스레드 풀을 생성하면, 한 번에 n 개의 작업만 실행됨

```java
       ExecutorService executorService = Executors.newFixedThreadPool(5);

for (int i = 0; i < 10; i++) {
   final int taskId = i;

   executorService.execute(() -> {
       System.out.println("Task ID : " + taskId + " performed by "
                          + Thread.currentThread().getName());
   });
}

executorService.shutdown();
```

-   `Executors`: Executor, ExecutorService, ScheduledExecutorService, ThreadFactory, Callable 클래스 등에 대한 팩토리 및 유틸리티 메소드를 제공하는 클래스이다.

**2. 동시성 컬렉션(Concurrent Collections)**

**3. 동기화 장치(Synchronizer)**

<br/>

### 동시성 컬렉션

-   동시성 컬렉션은 List, Queue, Map과 같은 표준 컬렉션 인터페이스에 동시성을 가미하여 구현한 컬렉션
-   높은 동시성을 구현하기 위하여 내부적으로 동시성을 제어
-   때문에 동시성 컬렉션을 사용한다면 동시성을 무력화하는 것은 불가능하며 외부에서 락을 추가로 사용하면 오히려 성능이 저하될 수 있음

-   `ConcurrentHashMap`: Hashtable과 유사하지만, 멀티스레드 환경에서 더 나은 성능을 제공하는 Map 인터페이스 구현이다.
    > Hashtable은 모든 메서드에 synchronized 키워드가 붙어 있어서 멀티스레드 환경에서 동기화를 보장하지만, ConcurrentHashMap은 일부 메서드에만 synchronized 키워드가 붙어 있어서 멀티스레드 환경에서 동기화를 보장하지 않는다. 그 대신 ConcurrentHashMap은 동시성을 높이기 위해 내부적으로 동기화를 제공한다.
-   `CopyOnWriteArrayList`: ArrayList와 유사하지만, 쓰기 작업 시 새로운 배열을 생성하여 원본 데이터의 안전성을 보장하는 List 인터페이스 구현이다.
-   `BlockingQueue`: 스레드 안전한 Queue로, 프로듀서-컨슈머 패턴 구현에 유용하다.

<br/>

### 동기화 장치

-   동기화 장치는 스레드가 다른 스레드를 기다릴 수 있게 하여 서로 작업을 조율할 수 있도록 해줌
-   자주 쓰이는 동기화 장치로는 CountDownLatch와 Semapore가 있으며 CyclicBarrier와 Exchanger, Phaser도 있음

-   `Semaphore`: 한 번에 특정 수의 스레드만 액세스할 수 있도록 하는 동기화 메커니즘이다. 동시에 특정 수의 스레드만 작업을 수행하도록 제한할 수 있다. 예를 들어, 데이터베이스 연결과 같이 한정된 리소스에 대한 액세스를 제한할 때 유용하다.
-   `CountDownLatch`: 한 스레드가 다른 하나 또는 그 이상의 스레드가 특정 작업을 완료하기를 기다리도록 하는 동기화 메커니즘이다.
-   `CyclicBarrier`: 여러 스레드가 특정 지점에서 동기화되기를 기다리는데 사용되는 동기화 메커니즘이다.
-   `Exchanger`: 두 스레드가 서로의 데이터를 교환할 수 있도록 하는 동기화 메커니즘이다.

<br/>

### 결론

고수준 동시성 유틸리티를 사용하는 이유는 다음과 같습니다.

**간결성**: 고수준 동시성 유틸리티는 동시성 관련 작업을 추상화하여 복잡성을 줄이고, 가독성을 향상시킵니다. wait(), notify(), synchronized와 같은 저수준의 메소드를 사용하는 것보다 ExecutorService, Future, ConcurrentHashMap 등의 고수준 동시성 유틸리티를 사용하면 코드가 더욱 간결해지고 이해하기 쉬워집니다.

**안전성**: 동시성 프로그래밍은 매우 복잡하며, 실수하기 쉽습니다. wait()와 notify()를 사용하면 race condition, deadlocks, thread leaks 등의 문제를 초래할 수 있습니다. 반면, 고수준 동시성 유틸리티를 사용하면 이러한 문제를 피할 수 있습니다. 예를 들어, ExecutorService를 사용하면 작업의 수명 주기를 쉽게 관리할 수 있고, ConcurrentHashMap을 사용하면 동시에 맵에 액세스하는 여러 스레드 간의 동기화를 걱정할 필요가 없습니다.

**성능**: 고수준 동시성 유틸리티는 효율적인 동시성 제어를 위해 설계되었습니다. Executors와 같은 클래스를 사용하면 적절한 수의 스레드를 사용하여 시스템의 리소스를 최대한 활용할 수 있습니다. 또한, ConcurrentHashMap, CopyOnWriteArrayList와 같은 동시성 컬렉션은 동기화 비용을 최소화하기 위해 세밀한 동시성 제어를 사용합니다.
