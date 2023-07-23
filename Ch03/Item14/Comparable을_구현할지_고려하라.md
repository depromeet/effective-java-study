# item14. Comparable을 구현할지 고려하라

Comparable 인터페이스를 구현한 클래스의 인스턴스는 자연적인 순서를 가지게 된다.

Comparable 인터페이스는 compareTo라는 메서드를 하나 가진다.
compareTo 메서드는 동치성 비교와 순서 비교가 가능하다.

자바의 거의 모든 값 클래스와 열거 타입이 Comparable을 구현한 상태이다.

### CompareTo 메서드의 일반 규약
#### 동치성
인스턴스 자기 자신을 compareTo로 비교한 값은 0이 나와야 한다.
```
n1.compareTo(n1)
```
#### 대칭성
인스턴스 두개의 비교 순서를 뒤집으면 항상 반대의 값이 나와야 한다.
```
n1.compareTo(n2) // 양수
n2.compareTo(n1) // 음수
```
#### 추이성
```
n1.compareTo(n2) > 0 
n2.compareTo(n3) > 0
// 추이성을 고려했을때의 결과
n1.compareTo(n3) > 0
```
여기서 중요시 봐야할 부분은 compareTo와 equals가 같은 동치성을 비교하지 않을 수도 있다는 점이다.
```
 public static void main(String[] args) {
        // 스케일이 다른 숫자를 넣는다.
        BigDecimal bd1  = new BigDecimal("1.0");
        BigDecimal bd2 = new BigDecimal("1.00");

        HashSet<BigDecimal> hset = new HashSet<>();
        TreeSet<BigDecimal> tset = new TreeSet<>();

        hset.add(bd1);
        hset.add(bd2);
        System.out.println("hset cnt is " + hset.size());

        tset.add(bd1);
        tset.add(bd2);
        System.out.println("tset cnt is " + tset.size());
}
```
위의 코드의 결과를 보자
```
hset cnt is 2
tset cnt is 1
```
HashSet 클래스의 경우 내부적으로 순서를 고려하지 않기 때문에 동치성 비교시 equals를 사용한다.<br>
equals 메서드는 동치성을 비교할때 스케일, 즉 소수점 정보까지 모두 고려해서 같은지 비교한다.<br>
따라서 HashSet은 두 BigDecimal이 다르다고 판단했고 개수는 2개가 나왔다.


반대로 TreeSet의 경우 내부적으로 정렬을 하기 때문에 동치성 비교에 compareTo를 사용한다.<br>
compareTo는 비교 시 스케일 정보를 포함하지 않으므로 두 BigDecimal을 같다고 판단한다.

compareTo 비교를 활용하는 클래스
TreeSet, TreeMap
Collections, Arrays

### 상속 관계에서의 Comparable 구현
Comparable을 구현한 부모 클래스를 상속한 자식 클래스는 다시 Comparable을 구현하지 못한다.

해결 방법
상속을 유지할 시, 정렬이 필요한 클래스(ex> TreeSet)에서 익명 클래스로 Comparator 구현<br>
상속을 사용하지 않을 시, 컴포지션을 사용해서 Comparable 구현

### CompareTo 내부 구현 방법
자바 7 이전에는 compareTo 내부에서 정수 타입을 비교할때는 관계 연산자인 >와 <를, 실수 타입을 비교할때는 Double.compare와 Float.compare를 사용하게 함.

자바 7 이후부터는 박싱된 기본 클래스의 compare 정적 메서드 사용하면 된다.

### 자바 8부터의 CompareTo와 Comparator의 조합
Comparator의 비교자 생성 메서드를 통해서 compareTo 메서드에 정렬 기준을 부여해줄 수 있다.
```java
 private static final Comparator<PhoneNumber> COMPARATOR =
            comparingInt((PhoneNumber pn) -> pn.areaCode) 
                    .thenComparingInt(pn -> pn.getPrefix()) 
                    .thenComparingInt(pn -> pn.lineNum);

    @Override
    public int compareTo(PhoneNumber pn) {
        return COMPARATOR.compare(this, pn);
    }
```
Comparator.comparingInt 정적 메서드는 int 타입의 키를 추출하는 함수를 람다로 전달 받아서 그 키를 기준으로 비교자를 반환해준다.<br>
여기서는 pn의 areaCode를 기준으로 전화번호 순서를 정하는 비교자를 반환해준다. 그 다음부터 메서드 체이닝을 통해 계속 비교자를 반환할 수 있다.


실제 사용하는 코드에서는 COMPARATOR.compare(호출하는 자신, 비교할 대상)을 호출해서 비교 기준을 compareTo에 반환해준다.

### Comparator 사용시 주의할 점
```
static Comparator<Object> hashCodeOrder = new Comparator<>(){

 public int compare(Object o1, Object o2)
 {
   return o1.hashcode() - o2.hashcode();
 }
}
```
이렇게 단순히 두 값의 차이를 반환하는 메서드는 만들면 안된다.<br>
정수 오버플로나 부동소수점 계산 방식에 따른 오류를 낼 수 있다.

### 핵심정리
순서를 고려해야 하는 값 클래스를 작성한다면 꼭 Comparable 인터페이스를 구현하여,<br>
그 인스턴스들을 쉽게 정렬하고, 검색하고, 비교 기능을 제공하는 컬렉션과 어우러지도록 해야 한다.<br>
compareTo 메서드에서 필드의 값을 비교할때 <와 > 연산자는 쓰지 말자.<br>
그 대신 박싱된 기본 타입 클래스가 제공하는 정적 compare 메서드나 Comparator 인터페이스가 제공하는 비교자 생성 메서드를 사용하자.