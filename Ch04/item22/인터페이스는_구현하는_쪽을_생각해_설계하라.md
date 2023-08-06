# item21. 인터페이스는 구현하는 쪽을 고려해 설계해라
> 디폴트 메서드는 구현 클래스에 대해 아무것도 모른 채 합의 없이 무작정 ‘삽입’될 뿐이다. 인터페이스 설계 시 주의가 필요하다. 인터페이스 릴리즈 전에 테스트를 거치자.

## default method Interface
```java
public interface Interface {
   // 추상 메서드 
    void abstractMethodA();
    void abstractMethodB();

	// default 메서드
    default int defaultMethodA(){
    	...
    }
}
```

### 장점
- 자바 8부터 디폴트 메서드를 통해서 기존 인터페이스에 메서드를 추가할 수 있게 되었다
- 기존에 존재하던 인터페이스를 이용해서 구현된 클래스를 만들고 사용하고 있는데, 인터페이스를 보완하는 과정에서 추가적으로 구현 해야 할 혹은 필수적으로 존재해야 할 메소드가 있다면, 이미 이 인터페이스를 구현한 클래스와의 호환성이 떨어지게 된다
- default 메소드를 추가하게 된다면 하위 호환성은 유지되고 인터페이스의 보완을 할 수 있다
- 확장에 개방(Open)되어 있고, 변경에 닫혀(Close)있는 코드를 설계할 수 있게 된다(OCP)

### 단점
- 디폴트 메서드를 선언하면, 그 인터페이스를 구현한 후 디폴트 메서드를 재정의하지 않은 모든 클래스에서 디폴트 구현이 쓰이게 된다
- 디폴트 메서드를 추가한다고 해도 기존 구현체들과 매끄럽게 연동된다는 보장은 없다
- 디폴트 메서드는 구현 클래스에 대해 아무것도 모른 채 합의 없이 무작정 ‘삽입’될 뿐이다


## Collection의 removeIf 디폴트 메서드
```java
default boolean removeIf(Predicate<? super E> filter) {
        Objects.requireNonNull(filter);
        boolean result = false;
        for (Iterator<E> it = iterator(); it.hasNext();){
            if(filter.test(it.next())){
                it.remove();
                result = true;
            }
        }
        return result;
}
```
- 동기화와 관련된 코드가 전혀 없다
- Collection를 구현하고 동기화를 제공하는 `Collection.synchronizedCollection` 에서 디폴트 메서드 removeIf가 제대로 동작하지 않는다
- `ConcurrentModificationException`이 발생하거나 다른 오류가 발생할 가능성이 있다
- 한 스레드가 어떤 Collection을 반복자(iterator)를 이용하여 순회하고 있을때, 다른 한스레드가 해당 Collection에 접근하여 변경을 시도하는 경우이다

## 디폴트 메서드 동기화 유지
- `Collection.synchronizedCollection`이 반환하는 package-private 클래스들은 removeIf를 재정의하고, 이를 호출하는 다른 메서드들은 디폴트 구현을 호출하기 전에 동기화 하도록 했다
```java
@Override
    public boolean removeIf(final Predicate<? super E> filter) {
        synchronized (lock) {
            return decorated().removeIf(filter);
        }
    }
```

## 디폴트 메서드는 오류를 일으킬 가능성이 있다
- 디폴트 메서드는 컴파일에 성공하더라도 기존 구현체에 런타임 오류를 일으킬수 있다
- 기존 인터페이스에 디폴트 메서드로 새 메서드를 추가하는 일은 피해야 한다
- 반면, 새로운 인터페이스를 만드는 경우라면 표준적인 메서드 구현을 제공하는데 아주 유용한 수단이며, 그 인터페이스를 더 쉽게 구현해 활용할 수 있게끔 해준다

## 인터페이스 설계 시 주의가 필요하다
- 디폴트 메서드라는 도구가 생겼지만 인터페이스를 설계할 때는 여전히 주의를 기울일 필요가 있다
- 기존 인터페이스에 디폴트 메서를 추가할 시 어떤 위험이 딸려 올지 알 수 없기 때문이다

## 인터페이스 릴리즈 전에 테스트를 거치자
- 새로운 인터페이스라면 릴리스 전에 반드시 테스트를 거쳐야 한다
- 서로 다른 방식으로 최소 세가지는 구현해보는 것을 추천한다
- 다양한 클라이언트도 만들어 보는 것이 좋다