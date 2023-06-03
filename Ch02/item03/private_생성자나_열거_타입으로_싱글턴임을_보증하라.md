# item03. private 생성자나 열거 타입으로 싱글턴임을 보증하라

> 클래스를 싱글턴으로 만들면 이를 사용하는 클라이언트를 테스트하기가 어려워질 수 있다.
> 대부분의 상황에서는 원소가 하낭뿐인 열거 타입이 싱글턴을 만드는 가장 좋은 방법이다.

## 1. 싱글턴이란?

-   인스턴스를 오직 하나만 생성할 수 있는 클래스

## 2. 싱글턴을 만드는 방법

### 2.1. public static final 필드 방식의 싱글턴

```java
public class Elvis {
    public static final Elvis INSTANCE = new Elvis();
    private Elvis() { ... }

    public void leaveTheBuilding() { ... }
}
```

-   해당 클래스가 싱글턴임이 API에 명백히 드러남
-   간결함
-   전체 시스템에서 단 하나의 인스턴스임을 보장 (public 이나 protected 생성자가 없음)

<br/>

### 2.2. 정적 팩터리 메서드 방식의 싱글턴

```java

public class Elvis {
    private static final Elvis INSTANCE = new Elvis();
    private Elvis() { ... }
    public static Elvis getInstance() { return INSTANCE; }

    public void leaveTheBuilding() { ... }
}
```

-   정적 팩터리 메서드 방식은 위와 마찬가지로 생성자를 private으로 만들지만
-   인스턴스를 갖는 static final 필드를 private으로 두고 정적 팩터리 메서드에서 이를 반환

***\*원문**: All calls to Elvis.getInstance return the same object reference, and no other Elvis instance will ever be created (with the same caveat mentioned earlier)*

***\*번역** : Elvis.getInstance 호출마다 같은 객체의 참조를 반환하고, 이와 같은 Elvis 인스턴스가 절대로 만들어지지 않는다(이전에 언급한 예외를 제외하고).*

> "with the same caveat mentioned earlier" 부분은 해당 구문이 특정한 예외 사항이 적용되는 경우를 가정한다는 의미

***\*번역본**: (역시 \*리플렉션을 통한 예외는 똑같이 적용된다.)*

\*리플렉션(reflection)은 Java와 같은 몇몇 프로그래밍 언어에서 런타임(runtime)에 클래스와 객체의 메타데이터에 접근하고 조작할 수 있는 기능을 제공하는 개념

-   https://docs.oracle.com/javase/tutorial/reflect/
-   컴파일 타임이 아닌 런타임에 동적으로 타입을 분석하고 정보를 가져오므로 JVM을 최적화할 수 없음
-   뿐만 아니라 직접 접근할 수 없는 private 인스턴스 변수, 메서드에 접근하기 때문에 내부를 노출하면서 추상화가 깨짐
-   이로 인해 예기치 못한 부작용이 발생할 수 있음

<br/>

### 2.3. 열거 타입 방식의 싱글턴

```java
public enum Elvis {
    INSTANCE;
    public void leaveTheBuilding() { ... }
}
```

-   public 필드 방식과 비슷하지만 더 간결함
-   추가 노력 없이 직렬화할 수 있음
    -   심지어 아주 복잡한 직렬화 상황이나 리플렉션 공격에서도 제 2의 인스턴스가 생기는 일을 완벽하게 막아줌
    -   **\*원문**: even in the face of sophisticated serialization or \*reflection attacks.\*
        -   참고: https://aws.amazon.com/ko/blogs/korea/how-to-help-prepare-for-ddos-attacks-by-reducing-your-attack-surface/
-   대부분 상황에서는 원소가 하나뿐인 열거 타입이 싱글턴을 만드는 가장 좋은 방법임
    -   단, 만들려는 싱글턴이 Enum 외의 클래스를 상속해야 한다면 이 방법은 사용할 수 없음
    -   열거 타입이 다른 인터페이스를 구현하도록 선언할 수는 있음

<br/>

---

### 싱글톤 패턴의 문제점

-   싱글톤 패턴을 구현하는 코드 자체가 많이 필요
-   의존관계상 클라이언트가 구체 코드에 의존하므로 \*DIP(의존관계 역전 원칙)를 위반 ( ex : 구체클래스.getInstance())
-   클라이언트가 구체 코드에 의존하므로 \*OCP(개방-폐쇄 원칙)를 위반할 가능성이 높음
-   테스트하기 어려움
-   내부 속성을 변경하거나 초기화하기 어려움
-   private 생성자로 자식 클래스를 만들기 어려움
-   결론적으로 유연성이 떨어짐

<br/>

> \*DIP(Dependency Inversion Principle)는 객체 지향 프로그래밍에서 중요한 설계 원칙 중 하나
> DIP의 핵심 개념은 다음과 같다:
> 고수준 모듈은 저수준 모듈에 의존해서는 안 됩니다. 둘 모두 추상화에 의존해야 합니다.
> 추상화는 구체적인 사항에 의존해서는 안 됩니다. 구체적인 사항은 추상화에 의존해야 합니다.
> 간단히 말해, DIP는 클래스 간의 의존 관계를 형성할 때 추상화와 인터페이스에 의존하도록 권장합니다. 이를 통해 상위 수준의 모듈이 하위 수준의 모듈에 의존하지 않고, 두 모듈 모두 추상화에 의존함으로써 유연하고 확장 가능한 시스템을 구축할 수 있습니다.

<br/>

> \*개방 폐쇄의 원칙(OCP)이란 기존의 코드를 변경하지 않으면서, 기능을 추가할 수 있도록 설계가 되어야 한다는 원칙을 말한다. 보통 OCP를 확장에 대해서는 개방적(open)이고, 수정에 대해서는 폐쇄적(closed)이어야 한다는 의미로 정의

```java
public class Singleton {
    private static Singleton instance;

    private Singleton() {
        // private 생성자
    }

    public static Singleton getInstance() {
        if (instance == null) {
            instance = new Singleton();
        }
        return instance;
    }
}
```

-   Singleton 클래스의 생성자가 private으로 선언
-   외부에서 직접 객체를 생성할 수 없고, getInstance() 메소드를 통해 인스턴스를 얻어야 함
-   객체 수정이 필요하거나 대체하기 위해서는 기존 코드를 수정해야하기 때문에 수정에 폐쇄적이지 않음 `== 설계나 구조에 변경이 필요하다는 의미`
    -   개방적(Open): 설계된 소프트웨어 구조나 모듈은 새로운 기능을 추가하거나 확장할 수 있는 유연성을 갖추어야 한다. 즉, 새로운 요구사항이나 기능이 추가되더라도 기존 코드를 수정하지 않고도 기능을 확장할 수 있어야 한다.
    -   폐쇄적(Closed): 기존의 코드나 모듈은 변경에 닫혀 있어야 한다. 즉, 기존의 코드를 수정하지 않고도 새로운 기능을 추가하거나 변경된 요구사항을 처리할 수 있어야 한다. 이는 기존의 동작을 변경하지 않으면서 새로운 기능을 추가할 수 있는 안정성과 호환성을 제공한다.
