# item09. try-finally 보다는 try-with-resources를 사용하라
> 요약: 꼭 회수해야 하는 자원을 다룰 때는 try-fianlly 말고, try-with-resources를 사용하자. 예외는 없다.
코드는 더 짧고 분명해지고, 만들어지는 예외 정보도 훨씬 유용하다.

## try-finally
- 전통적으로 자원이 제대로 닫힘을 보장하는 수단으로 try-finally가 쓰였다.
- 더 이상 자원을 회수하는 최선의 방책이 아니다.
```java
static String firstLineOfFile(String path) throws IOException {
    BufferReader br = new BufferReader(new FileReaderPath(path));
    try {
        return br.readLine();
    } finally {
        br.close();
    }
}
```

- 자원이 둘 이상이면 try-fianlly 방식은 너무 지저분 하다.
- read 도 하고 write 도 하고싶은 경우, close 를 2번 해줘야 한다.
```java
static void copy(String src, String dst) throws IOException {
    InputStream in = new FileInoutStream(src);
    try {
        OutputStream out = new FileOutputStream(dst);
        try {
             byte[] buf = new byte[BUFFER_SIZE];
             int n;
             while ((n = in.read(buf)) >= 0)
                out.write(buf, 0, n);
        } finally {
            out.close();
        }
    } finally {
        in.close();
    }
}
```

- 예외는 try 블록과 finally 블록 모두에서 발생할 수 있다.
    - readLine 메서드가 먼저 예외를 던진다 -> 파일을 읽을 수 없습니다.
    - 그 다음 close 메서드가 예외를 던진다 -> 파일을 close 할 수 없습니다.
- 이런 상황이 발생하면 스택 추적 내역에 첫번째 예외에 관한 정보는 남지 않는다 -> **디버깅이 어려워짐**
```java
static String firstLineOfFile(String path) throws IOException {
    BufferReader br = new BufferReader(new FileReaderPath(path));
    try {
        return br.readLine();
    } finally {
        br.close();
    }
}
```

## try-with-resources
- try 안에 파라미터로 자원을 넘겨주는 방식
- try 문에서 선언된 객체들에 대해서 try가 종료될 때 자동으로 자원을 해제해주는 기능이 있다.
- 먼저 이 구조를 사용하려면 해당 자원이 **AutoCloseable** 인터페이스를 구현해야 한다.
    - AutoCloseable 인터페이스 : void 타입의 close 메서드 하나만 덩그러니 정의된 인터페이스
    ```java
    public interface AutoCloseable { void close() throws Exception; }
    ```
    - try-with-resources가 모든 객체의 close를 호출해주지는 않는다.
    - AutoCloseable을 구현한 객체만 close가 호출된다.

```java
public abstract class BufferedReader implements AutoCloseable { ... }
```

```java
static String firstLineOfFile(String path) throws IOException {
    try (BufferedReader br = new BufferedReader(new FileReader(path))) {
				return br.readLine();
    }
}
```

```java
static void copy(String src, String dst) throws IOException {
    try (Input in = new FileInput(src); Output out = new FileOutput(dst)) {
        byte[] buf = new byte[BUFFER_SIZE];
        int n;
        while ((n = in.read(buf)) >= 0) out.write(buf, 0, n);
    }
}
```

- 코드의 가독성이 훨씬 좋아지고 문제 진단에도 훨씬 유리하다.
- readLine 과 close 에서 모두 예외가 발생하면 먼저 발생한 readLine의 예외가 기록되고, close에서 발생한 예외는 숨겨진다.
- 이렇게 숨겨진 예외들은 스택 추적 내역에 **숨겨졌다(suppressed)는 꼬리표를 달고 출력된다.**
    - `Throwable`의 `getSuppressed` 메서드를 이용하면 프로그램 코드에서 가져올 수도 있다.
    ```java
    void addSuppressed(Throwwable exception)
    public final Throwable[] getSuppressed()
    ```

## try-with-resources + catch
- catch절 덕분에 try문을 더 중첩하지 않더라도 다수의 예외를 처리할 수 있다.
- 파일을 여는 것에 실패하거나, 데이터를 읽지 못했을 때 예외 대신 기본값을 반환하도록 수정된 코드
```java
static String firstLineOfFile (String path, String defaultVal){
    try (BufferedReader br = new BufferedReader(new FileReader(path))) {
        return br.readLine();
    } catch (IOException e) { return defaultVal; }
}
```

- 예외처리란, 프로그래머가 예기치못한 예외의 발생에 미리 대처하는 코드를 작성하는 것
    - (나쁜)사용자가 발생시키는 예외에 대해, 개발자가 미리 대처를 해줄 수 있다.
    - 실행중인 **쓰레드**의 **비정상적인 종료**를 막고 상태를 정상상태로 유지하는 것이 목적
- 예외가 처리되지 못한경우, **쓰레드**은 비정상적으로 종료되며 처리되지 못한 예외의 원인을 JVM의 예외처리기(UncaughtExceptionHandler)가 화면에 출력해준다