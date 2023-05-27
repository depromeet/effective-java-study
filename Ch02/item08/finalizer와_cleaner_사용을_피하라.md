# Item 8. finalizer와 cleaner 사용을 피하라

> cleaner (Java 8까지는 finalizer)는 안전망 역할이나 중요하지 않은 네이티브 자원 회수용으로만 사용하자. 물론 이런 경우라도 불확실성과 성능 저하에 주의해야 한다.

<br/>

### 자바의 두 가지 객체 소멸자

**finalizer**

-   예측할 수 없고 상황에 따라 위험할 수 있어서 일반적으로 불필요함
-   오작동, 낮은 성능, 이식성 문제의 원인이 되기도 함
-   Java 9부터 deprecated API로 지정되어 cleaner를 대안으로 사용

**cleaner**

-   finalized 보다는 덜 위험하지만 여전히 예측할 수 없고, 느리고, 일반적으로 불필요함

<br/>

### appendix. 코틀린에서의 객체 소멸

-   코틀린에서는 가비지 컬렉터(Garbage Collector)가 더 이상 사용되지 않는 객체들을 자동으로 감지하고, 이를 메모리에서 해제
-   이를 통해 개발자는 명시적으로 객체를 해제할 필요 없이 메모리 관리에 집중할 필요가 없음
-   따라서 코틀린에서는 finalizer나 cleaner와 같은 객체 소멸자와 관련된 개념을 직접 사용할 필요가 없습니다.
-   대신 메모리 누수를 방지하기 위해 관리되지 않는 리소스를 사용하는 경우에는 close() 메서드나 use 함수와 같은 구조를 활용하여 명시적으로 리소스를 해제
-   이렇게 명시적으로 리소스를 해제하면 가비지 컬렉터가 자동으로 메모리를 해제하는 것보다 더 적절하게 리소스를 관리할 수 있음

<br/>

### `finalizer`

-   `finalizer`는 자바 객체의 소멸자입니다.
-   객체가 가비지 컬렉션(Garbage Collection)에 의해 수거되기 전에 실행됩니다.
-   `finalize()` 메서드를 오버라이딩하여 `finalizer`를 구현할 수 있습니다.
-   `finalize()` 메서드는 객체가 소멸될 때 자동으로 호출되며, 수동으로 호출할 수는 없습니다.
-   `finalize()` 메서드는 객체가 소멸되기 전에 실행되기 때문에 일반적으로 예측할 수 없습니다.
-   `finalize()` 메서드의 실행 시점과 순서는 JVM에 의해 결정되므로 제어할 수 없습니다.
-   `finalize()` 메서드의 오작동이나 잘못된 사용은 예기치 않은 결과를 초래할 수 있습니다.
-   `finalize()` 메서드는 성능 문제를 야기할 수 있으며, 객체 소멸에 필요한 추가적인 비용이 들어갑니다.
-   Java 9부터 `finalize()` 메서드는 deprecated API로 지정되어 `cleaner`를 대안으로 권장합니다.

<br/>

### `cleaner`

-   `cleaner`는 자바 9부터 도입된 객체 소멸자의 대안입니다.
-   `cleaner`는 `PhantomReference`와 연결되어 사용됩니다.
-   `PhantomReference`는 가비지 컬렉션의 수집 대상이지만, 수집될 때 특별한 액션이 필요한 객체를 추적하는 데 사용됩니다.
-   `cleaner` 객체는 `PhantomReference`와 연결된 객체의 소멸 시 동작할 코드를 제공합니다.
-   `cleaner` 객체는 `Cleanable` 인터페이스를 통해 생성되며, `clean()` 메서드를 오버라이딩하여 동작을 구현합니다.
-   `clean()` 메서드는 객체 소멸 시 수동으로 호출하거나, `PhantomReference`가 수집될 때 자동으로 실행됩니다.
-   `cleaner`는 `finalize()` 메서드보다 예측 가능하고, 더 적은 위험성을 가지며, 일반적으로 불필요합니다.
-   하지만 `cleaner` 역시 실행 시점과 순서가 JVM에 의해 결정되므로 완전한 제어는 어렵습니다.
-   `cleaner`는 일반적으로 리소스 관리에 사용되며, `try-finally` 블록과 유사한 역할을 수행할 수 있습니다.

<br/>

### Finalizer 공격

> The method finalize() from the type Object is deprecated since version 9Java(67110270)

Finalizer는 일반적으로 실행 시점과 순서가 예측할 수 없기 때문에, 공격자가 악성 코드를 finalizer에 포함시켜 시스템을 공격하는 방식으로 공격이 가능함

-   [공격 코드](https://github.com/gunh0/java-atoz/blob/5c9f4def6fa7063dcd4e0b5a44c4a85acda39412/effective/08_FinalizerAttack.java)와 [실행 결과](https://github.com/gunh0/java-atoz/blob/5c9f4def6fa7063dcd4e0b5a44c4a85acda39412/effective/runner.ipynb)

<br/>

### finalizer와 cleaner를 대신할 AutoCloseable

Autocloseable을 구현하고, 클라이언트에서 인스턴스를 다 쓰고 나면 close 메서드를 호출하자. (일반적으로 예외가 발생해도 제대로 종료되도록 try-with-resources를 사용해야 한다.)

```java
import java.lang.ref.Cleaner;

public class Restaurant implements AutoCloseable {
    private static final Cleaner cleaner = Cleaner.create();

    private static class Table implements Runnable {
        int cups; // 치워야 하는 컵
        int plates; // 치워야 하는 접시

        Table (int cups, int plates) {
            this.cups = cups;
            this.plates = plates;
        }

        @Override public void run() {
            System.out.println("자리 정리중...");
            this.cups = 0;
            this.plates = 0;
        }
    }

    private final Table table;

    private final Cleaner.Cleanable cleanable;

    public Restaurant (int cups, int plates) {
        table = new Table(cups, plates);
        cleanable = cleaner.register(this, table);
    }
    @Override
    public void close() throws Exception {
        cleanable.clean();
    }
}
```

이 때, 각 인스턴스는 자신이 닫혔는지 추적할 수 있도록 close 메서드에서 이 객체가 더 이상 유효하지 않음을 필드에 기록하고, 다른 메서드는 이 필드를 검사해서 객체가 닫힌 후에 불렸다면 IllegalStateException을 던져야 한다.

<br/>

### finalizer와 cleaner의 적절한 쓰임새

**1. 자원의 소유자가 close 메서드를 호출하지 않는 것에 대비한 안전망 역할**

자바 라이브러리의 FileInputStream, FileOutputStream, ThreadPoolExecutor가 대표적으로 이런 예시에서 사용 중

**2. 네이티브 피어(native peer)와 연결된 객체에서 사용**

네이티브 피어: 일반 자바 객체가 네이티브 메서드를 통해 기능을 위임한 네이티브 객체.

자바 객체가 아니기 때문에 가비지 컬렉터에서는 이 존재를 알지 못함. 따라서 cleaner나 finalizer가 나서서 처리하기에 적당하다.
단, 성능 저하를 감당할 수 없거나 네이티브 피어가 사용하는 자원을 즉시 회수해야 한다면 앞서 설명한 close 메서드를 사용해야 한다.
