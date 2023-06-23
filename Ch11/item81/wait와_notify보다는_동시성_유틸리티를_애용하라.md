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
-   이러한 메서드들은 동기화 블록 내에서 호출되어야 하는데, 그렇지 않을 경우 `IllegalMonitorStateException`이 발생

<br/>

### 중간 결론

-   지금은 중요도가 예전만 못한 것 또한 사실이고, wait와 notify를 사용해야 할 이유가 많이 줄었음
-   wait와 notify는 고수준 동시성 유틸리티를 구현할 때만 사용해야 함
-   하지만 올바르게 사용하기가 아주 까다로우니 고수준 동시성 유틸리티를 사용하는 것이 바람직함

<br/>

### 동시성 유틸리티

java.util.concurrent 패키지가 제공하는 고수준 동시성 유틸리티는 다음과 같은 범주로 나눌 수 있다.

-   실행자 프레임워크 (item 80): Executor Framework, ExecutorService, Executors 등을 지원하는 실행자 프레임워크
-   동시성 컬렉션: Concurrent Collections
-   동기화 장치: Synchronizer

<br/>

### 동시성 컬렉션

-   동시성 컬렉션은 List, Queue, Map과 같은 표준 컬렉션 인터페이스에 동시성을 가미하여 구현한 컬렉션
-   높은 동시성을 구현하기 위하여 내부적으로 동시성을 제어
-   때문에 동시성 컬렉션을 사용한다면 동시성을 무력화하는 것은 불가능하며 외부에서 락을 추가로 사용하면 오히려 성능이 저하될 수 있음

<br/>

### 동기화 장치

-   동기화 장치는 스레드가 다른 스레드를 기다릴 수 있게 하여 서로 작업을 조율할 수 있도록 해줌
-   자주 쓰이는 동기화 장치로는 CountDownLatch와 Semapore가 있으며 CyclicBarrier와 Exchanger, Phaser도 있음
