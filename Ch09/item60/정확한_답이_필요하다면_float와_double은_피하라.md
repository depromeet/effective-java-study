# Item 60. 정확한 답이 필요하다면 float와 double은 피하라

### float과 double의 문제

-   float과 double의 연산은 정확하지 않다.

```java
    @Test
    void 부동소수점_연산은_정확하지_않음 (){
        double result = 1.03 - 0.42;
        System.out.println(result); // 0.6100000000000001
        assertThat(result).isNotEqualTo(0.61);
    }
```

-   1.03 - 0.42와 같은 단순한 계산에서도 오차가 발생한다.
-   그 이유는 float과 double은 부동소수점 방식을 사용하고 있기 때문이다.

<br/>

### 고정소수점과 부동소수점

-   컴퓨터는 숫자를 2진수로 표현한다.
-   이 때, 정수를 2진수로 표현하는 것은 문제가 없다.
-   하지만 소수점을 포함한 실수를 표현할 때는 상황이 조금 달라진다.
-   소수점의 위치를 표현하고, 무엇이 정수 부분이고 무엇이 실수 부분인지 구분해야 하기 때문이다.
-   컴퓨터는 이를 위해 고정소수점 방식과 부동소수점 방식을 사용한다.

<br/>

### 올바른 방안

-   위 같은 문제를 올바르게 해결하기 위해서는 정수를 이용하여 실수를 표현한 `BigDecimal`이나 `int`, `long`을 사용해야 한다.

**BigDecimal**

Java에는 정수를 이용하여 실수를 표현하기 위한 `BigDecimal`이 제공된다.

-   아래는 BigDecimal의 실제 내부 구현 중 일부분 이다.

```java
public class BigDecimal extends Number implements Comparable<BigDecimal> {
...
private final BigInteger intVal;
private final int scale;
private transient int precision;
...
}
```

-   private final BigInteger intVal: 정수를 저장하는데 사용한다.
-   private final int scale: 총 소수점 자리수를 가리킨다.
-   private transient int precision: 정밀도이다. 수가 시작하는 위치부터 끝나는 위치까지 총 자리수이다.
-   위 같은 내부 변수를 활용하여 실수를 정확하게 표현한다.

```java
    @Test
    void BigDecimal_사용(){
        BigDecimal result = new BigDecimal("1.03").subtract(new BigDecimal("0.42"));
        System.out.println(result); // 0.61
        assertThat(result).isEqualTo(new BigDecimal("0.61"));
    }
```

-   위 테스트는 정확하게 일치하여 성공한다.

<br/>

**int 혹은 long**

BigDecimal의 대안으로 int 혹은 long 타입을 사용할 수 있다. 하지만 다룰 수 있는 값의 크기가 제한되며 소수점이 필요한 경우 따로 관리해야 한다.

**int**

-   4 byte
-   저장 가능 범위: –2,147,483,648 ~ 2,147,483,647
-   만약 다뤄야 하는 숫자가 9자리 이하이면 int의 사용을 고려할 수 있다.

**long**

-   8 byte
-   저장 가능 범위: -9,223,372,036,854,775,808 ~ 9,223,372,036,854,775,807
-   만약 다뤄야 하는 숫자가 18자리 이하이면 long의 사용을 고려할 수 있다.

<br/>

### 정리

-   정확한 답이 필요한 경우 float나 double을 피해야 한다.
-   소수점 관리 및 성능 저하를 신경 쓰지 않는 다면 BigDecimal의 사용은 좋은 대안이 된다.
-   하지만 성능이 중요하고 숫자가 너무 크지 않으며 소수점을 직접 관리할 자신이 있다면 int나 long 사용을 고려할 수 있다.
