# Item 10. equals는 일반 규약을 지켜 재정의하라

> equals 메서드는 재정의하기 쉬워 보이지만, 함정이 도사리고 있다. 잘못 재정의한 equals 메서드는 프로그램을 오동작하게 만든다. equals 메서드를 재정의해야 하는 상황과 그 규약을 알아보자.

<br/>

### equals를 재정의하지 않아도 되는 경우

**1. 각 인스턴스가 본질적으로 고유하다.**

-   값을 표현하는 게 아니라 동작하는 개체를 표현하는 클래스 (**Thread**)

```java
@Override
public boolean equals(Object obj) {
    if (obj == this)
        return true;

    if (obj instanceof WeakClassKey) {
        Object referent = get();
        return (referent != null) && (referent == ((WeakClassKey) obj).get());
    } else {
        return false;
    }
}
```

**2. 인스턴스의 '논리적 동치성'을 검사할 일이 없다.**

-   equals를 재정의하여 각 인스턴스가 같은 정규표현식을 나타내는지 검사할 수 있지만,
    그럴 일이 없다면 굳이 재정의할 필요가 없다. (**Pattern**)

```java
void patternTest() {
    final Pattern P1 = Pattern.compile("//.*");
    final Pattern P2 = Pattern.compile("//.*");

    System.out.println(P1.equals(P1)); // true
    System.out.println(P1.equals(P2)); // false
    System.out.println(P1.pattern()); // //.*
    System.out.println(P1.pattern().equals(P1.pattern())); // true
    System.out.println(P1.pattern().equals(P2.pattern())); // true
}
```

**3. 상위 클래스에서 재정의한 equals가 하위 클래스에도 딱 들어맞는다.**

-   Set의 구현체들은 모두 AbstractSet이 구현한 equals를 상속 받아서 쓴다.
    그 이유는 서로 같은 Set인지 구분하기 위해서 사이즈, 내부 값들을 비교하면 되기 때문이다.

```java
public boolean equals(Object o) {
    if (o == this) {
        return true;
    }

    if (!(o instanceof Set)) {
        return false;
    }

    Collection<?> c = (Collection<?>) o;
    if (c.size() != size()) { // 사이즈 비교
        return false;
    }
    try {
        return containsAll(c); // 내부 인스턴스 비교
    } catch (ClassCastException | NullPointerException unused) {
        return false;
    }
}
```

그런데 그로 인해서 이런 일도 생긴다.

```java
void setTest() {
    Set<String> hash = new HashSet<>();
    Set<String> tree = new TreeSet<>();
    hash.add("Set");
    tree.add("Set");

    System.out.println(hash.equals(tree)) // true;
}
```

**4. 클래스가 private이거나 package-private이고 equals 메서드를 호출할 일이 없다.**

```java
@Override
public boolean equals(Object o){
    throw new AssertionError(); // 호출 금지
}
```

<br/>

### 언제 equals를 재정의해야 하는가?

> 세상에 홀로 존재하는 클래스는 없기에, 협력 관계에서 수 많은 클래스는 자신에게 전달된 객체가 equals 규약을 지킨다고 가정하고 동작한다.

**1. 반사성 (reflexivity)**

-   객체는 자기 자신과 같아야 한다.

**2. 대칭성 (symmetry)**

-   x.equals(y)가 참이면 그 반대(y.equals(x))도 참이어야 한다.

다음과 같이 대소문자를 구분하지 않는 CaseInsensitiveString 클래스가 있다.

```java
public class CaseInsensitiveString {
    private final String s;

    public CaseInsensitiveString(String s) {
        this.s = s;
    }

    @Override
    public boolean equals(Object o) {
        // ...생략
        if (o instanceof String) {  //String과의 비교 연산 시도
            return s.equalsIgnoreCase((String) o);
        }
        // ...생략
        return false;
    }
}
```

이 클래스의 equals에서는 일반 문자열(String)과 비교하려 하고 있다.

```java
CaseInsensitiveString cis = new CaseInsensitiveString("AAA");
String s = "aaa";

cis.equals(s); //true
s.euqals(cis); //false
```

CaseInsensitiveString의 equals 자체는 잘 작동하지만 **그 반대는 작동하지 않는다.**

String의 equals는 CaseInsensitiveString의 존재를 알지 못하기 때문이다.

이를 해결하기 위해서는 equals에서 일반 String과 비교하는 부분을 없애야 한다.

**3. 추이성(transitivity)**

-   첫 번째 객체와 두 번째 객체가 같고, 두 번째 객체와 세 번째 객체가 같다면, 첫 번째 객체와 세 번째 객체도 같아야 한다.

추이성 문제는 주로 상위 클래스에 없는 새로운 필드를 하위 클래스에 추가할 때 발생한다.

다음과 같이 스마트폰을 표현하는 GalaxyS 클래스와 그 하위의 GalaxyNote 클래스가 있다.

```java
public final String name;

    public GalaxyS(String name) {
        this.name = name;
    }
    @Override
    public boolean equals(Object o) {
        if (o instanceof GalaxyS) {
            return name.equals(((GalaxyS) o).name);
        }
        return false;
    }
```

GalaxyNote 클래스에서는 S펜 기능을 위한 pen 필드가 추가되었다.

```java
public class GalaxyNote extends GalaxyS {
    public final String pen;

    public GalaxyNote(String name, String pen) {
        super(name);
        this.pen = pen;
    }

    @Override
    public boolean equals(Object o) {
        if(!(o instanceof GalaxyS)) return false;
        if (o instanceof GalaxyNote) { //GalaxyNote 일 경우 이름과 펜 모두 비교
            return super.equals(o) && pen.equals(((GalaxyNote) o).pen);
        }
        return name.equals(((GalaxyS) o).name); //GalaxyS 일 경우 이름만 비교
    }
}
```

대칭성을 지키기 위해 GalaxyS일 경우와, GalaxNote일 경우를 나누어 equals를 정의하였다.

하지만 이는 또다른 문제가 발생한다. 추이성을 위반하는 것이다.

```java
@Test
    void equalsTest(){
        GalaxyNote g1 = new GalaxyNote("s20", "blue pen");
        GalaxyS g2 = new GalaxyS("s20");
        GalaxyNote g3 = new GalaxyNote("s20", "red pen");

        assertThat(g1).isEqualTo(g2);   //pass
        assertThat(g2).isEqualTo(g3);   //pass
        assertThat(g1).isEqualTo(g3);   //fail
    }
```

이처럼 g1, g2는 같고 g2, g3도 같지만 g1, g3는 서로 같지 않다. (추이성 위반)

이를 해결하기 위해 equals를 instanceof 검사 대신 getClass 검사로 바꾸면 추이성을 지킬 수는 있다. 하지만...

```java
if (o == null || getClass() != o.getClass()) return false;
```

이는 \*리스코프 치환 원칙의 위배한다는 문제가 있다.

> 상위 타입의 자료형은 하위 타입으로 변환되어도 문제 없이 작동해야 한다.

즉 상위 클래스인 GalaxyS는 하위클래스인 GalaxyNote와 getClass가 다르다.

아쉽게도, 클래스를 확장해 새로운 값을 추가하면서 equals 규약을 만족시킬 방법은 존재하지 않는다.

다만 \*컴포지션 패턴을 통해 이를 우회할 수 있다.

> 컴포지트 패턴(Composite pattern)이란,
> 객체들의 관계를 트리 구조로 구성하여 부분-전체 계층을 표현하는 패턴으로,
> 사용자가 단일 객체와 복합 객체 모두 동일하게 다루도록 하는 패턴

```
       +---------------------+
       |       ClassA        |
       +---------------------+
       |    - fieldA1        |
       |    - fieldA2        |
       |    - fieldA3        |
       |                     |
       |    +-------------+  |
       |    |  ClassB     |  |
       |    +-------------+  |
       |    |  - fieldB1  |  |
       |    |  - fieldB2  |  |
       |    +-------------+  |
       |    |             |  |
       |    +-------------+  |
       |    |  ClassC     |  |
       |    +-------------+  |
       |    |  - fieldC1  |  |
       |    |  - fieldC2  |  |
       |    +-------------+  |
       +---------------------+
```

ClassA는 ClassB와 ClassC의 객체를 생성하여 자신의 필드로 포함합니다.

이렇게 함으로써 ClassA는 ClassB와 ClassC의 기능을 사용할 수 있고, 더 복잡한 구조의 클래스를 구성할 수 있습니다.

```java
public class GalaxyNoteComposition {
    private final GalaxyS galaxyS;
    private final String pen;

    public GalaxyNoteComposition(String name, String pen) {
        this.galaxyS = new GalaxyS(name);
        this.pen = pen;
    }

//    GalaxyS의 뷰를 반환
    public GalaxyS asGalaxyS(){
        return galaxyS;
    }

    @Override
    public boolean equals(Object o) {
        if(!(o instanceof GalaxyNoteComposition)) return false;
        GalaxyNoteComposition gn = (GalaxyNoteComposition) o;
        //컴포지션 내부의 멤버들을 비교
        return galaxyS.equals(gn.galaxyS) && pen.equals(gn.pen);
    }
}
```

컴포지션 패턴을 통해 GalaxyS를 상속 받는 대신 private 멤버로 두고 필요할 경우 GalaxyS의 뷰를 반환하는 메소드를 만든다.

이후 equals를 재정의할 때는 상위/하위 클래스를 따질 필요 없이 **컴포지션 클래스 내부의 멤버들을 비교**하기만 하면 된다.

**4. 일관성(consistency)**

두 객체가 같다면 (수정되지 않는한) 앞으로 영원히 같아야 한다.

equals를 판단하는 기준이 신뢰할 수 없는 자원이 되어서는 안된다.

**5. null-아님(non-null)**

모든 객체가 null이 아니어야 한다. (x.equals(null)은 항상 false)

<br/>

### equals 좋은 재정의 방법

```java
@Override
public boolean equals(final Object o) {
    // 1. == 연산자를 사용해 입력이 자기 자신의 참조인지 확인한다.
    if (this == o) {
        return true;
    }

    // 2. instanceof 연산자로 입력이 올바른 타입인지 확인한다.
    if (!(o instanceof Point)) {
        return false;
    }

    // 3. 입력을 올바른 타입으로 형변환 한다.
    final Piece piece = (Piece) o;

    // 4. 입력 개체와 자기 자신의 대응되는 핵심 필드들이 모두 일치하는지 하나씩 검사한다.

    // float와 double을 제외한 기본 타입 필드는 ==를 사용한다.
    return this.x == p.x && this.y == p.y;

    // 필드가 참조 차입이라면 equals를 사용한다.
    return str1.equals(str2);

    // null 값을 정상 값으로 취급한다면 Objects.equals로 NullPointException을 예방하자.
    return Objects.equals(Object, Object);
}
```

<br/>

### Conclusion

-   꼭 필요한 경우가 아니라면 재정의하지 말자.
-   그래도 필요하다면 핵심필드를 빠짐 없이 비교하며 다섯 가지 규약을 지키자.
-   어떤 필드를 먼저 비교하느냐가 equals의 성능을 좌우하기도 한다. 최상의 성능을 바란다면 다를 가능성이 더 크거나 비교하는 비용이 더 싼 필드를 먼저 비교하자.
-   객체의 논리적 상태와 관련 없는 필드는 비교하면 안 된다.
-   핵심 필드로부터 파생되는 필드가 있다면 굳이 둘다 비교할 필요는 없다. 편한 쪽을 선택하자.
