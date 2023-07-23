# item 11. equals를 재정의하려거든 hashCode도 재정의하라
> equals를 재정의한 클래스는 hashCode도 재정의 해야 한다. 재정의한 hashCode는 Object의 API 문서에 기술된 일반 규약을 따라야 하며, 서로 다른 인스턴스라면 되도록 해시코드도 서로 다르게 구현해야 한다.


### hashCode 일반 규약
- equals 비교에 사용되는 정보가 변경되지 않았다면, hashCode 도 변하면 안 된다.
    - (애플리케이션을 다시 실행한다면 이 값이 달라져도 상관 없음)
- equals가 두 객체가 같다고 판단했다면, 두 객체의 hashCode는 똑같은 값을 반환한다.
    - → 논리적으로 같은 객체는 같은 해시코드를 반환해야 한다.
- equals가 두 객체를 다르다고 판단했더라도, hashCode는 꼭 다를 필요는 없다.
    - 하지만, 다른 객체에 대해서는 다른 값을 반환해야 해시테이블의 성능이 좋아진다.


### 논리적으로 같은 객체는 같은 해시코드를 반환해야 한다
```java
Map<PhoneNumber, String> map = new HashMap<>();
map.put(new PhoneNumber(010, 1234, 5678), new Person("시연"));

map.get(new PhoneNumber(010, 1234, 5678));  // null임
```
- PhoneNumber 클래스는 hashCode를 재정의하지 않았기 때문에, **논리적 동치인 두 객체가 서로 다른 해시코드를 반환**하여 get 메서드는 엉뚱한 해시 버킷에 가서 객체를 찾으려 한 것이다.
- HashMap은 해시코드가 서로 다른 엔트리끼리는 동치성 비교를 시도조차 않도록 최적화 되어 있다.


### 동치인 모든 객체에서 똑같은 hashCode를 반환하는 코드
```java
@Override 
public int hashCode() {
    return 42;
}
```
- 모든 객체에게 똑같은 값만 내어주므로 모든 객체가 해시 테이블의 버킷 하나에 담겨 마치 연결리스트처럼 동작한다.
- 그 결과 평균 수행 시간이 O(1)인 해시테이블이 O(n)으로 느려져서, 도저히 쓸 수 없게 된다.


### Objects.hash() 메서드
```java
@Override
public int hashCode() {
    return Objects.hash(lineNum,prefix,areaCode);
}
```
- 속도가 느리다.
- 내부적으로는 Objects.hash() 메서드가 인수들을 배열로 만들고, 이 배열을 이용하여 해시코드를 계산한다. 
    - 객체를 감싸는 임시 객체로 박싱을 수행함
- 박싱과 언박싱 과정은 성능 저하를 가져올 수 있다. 
    - 본 타입에서 해당 객체를 생성하거나, 객체를 기본 타입으로 변환하기 위해 불필요한 오버헤드가 발생하기 때문


### 좋은 해시 함수라면 서로 다른 인스턴스에 다른 해시코드를 반환한다
좋은 hashCode를 작성하는 요령
1. int 변수인 result를 선언한 후 값을 c로 초기화한다.
    - 이 때, c는 해당 객체의 첫번째 핵심 필드를 단계 2.1 방식으로 계산한 해시코드이다.
    - 여기서 핵심 필드는 equals 비교에 사용되는 필드를 말한다.
2. 해당 객체의 나머지 핵심 필드인 f 각각에 대해 다음 작업을 수행한다.
    1. 해당 필드의 해시코드 c 를 계산한다.
        - 기본 타입 필드라면, Type.hashCode(f)를 수행한다. 여기서 Type은 해당 기본타입의 박싱 클래스다.
        - 참조 타입 필드면서, 이 클래스의 equals 메소드가 이 필드의 equals를 재귀적으로 호출하여 비교한다면, 이 필드의 hashCode를 재귀적으로 호출한다.
        - 필드가 배열이라면, 핵심 원소 각각을 별도 필드처럼 다룬다. 모든 원소가 핵심 원소라면 Arrays.hashCode를 사용한다.
    2. 단계 2.1에서 계산한 해시코드 c로 result를 갱신한다.
        - result = 31 * result + c;
3. result를 반환한다.


```java
@Override
public int hashCode() {
    int result = Integer.hashCode(areaCode);
    result = 31 * result + Integer.hashCode(prefix);
    result = 31 * result + Integer.hashCode(lineNum);
    return result;
}
```
- 동치인 PhoneNumber 인스턴스는 서로 같은 해시코드를 가진다.


### hashCode의 캐싱과 지연 초기화
- 클래스가 불변이고 해시코드를 계산하는 비용이 크다면, 매번 새로 계산하기 보다 캐싱을 고려해야 한다.
    - → 이 타입의 객체가 주로 해시의 키로 사용될 것 같다면 인스턴스가 만들어질 때 해시코드를 계산해 둔다.
- 해시의 키로 사용되지 않는 경우라면 hashCode가 처음 불릴 때 계산한느 지연 초기화하면 좋다.
    - → 필드를 지연 초기화 하려면 그 클래스가 thread-safe가 되도록 동기화에 신경 쓰는 것이 좋다.

```java
private int hashCode;

@Override
public int hashCode() {
      	int result = hashCode;  // 캐시된 해시코드 값을 가져옴
        if(result == 0) {  // 캐시된 값이 없다면 새로 계산
        int result = Integer.hashCode(areaCode);
        result = 31 * result + Integer.hashCode(areaCode);
        result = 31 * result + Integer.hashCode(areaCode);
        hashCode = result;
        }
        return result;
}
```
- 해시코드를 지연 초기화하는 hashCode 메서드