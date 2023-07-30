# item16. public 클래스에서는 public 필드가 아닌 접근자 메서드를 사용하라
> public 클래스는 절대 가변 필드를 노출해서는 안 된다. 불변 필드라면 노출해도 덜 위험하지만 완전히 안심할 수는 없다. 하지만 package-private 클래스나 private 중첩 클래스에서는 종종 필드를 노출하는 편이 나을 때도 있다.

## CASE 1) public 필드
```java
class Point {
  public double x;
  public double y;
}
```

### 단점
- 캡슐화 X
    - 다른 컴포넌트에서 해당 클래스 데이터 필드에 직접 접근이 가능하기 때문이다
    - 참고) 캡슐화: 클래스 안에 서로 연관있는 속성과 기능들을 하나의 캡슐(capsule)로 만들어 데이터를 외부로부터 보호하는 것
- 불변식을 보장할 수 없다
    - 클라이언트가 데이터를 변경할 수 있다
- API를 수정하지 않고는 내부 표현을 바꿀 수 없다
    - 값을 변경하려면 필드에 직접 접근해야 한다
- 외부에서 필드에 접근할 때 부수 작업을 수행할 수 없다
    - Point.x 라는 필드를 조회했을 때, 부수적인 로직(EX. 연산 로직)을 추가할 수가 없다

---

## CASE 2) private 필드 + public 접근자
```java
public class Point {
    private double x;
    private double y;

    public Point(final double x, final double y) {
        this.x = x;
        this.y = y;
    }

    public double getX() {
        return x;
    }

    public void setX(final double x) {
        this.x = x;
    }

    public double getY() {
        return y;
    }

    public void setY(final double y) {
        this.y = y;
    }
}
```

### 장점
- 유연성
    - 클라이언트는 public 메서드를 통해서만 데이터 접근이 가능하다
    - 클래스의 내부 표현 방식은 직접 노출되지 않는다
- 접근자
    - 패키지 바깥에서 접근자(getter/setter)로 접근이 가능하다
    - 클래스의 내부 표현 방식을 언제든지 바꿀 수 있다

---

## CASE 3) private-package 클래스, private nested 클래스
- private-package 클래스(default 클래스) : 같은 패키지 및 같은 클래스에서만 접근 가능
- private nested 클래스 : 내부 클래스가 private 형태로 된 클래스

```java
public class ColorPoint {
    private static class Point{
        public double x;
        public double y;
    }

    public Point getPoint(){   // 클라이언트 코드가 클래스 내부에 묶인다
        Point point = new Point();   // ColorPoint 외부에서는 
        point.x = 1.2;   // Point 클래스 내부 조작이 불가능하다
        point.y = 5.3;

        return point; 
    }
}
```

### 장점
- 데이터 필드를 노출한다 해도 아무런 문제가 없다
    - 외부에서는 Point 클래스의 필드에 접근하는 것이 불가능하지만, ColorPoint에서는 Point 클래스 필드 조작이 가능하다
- 해당 클래스가 표현하려는 추상 개념만 올바르게 표현해주면 된다

### 단점
- 클라이언트 코드가 클래스 내부에 묶인다

---

## CASE 4) final 필드
```java
public final class Time {
    private static final int HOURS_PER_DAY    = 24;
    private static final int MINUTES_PER_HOUR = 60;

    public final int hour;
    public final int minute;

    public Time(int hour, int minute) {
        if (hour < 0 || hour >= HOURS_PER_DAY)
            throw new IllegalArgumentException("Hour: " + hour);
        if (minute < 0 || minute >= MINUTES_PER_HOUR)
            throw new IllegalArgumentException("Min: " + minute);
        this.hour = hour;
        this.minute = minute;
    }
}
```

### 단점
- public 클래스의 필드가 불변이라면 직접 노출할 때의 단점이 조금은 줄어들지만, 여전히 결코 좋은 생각이 아니다
- 불변식은 보장할 수 있게 되지만 여전히 API를 변경하지 않고는 표현 방식을 바꿀 수 없다
- 필드를 읽을 때 부수 작업을 수행할 수 없다
- 참고) 자바 플랫폼 라이브러리에도 public 클래스의 필드를 직접 노출시키지 말라는 규칙을 어긴 사례가 존재한다
    - 예: java.awt.package 클래스의 Point와 Dimension 클래스