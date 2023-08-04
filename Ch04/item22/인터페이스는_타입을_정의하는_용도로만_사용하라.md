# 인터페이스는 타입을 정의하는 용도로만 사용하라
### 상수를 정의하는 용도로 인터페이스를 사용하지 말 것!

- 인터페이스에 상수를 구현하면 아래와 같이 상수들을 아무런 네임스페이스없이 참조해서 사용 가능
```java
public interface PhysicalConstants {
    static final double AVOGADROS_NUMBER   = 6.022_140_857e23;

    static final double BOLTZMANN_CONSTANT = 1.380_648_52e-23;

    static final double ELECTRON_MASS      = 9.109_383_56e-31;
}
```

```java
public class MyClass implements PhysicalConstants {

    public static void main(String[] args) {
        System.out.println(BOLTZMANN_CONSTANT);
    }

}
```

- 네임스페이스 없이 인터이스에 상수를 선언하는 것은 안티패턴
- 인터페이스는 타입을 정의하는 목적
- MyClass는 PhysicalConstants인터페이스와 관련이 없음
- 인터페이스의 내부 구현을 참조하기 때문에 캡슐화가 깨짐
- 유틸리티 클래스를 사용할 것

```java
public class PhysicalConstants {
  private PhysicalConstants() { } 

  public static final double AVOGADROS_NUMBER = 6.022_140_857e23;

  public static final double BOLTZMANN_CONST  = 1.380_648_52e-23;

  public static final double ELECTRON_MASS    = 9.109_383_56e-31;
}
```