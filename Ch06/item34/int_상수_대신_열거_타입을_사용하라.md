# item34. int 상수 대신 열거 타입을 사용하라
> 열거 타입은 더 읽기 쉽고 안전하고 강력하다. 하나의 메서드가 상수별로 다르게 동작할 때가 있는데, 이때는 switch문 보다는 **상수별 메서드 구현**을 사용하자. 열거 타입 상수 일부가 같은 동작을 공유한다면 **전략 열거 타입 패턴**을 사용하자.

## int enum pattern(정수 열거 패턴)
```java
public static final int APPLE_FUJI = 0;
public static final int APPLE_PIPPIN = 1;
public static final int APPLE_GRANNY_SMITH = 2;

public static final int ORANGE_NAVEL = 0;
public static final int ORANGE_TEMPLE = 1;
public static final int ORANGE_BLOOD = 2;
```
- 타입 안전을 보장할 수 없다
- 표현력이 좋지 않다
    - 예제에서는 사과용 상수는 APPLE_로, 오렌지용 상수는 ORANGE_로 시작한다
- 정수 열거 패턴을 위한 별도 이름공간이 없다
- 프로그램이 깨지기 쉽다
    - 평범한 상수를 나열한 것뿐이라 컴파일하면 그 값이 클라이언트 파일에 그대로 새겨진다
    - 상수의 값이 바뀌면 클라이언트도 반드시 다시 컴파일해야 한다
- 문자열로 출력하기 까다롭다
    - 값을 출력하거나 디버거로 살펴보면 단지 숫자로만 보여서 썩 도움이 되지 않는다
    - 같은 정수 열거 그룹에 속한 모든 상수를 순회하는 방법도 마땅하지 않다
    - 또 이 안에 상수가 몇 개인지도 알 수 없다

### string enum pattern(문자열 상수를 사용하는 변형 패턴)
```java
public static final int APPLE_FUJI = "apple fuji";
public static final int APPLE_PIPPIN = "apple pippin";
public static final int APPLE_GRANNY_SMITH = "apple granny smith";

public static final int ORANGE_NAVEL = "orange navel";
public static final int ORANGE_TEMPLE = "orange temple";
public static final int ORANGE_BLOOD = "orange blood";
```
- 더 나쁘다
- 상수의 의미를 출력할 수 있다는 점은 좋지만, 문자열 상수 이름 대신 문자열 값을 그대로 하드코딩하게 만들기 떄문이다
- 문자열에 오타가 있어도 컴파일러는 확인할 길이 없으니 자연스럽게 런타임 버그가 생긴다
- 문자열 비교는 비교적 성능 저하를 일으킨다

---

## ✅ enum type(열거 타입)
```java
public enum Apple { FUJI, PIPPIN, GRANNY_SMITH }
public enum Orange { NAVEL, TEMPLE, BLOOD }
```

### 장점
- 열거 타입은 인스턴스 통제된다
    - 밖에서 접근할 수 있는 생성자를 제공하지 않으므로, 사실상 final이기 때문이다
- 컴파일타임 타입 안정성을 제공한다
    - 위 코드에서 Apple의 열거 타입을 매개변수로 받는 메서드를 선언했다면, 건네받은 참조는 Apple의 세 가지 값 중 하나임이 확실하다
    - 다른 타입을 넘기려 하면 컴파일 오류가 난다
- 각자의 이름공간이 있다
    - 이름이 같은 상수도 공존 가능하다
- 클라이언트에는 아무 영향이 없다
    - 열거 타입에 새로운 상수를 추가하거나 순서를 바꿔도 다시 컴파일 하지 않아도 된다
- 문자열로 출력하기 적합하다
    - `toString()`은 출력하기에 적합한 문자열을 내어준다
- 임의의 메서드, 필드를 추가하고 임의의 인터페이스를 구현할 수도 있다
    - Object 메서드들을 높은 품질로 구현해놨다(3장)
    - Comparable(item 14)과 Serializable(12장)을 구현해놨다
    - 직렬화 형태도 웬만큼 변형을 가해도 문제없이 동작하게끔 구현해놨다

## enum type 사용 경우
### 1) 각 상수와 연관된 데이터를 해당 상수 자체에 내재시킬 때
```java
/* 태양계의 여덟 행성 */

@Getter
public enum Planet {
    MERCURY(3.302e+23, 2.439e6),
    VENUS(4.869e+24, 6.052e6),
    EARTH(5.975e+24, 6.378e6),
    MARS(6.419e+23, 3.393e6),
    JUPITER(1.899e+27, 7.149e7),
    SATURN(5.685e+26, 6.027e7),
    URANUS(8.683e+25, 2.556e7),
    NEPTUNE(1.024e+26, 2.447e7);
    
    private final double mass;            // 질량(단위: 킬로그램)
    private final double radius;          // 반지름(단위: 미터)
    private final double surfaceGravity;  // 표면중력(단위: m / s^2)
    
    // 중력상수 (단위: m^3 / kg s^2)
    private static final double G = 6.67300E-11;
    
    // 생성자
    Planet(double mass, double radius) {
        this.mass = mass;
        this.radius = radius;
        this.surfaceGravity = G * mass / (radius * radius);
    }
    
    public double surfaceWeight(double mass) {
        return mass * surfaceGravity;
    }
}
```
- 열거 타입 상수 각각을 특정 데이터와 연관지으려면 생성자에서 데이터를 받아 인스턴스 필드에 저장하면 된다
- 열거 타입은 근본적으로 불변이라 모든 필드는 final이어야 한다(item 17)
- 필드를 public으로 선언해도 되지만 private로 두어 별도의 public 접근자 메서드를 두는게 낫다(item 16)

어떤 객체의 지구에서의 무게를 입력받아 행성에서의 무게를 출력하는 일을 다음같이 짧게 작성할 수도 있다
```java
public classs WeightTable {
	public static void main(String[] args) {
    	double earthWeight = Double.parseDouble(args[0]);
        double mass = earthWeight / Planet.EARTH.surfaceGravity();
        for (Planet p : Planet.values()) 
        	System.out.println("%s에서 무게는 %f이다. %n", p, p.surfaceWeight(mass));
    }
}
```

### 2) 상수마다 동작이 달라져야 할 경우
사칙연산 계산기의 연산 종류를 열거 타입으로 선언하고, 실제 연산까지 열거 타입 상수가 직접 수행하기로 한다
```java
public enum Operation {
    PLUS, MINUS, TIMES, DIVIDE
}
```

### switch문을 이용해 상수의 값에 따라 분기하는 방법
```java
/* 상수가 뜻하는 연산을 수행한다 */
public double apply(double x, double y) {
    switch(this) {
        case PLUS: return x + y;
        case MINUS: return x - y;
        case TIMES: return x * y;
        case DIVIDE: return x / y;
    }
    throw new AssertionError("알 수 없는 연산: " + this);
}
```
- 깨지기 쉬운 코드이다
    - 새로운 상수를 추가하면 해당 case 문도 추가해주어야 한다

### ✅ 상수별 메서드 구현을 활용한 열거 타입
```java
public enum Operation {
    PLUS("+") {
        public double apply(double x, double y) {return x + y;}
    }
    MINUS("-") {
        public double apply(double x, double y) {return x - y;}
    }
    TIMES("*") {
        public double apply(double x, double y) {return x * y;}
    }
    DIVIDE("/") {
        public double apply(double x, double y) {return x / y;}
    };
    
    private final String symbol;
    
    Operation(String symbol) {this.symbol = symbol;}
    
    @Override public String toString() {return symbol;}

    public abstract double apply(double x, double y);
}
```
- 상수별 메서드 구현(constant-specific method implementation)
    - 추상 메서드 `apply()`를 선언하고, 각 상수에서 자신에 맞게 재정의한다
- `toString()`을 재정의해 연산 기호를 반환한다

### 단점
-  열거 타입 상수끼리 코드를 공유하기 어렵다
```java
/* 급여명세서에서 쓸 요일을 표현하는 열거 타입
   직원의 (시간당) 기본 임금 + 그날 일한 시간(분 단위) -> 일당 계산 */

enum PayrollDay {
    MONDAY, TUESDAY, WEDSDAY, THURSDAY, FRIDAY, SATURDAY, SUNDAY;
    
    private static final int MINS_PER_SHIFT = 8 * 60;
    
    int pay(int minutesWorked, int payRate) {
        int basePay = minutesWorked * payRate;
        
        int overtimePay;
        switch(this) {
            case SATURDAY: case SUNDAY: // 주말
                overtimePay = basePay / 2;
                break;
            default: // 주중
                overtimePay = minutesWOrked <= MINS_PER_SHIFT ?
                    0 : (minutesWorked - MINS_PER_SHIFT) * payRate / 2;
        }
        
        return basePay + overtimePay;
    }
}
```
- 관리 관점에서 위험한 코드다
    - OCP 위반
    - 휴가와 같은 새로운 값을 열거 타입에 추가하려면, 그 값을 처리하는 case문을 잊지 말고 쌍으로 넣어줘야 한다

### ✅ 전략 열거 타입 패턴
- 새로운 상수를 추가할 때 잔업수당 '전략'을 선택하도록 하는 것이다
- 열거 타입 상수 일부가 같은 동작을 공유한다면 전략 열거 타입 패턴을 사용하자
```java
enum PayrollDay {
    MONDAY, TUESDAY, WEDSDAY, THURSDAY, FRIDAY, 
    SATURDAY(PayType.WEEKEND), SUNDAY(PayType.WEEKEND);
    
    private final PayType payType;
    
    PayrollDya(PayType payTyoe) {this.payType = payType;}
    
    int pay(int minutesWorked, int payRate) {
    	return payType.pay(minutesWorked, payRate);
    }
    
    /* 전략 열거 타입 */
    enum PayType {
        WEEKDAY {
            int overtimePay(int minusWorked, int payRate) {
                return minusWorked <= MINS_PER_SHIFT ? 0 :
                    (minusWorked - MINS_PER_SHIFT) * payRate / 2;
            }
        },
        WEEKEND {
            int overtimePay(int minusWorked, int payRate) {
                return minusWorked * payRate / 2;
            }
        };
        
        abstract int overtimePay(int mins, int payRate);
        private static final int MINS_PER_SHIFT = 8 * 60;
        
        int pay(int minsWorked, int payRate) {
            int basePay = minsWorked * payRate;
            return basePay + overtimePay(minsWorked, payRate);
        }
    }
}
```
- 잔업 수당 계산을 `PayType`(전략 열거 타입)으로 옮기고, `PayrollDay`(열거 타입)의 생성자에서 이 중 적당한 것을 선택한다
- `PayrollDay`은 잔업수당 계산을 `PayType`에 위임하여, switch 문이나 상수별 메서드 구현이 필요 없게 된다

### 때로는 switch문이 좋은 선택이 될 수도 있다
- 추가하려는 메서드가 의미상 열거 타입에 속하지 않을 때
```java
public static Operation inverse(Operation op) {
    swith(op) {
        case PLUS: return Operation.MINUS;
        case MINUS: return Operation.PLUS;
        case TIMES: return Operation.DIVIDE;
        case DIVIDE: return Operation.TIMES;

        default: throw new AssertionError("알 수 없는 연산: " + op);
    }
}
```