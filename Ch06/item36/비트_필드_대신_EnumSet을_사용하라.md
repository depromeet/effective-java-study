### ☁️ 비트 필드 열거 상수

**열거한 값들이 단독이 아닌 집합으로 사용**되어야 할 경우, `EnumSet`이 나오기 전에는 각 상수에 서로 다른 2의 거듭제곱 값을 할당한 정수 열거 패턴을 사용했다.

```java
public class Text{
	public static final int STYLE_BOLD = 1 << 0;  //1
    public static final int STYLE_ITALIC = 1 << 1;  //2
    public static final int STYLE_UNDERLINE = 1 << 2;  //4
    public static final int STYLE_STRIKETHROUGH = 1 << 3;  //8
    
    public void applyStyles(int styles) {
        System.out.printf("Applying styles %s to text%n", styles);
   }
}
```

여러 상수를 하나의 집합으로 모을려면, 아래와 같이 비트 연산자 `OR`을 사용하면 된다. 이렇게 되면 비트 필드로 표현되는 하나의 집합을 생성할 수 있다.

```java
text.applyStyles(STYLE_BOLD | STLYE_ITALIC); // 0001 OR 0010 = 0011
```

> 🫧 **비트마스킹**<br>
비트 필드를 사용하여 비트별 연산을 통해 **집합 연산**을 수행하는 것을 의미한다.

#### 비트 필드 단점 

1. 예전에 보았던 정수 열거 상수의 단점을 그대로 지닌다.(타입 안전성 X, 이름 공간 충돌 등) 
2. 비트 값 필드가 그대로 출력되었을 때 해석하기 어렵다.
3. 최대 몇 비트가 필요한지 예측해서 적절한 타입(int/long)을 선택해야 한다. 

```java
public static void main(String[] args) {
     Text text = new Text();
     text.applyStyles(Text.STYLE_BOLD | Text.STYLE_ITALIC);
}
```
![](https://velog.velcdn.com/images/semi-cloud/post/2a4d6200-5abf-42d5-bef4-a1c77dbea2cf/image.png)


### ☁️ EnumSet
`java.util` 패키지의 **EnumSet 클래스**는 **열거 타입 상수의 값으로 구성된 집합을 효과적으로 표현**해준다.

![](https://velog.velcdn.com/images/semi-cloud/post/78de67ea-9c10-48c5-941b-e339dd1ea8f3/image.png)


`Set` 인터페이스 자체를 구현하고 있기 때문에 타입 안전하며, 출력했을 때도 알아보기 쉽게 정보를 담고 있다는 장점이 있다.

> 🫧 **참고**<br>
`EnumSet` 은 비트 벡터로 구현되었기 때문에, 원소가 `64` 개 이하라면 `EnumSet` 전체를 `long` 변수 하나로 표현하여 비트 필드에 비견되는 성능을 보여준다.
```java
class RegularEnumSet<E extends Enum<E>> extends EnumSet<E> {
    private long elements = 0L;   // 비트 필드
 }
 ```
 ```java
 class JumboEnumSet<E extends Enum<E>> extends EnumSet<E> {
     private long elements[];      
}
```

앞에서의 코드를 `EnumSet` 으로 수정해보자.

```java
public class Text {
    public enum Style {BOLD, ITALIC, UNDERLINE, STRIKETHROUGH}

    public void applyStyles(Set<Style> styles) {
        System.out.printf("Applying styles %s to text%n",
                Objects.requireNonNull(styles));
    }
}

public static void main(String[] args) {
     Text text = new Text();
     text.applyStyles(EnumSet.of(Style.BOLD, Style.ITALIC));
}
```

![](https://velog.velcdn.com/images/semi-cloud/post/e1292aa2-e07b-4bff-93f8-28ad637e5ff5/image.png)

#### EnumSet 단점

`EnumSet` 은 불변이 불가능하다. 
하지만 `Collections.unmodifiableSet` 으로 `EnumSet`을 감싸, 집합을 수정하려 하거든 예외를 터트리게 하는 방식으로 사용할 수 있는 방안은 존재한다.

> 📚 **핵심 정리**<br>
열거할 수 있는 타입을 한데 모아 집합 형태로 사용한다고 해도 비트 필드를 사용할 이유는 없다. `EnumSet` 클래스가 비트 필드 수준의 명료함과 성능을 제공하고, 열거 타입의 장점까지 주기 때문이다. 
