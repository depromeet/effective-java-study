# item30. 이왕이면 제네릭 메서드로 만들라
> 요약: 제네릭 타입 메서드가 타입 안전하며 사용하기 쉽다. 불필요한 형변환을 없애기 위해 메서드의 매개변수와 반환값에 적절히 제네릭을 사용하여 제네릭 메서드로 만들면 편의성과 안정성을 모두 잡는 좋은 방법이 될 수 있다.

## 제네릭 메서드 장점
- 타입 안전함 보장
- 유연함
- 쓰기가 쉬움

## 로타입은 사용하지 말라([item26 참고](https://github.com/depromeet/effective-java-study/blob/main/Ch05/item26/%EB%A1%9C_%ED%83%80%EC%9E%85%EC%9D%80_%EC%82%AC%EC%9A%A9%ED%95%98%EC%A7%80_%EB%A7%90%EC%9E%90.md))
매개변수의 타입이나 반환 타입으로 로 타입을 사용하게 되면 컴파일은 가능하지만 new HashSet을 하는 과정과 result에 s2를 addAll 하는 과정에서 로 타입에 대한 경고가 발생한다.
```java
public static Set union(Set s1, Set s2) {
    Set result = new HashSet(s1);
    result.addAll(s2);
    return result;
}
```

위 코드는 세개의 Set 집합이 타입이 모두 같아야 한다. 타입을 안전하게 만들어 수정하면 다음과 같다.
```java
public static <E> Set<E> union(Set<E> s1, Set<E> s2) {
    Set<E> result = new HashSet(s1);
    result.addAll(s2);
    return result;
}
```


## 제네릭 싱글턴 팩터리 패턴
자바의 제네릭이 실제화된다면 타입별로 하나씩 만들어야 했겠지만, 소거 방식을 사용한 덕에 제네릭 싱글턴 하나면 충분하다.

### 항등함수
어떤 값이든 자신을 casting하여 반환하는 함수

```java
private static UnaryOperator<Object> IDENTITY_FN = (t) -> t;

@SuppressWarinings("unchecked")
public static <T> UnaryOperator<T> identityFunction(){
    return (UnaryOperator<T>) IDENTITY_FN;
}
```

- `IDENTITY_FN` : `UnaryOperator`를 사용하여 인자 값을 그대로 출력하는 함수
<details>
<summary>참고: UnaryOperator</summary>

- 함수형 인터페이스
- `Type T`의 인자 하나를 받고, 동일한 `Type T` 객체를 리턴하는 함수형 인터페이스
- 입력값을 변환하거나 수정할 때 유용

```java
UnaryOperator<Integer> square = (x) -> x * x;
int result = square.apply(5);  // 입력값 5를 제곱하여 결과값 25를 반환
```
</details>

<details>
<summary>참고: @SuppressWarinings("unchecked")</summary>

- `@SuppressWarinings`: 특정 경고 무시
- `unchecked`: 제네릭 타입의 안전성 검사가 이루어지지 않은 경우 발생
- 경고 발생 원인: `UnaryOperator<T>` 타입이 `UnaryOperator<Object>`와 일치하지 않을 수 있음
- 따라서 unchecked 경고를 무시하기 위해 `@SuppressWarinings("unchecked")` 을 사용함
</details>

### 항등함수 실행
```java
String[] stringArr = {"A", "B", "C"};
UnaryOperator<String> u1 = IdentityFunction.identityFunction();
for (String s : stringArr) {
    System.out.println(u1.apply(s));   // A B C 출력
}
```

```java
int[] intArr = {1, 2, 3, 4};
UnaryOperator<Integer> u2 = IdentityFunction.identityFunction();
for (int i : intArr) {
    System.out.println(u2.apply(i));  // 1 2 3 4 출력
}
```

## 재귀적 한정적 타입
- 재귀적 한정적 타입: 자신이 들어간 표현식을 사용하여 타입 매개변수의 허용 범위를 한정한다. 
- 주로 타입의 자연적 순서를 지정해주는 Comparable과 함께 사용된다.
```java
public interface Comparable<T>{
	int compareTo(T o);
}
```
- 타입 매개변수 `T` : 비교할 수 있는 원소의 타입을 정의
    - 실제 거의 모든 타입은 자기 자신과 같은 타입만 비교가 가능하다.
    - `String`은 `Comparable<String>`을 구현하고 `Integer`는 `Comparable<Integer>`를 구현한다.

### 재귀적 타입 한정을 이용한 최댓값 계산 메서드
```java
public static <E extends Comparable<E>> E max(Collection<E> c){
    if(c.isEmpty()){
       throw new IllegalArgumentException("컬렉션이 비었습니다.");
    }
        
    E result = null;
    for (E e : c){
        if(result == null || e.compareTo(result) > 0){
            result = Objects.requireNonNull(e);
        }
    }
    
    return result;
}
```

- `<E extends Comparable<E>>`
    - 자신과 비교할 수 있음을 의미한다.
    - 정렬, 검색 등 기능을 수행하기 위해 컬렉션에 담긴 모든 원소가 상호 비교되어야 한다.
    - 따라서 재귀적 타입 한정을 이용해 상호 비교 할 수 있음을 표현한다.