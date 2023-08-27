# item19.상속을 고려해 설계하고 문서화 하라, 그러지 않았다면 상속을 금지하라

### 상속을 고려한 설계와 문서화
상속용 클래스는 
- 재정의 할 수 있는 메서드들을 내부적으로 어떻게 이용하는지(자기사용) 문서로 남겨야 한다.

- 재정의 가능 = public 과 protected 메서드 중 final 이 아닌 모든 메서드
### Implementation Requirements
API 문서의 메서드 설명 끝에서 Implementation Requirements 로 시작하는 절을 종종 볼 수 있다.

- 메서드의 내부 동작 방식을 설명하는 곳
- @implSpec 태그 → javadoc 도구가 생성해줌
- 클래스를 안전하게 상속할 수 있도록 내부 구현 방식을 설명해야 한다.

### protected 로 노출할 메서드의 결정
- 수는 가능한 적어야 한다.
- 3개 정도 실제 하위 클래스를 만들어 보는 것이 유일하다.
- 제 3 자의 작성으로도 검증해야 한다.
- 꼭 필요한 protected 멤버를 놓쳤다면 하위 클래스를 작성할 때 그 빈자리가 확연히 드러난다.
- 하위 클래스를 여러 개 만들 때까지 전혀 쓰이지 않는 protected 멤버는 private 이어야 할 가능성이 크다.
- 너무 적게 노출해서 상속으로 얻는 이점마저 없애지 않도록 주의 한다

### 상속용 클래스의 생성자는 직/간접적으로 재정의 가능메서드를 호출하면 안된다.
private, final, static 메서드는 재정의가 불가능하니 생성자에서 안심하고 호출해도 된다.

- 상위 클래스의 생성자가 하위 클래스의 생성자보다 먼저 실행된다.
- 하위 클래스에서 재정의한 메서드가, 하위 클래스의 생성자에서 초기화 하는 값에 의존 → 오동작
```java

public class Super {
// 잘못된 예 - 생성자가 재정의 가능 메서드를 호출함.
public Super() {
overrideMe();
}
public void overrideMe() { }
}

public final class Sub extends Super {
// 초기화 되지 않은 final 필드, 생성자에서 초기화 한다.
private final Instant instant;

    Sub() {
        instant = Instant.now();
    }
    // 재정의 가능 메서드, 상위 클래스의 생성자가 호출한다.    
    @Override public void overrideMe() {
        System.out.println(instant);    // null 출력   
    }

    public static void main(String[] args) {
        Sub sub = new Sub();
        sub.overrideMe();   
    }
}
```
System.out.println(instant); 이 실행 될 때, NullPointerException 이 발생 하지만 println 이 null 입력도 받아 들이기 때문에 에러를 던지지 않았다.

### 상속을 금지하는 방법
1. final 클래스 선언
2. 모든 생성자를 private 이나 package-private 으로 선언, public 정적 팩터리
3. 래퍼 클래스 패턴

Set, List, Map 같이 핵심기능을 정의한 인터페이스가 있고, 클래스가 그 인터페이스를 구현했다면 상속을 금지해도 개발하는 데 아무런 어려움이 없을 것이다.