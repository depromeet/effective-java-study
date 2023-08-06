# ordinal 메서드 대신 인스턴스 필드를 사용하라
대부분의 열거 타입 상수는 자연스럽게 하나의 정숫값에 대응되며 모든 열거 타입은 해당 상수가 그 열거 타입에서 몇 번째 위치인지를 반환하는 ordinal이라는 메서드를 제공한다.

#### 잘못 사용한 경우
```java
public enum Ensemble {
    SOLO, DUET, TRIO, QUARTET, QUINTET, SEXTET, SEPTET, OCTET, NONET, DECTET;

    public int numberOfMusicians() {
        return ordinal() + 1;
    }
}

```

- 상수 선언 순서를 바꾸는 순간 numberOfMusicians 메서드의 기능을 상실하게 된다.  -> 중간에 추가 되는 경우
- 값을 중간에 비우기도 어려운 문제가 발생
#### 인스턴스 필드를 사용하라

- 인스턴스 필드를 두어 값을 초기화 하여 사용

```java
public enum Ensemble {
    SOLO(1),
    DUET(2),
    TRIO(3),
    QUARTET(4),
    QUINTET(5),
    SEXTET(6),
    SEPTET(7),
    OCTET(8),
    DOUBLE_QUARTET(8),
    NONET(9),
    DECTET(10),
    TRIPLE_QUARTET(12);

    private final int numberOfMusicians;

    Ensemble(int numberOfMusicians) {
        this.numberOfMusicians = numberOfMusicians;
    }

    public int numberOfMusicians() {
        return numberOfMusicians;
    }
}
```

#### ordinal은 언제 사용하나?
- ordinal은 EnumSet이나 EnumMap 같은 열거 타입 기반의 범용 자료구조에 사용 될 목적으로 만들어 짐
