# item04. 인스턴스화를 막으려거든 private 생성자를 사용하라
> 요약: abstract로 만들어도 상속 받아서 인스턴스를 만들 수 있다. 따라서 인스턴스화를 막기 위해 private 생성자를 사용하라

- static 메서드와 static 필드만을 담은 클래스를 만들고 싶을 때가 있음 -> **utility class**

## 예: `java.lang.Math`, `java.util.Array` 등
- Arrays를 사용할 때를 `Arrays arrays = new Arrays()`로 생성하지 않음 `Arrays.asList(배열)` 식으로 바로 사용함
- 내부 메소드를 보면 전부 static으로 선언되어 있고 생성자가 private로 선언된 것을 확인할 수 있음
```java
public class Arrays {
    private Arrays() {}

    public static void sort(int[] a) { ... }
    public static void parallelSort(byte[] a)
    // ...
}
```

## abstract class
- `UtilClass.getName()` 로 인스턴스를 만들지 않고 바로 사용함
```java
public class UtilClass {
    public static String getName() {
        return "ABC";
    }

    public static void main(String[] args) {
        UtilClass.getName();  // 인스턴스를 만들지 않고 바로 사용함
    }
}
```

- 해당 클래스를 abstract로 만들어도 인스턴스를 만드는 걸 막을 순 없음 -> 상속 받아서 인스턴스를 만들 수 있기 때문
```java
public abstract class UtilClass {
    public static String getName() {
        return "ABC";
    }

    static class AnotherClass extends UtilClass {
    }

    public static void main(String[] args) {
        AnotherClass anotherClass = new AnotherClass();  // 인스턴스화 가능
    }
}
```

## ✅private 생성자를 추가하여 인스턴스가 생성되는 상황을 방지하자.
- 생성자를 제공하지만 쓸 수 없기 때문에 직관에 어긋나는 점이 있는데, 그 때문에 주석을 추가하는 것이 좋음
- 상속도 막을 수 있음 -> 상속한 경우에 명시적이든 암묵적이든 상위 클래스의 생성자를 호출해야 하는데, 이 클래스의 생성자가 private이라 호출이 막혔기 때문에 상속을 할 수 없음
```java
public class UtilityClass {
    // 기본 생성자가 만들어지는 것을 막는다(인스턴스화 방지용)
    private UtilityClass() { 
        throw new AssertionError(); 
    }
}
```