
>`Cloneable`은 문제가 많으므로 새로운 인터페이스를 만들 때는 `Cloneable` 확장 및 구현해서는 안된다. 따라서 복제 기능은 생성자와 팩터리를 이용하는게 최고이지만, 예외적으로 배열은 `clone` 을 통해 복제하는 것이 좋다.


`Cloneble` 이란,  **복제해도 되는 클래스임**을 명시하는 용도의 믹스인 인터페이스를 의미한다.

```java
public interface Cloneable {
}
```
인터페이스 안에 메서드가 없는데, 바로 이게 문제가 된다. `clone` 메서드는 `Object`에 선언되어 있이며 심지어 `protected` 메서드이기 때문에 단순 `Cloneable`을 구현하는 것만으로는 외부 객체에서 메서드 호출이 불가능하기 때문이다.

```java
public class Object {
    @HotSpotIntrinsicCandidate
    protected native Object clone() throws CloneNotSupportedException;
}
```

하지만 이러한 문제점에도 불구하고, `Cloneable` 방식은 널리 쓰인다.


### ☁️ Cloneable 인터페이스 역할

메서드가 존재하지 않지만, `Object` 의 메서드인 `clone`의 동작 방식을 결정한다.

`Cloneable` 을 구현한 클래스의 인스턴스에서 `clone()` 을 호출하면 그 객체의 필드들을 하나하나 복사한 객체를 반환하며, 그렇지 않은 클래스의 인스턴스에서 호출하면 `CloneNotSupportedException` 을 던진다. 

객체의 복사본을 생성해 반환한다는 것은, 복사본이 원본 객체와 달라야 함을 의미한다.
즉 반환된 객체와 원본 객체는 독립적인 주소를 가지고 있으면서 논리적으로는 동일해야 한다. 

```java
x.clone() != x   // True
x.clone().getClass() == x.getClass()  // True 
x.clone.equals(x)  // True
```

### ☁️ Clone 메서드의 문제점
문제는 클래스의 **하위 클래스**에서 `super.clone()`을 호출하는 경우에 발생한다.

예를 들어 클래스 B가 클래스 A를 상속할때, 하위 클래스인 B의 `clone()` 은 B 타입 객체를 반환하지만 상위 클래스 A의 `clone()` 역시 A 타입 객체를 반환할 수 밖에 없다. **즉, 연쇄 호출의 clone은 처음 호출된 상위 클래스의 객체가 만들어지는 문제**가 있는 것이다. 

만약 상위 클래스가`final` 이라면 상속 조차 안되니 걱정하지 않아도 된다.

### ☁️ Clone 메서드 구현 방법

> clone 메서드를 가진 상위 클래스를 상속하여 `Cloneable` 을 어떻게 제대로 구현할 수 있을까?

클래스가 불변인가, 아니면 가변 필드를 하나라도 가지고 있느냐에 따라 두가지로 나눠진다. 가변 필드를 가지고 있는 경우에도, `clone` 재귀 호출과 방어적 복사 사용 두가지 방법이 존재한다.

#### 1. 가변 상태 참조하지 않는 경우

`super.clone()` 을 호출해 얻은 원본의 완벽한 복제본은, 만약 모든 필드가 기본 타입이거나, 불변 객체를 참조한다면 더이상 손볼 것이 없다. 

하지만, **불변 클래스**는 굳이 `clone()` 메서드를 제공하지 않는 것이 좋다는 관점을 고려하면 다음처럼 구현 가능하다.


```java
public final class PhoneNumber implements Cloneable {
    private final short areaCode, prefix, lineNum;
    
    @Override public PhoneNumber clone() { // Object
        try {
            return (PhoneNumber) super.clone();  // 형변환 처리
         } catch (CloneNotSupportedException e) { // 검사 예외(체크 예외)
            throw new AssertionError();  // 일어날 수 없는 일이다.
        }
    }
}
```
`return (PhoneNumber) super.clone()` 을 보면 재정의한 메서드의 반환 타입은, 상위 클래스의 메서드가 반환하는 타입의 하위 타입일 수 있기 때문에, 단순 `Object` 가 아닌 하위 타입을 반환하여 클라이언트가 형변환하지 않아도 되게 해주었다.


#### 2. 가변 상태를 참조하는 경우

```java
public class Stack implements Cloneable {
    private Object[] elements;  // 가변 필드
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    public Stack() {
        this.elements = new Object[DEFAULT_INITIAL_CAPACITY];
    }

    public void push(Object e) {
        ensureCapacity();
        elements[size++] = e;
    }
    
    public Object pop() {
        if (size == 0)
            throw new EmptyStackException();
        Object result = elements[--size];
        elements[size] = null; // 다 쓴 참조 해제
        return result;
    }

    public boolean isEmpty() {
        return size ==0;
    }

    private void ensureCapacity() {
        if (elements.length == size)
            elements = Arrays.copyOf(elements, 2 * size + 1);
    }
}
```

1. super.clone 사용해서 복사

해당 클래스를 단순히 `super.clone()` 을 통해 복제하는 경우, `elements` 필드가 **원본 인스턴스와 똑같은 배열을 참조**하게 되어 불변식을 해친다. (원본이나 복제본 중 하나를 수정하면 다른 하나도 수정된다)

2. 생성자 사용해서 복사

`Stack`의 생성자를 호출한다면, 생성자에서 원본 객체를 건드리지 않은 채 복제된 객체의 불변식을 보장할 수 있다.
사실 `clone`도 생성자랑 같은 효과를 내기 때문에 스택 내부 정보 자체를 `clone` 을 통해 복사하면 된다.

복잡한 가변 객체를 복제하는 방법은 다음과 같이 두 가지가 있다. 

> #### 1) 복잡한 가변 객체 복제 : 배열 clone의 재귀 호출

`elements` 배열의 `clone` 을 재귀적으로 호출해주는 방식이다.


```java
    @Override public Stack clone() {
        try {
            Stack result = (Stack) super.clone();
            result.elements = elements.clone();
            return result;
        } catch (CloneNotSupportedException e) {
            throw new AssertionError();
        }
    }
 ```
배열의 `clone` 메서드는 런타임 타입과 컴파일타입 모두 원본 배열과 똑같은 배열을 반환하기 때문에,`Object[]` 로 형변환할 필요는 없다.

하지만 복제 가능한 클래스를 만들기 위해서는 일부 필드에서 `final` 한정자를 제거해야 하므로 `가변 객체를 참조하는 필드는 final로 선언하라` 라는 용법에 어긋난다.

 
> #### 2) 복잡한 가변 객체 복제 :  깊은 복사(Deep Copy)

깊은 복사란, **실제 값을 새로운 메모리의 공간에 복사**하는 것을 말한다. 반대로 얕은 복사(Shallow copy)는 **주소값 자체를 복사**하는 방식이다. 즉, 새로운 메모리에 할당되지 않는다.

**해시테이블용 clone 메서드**를 예시로 들어보자. 해시테이블 내부는 버킷들의 배열이고, 각 버킷은 키-값 쌍을 담는 연결 리스트의 첫 번째 엔트리를 참조하고 있다.

```java
public class HashTable implements Cloneable{
  private Entry[] buckets = ...;  // 가변 필드
  
  private statis class Entry {
    final Object key;
    Object value;
    Entry next;
    
    Entry(Object Key, Object value, Entry next){
    	this.key = key;
        this.value = value;
        this.next = next;
    }
 }
 ...
}
```
위 예제의 경우 `buckets.clone()` 만으로는 부족하다. 
 
복제본은 자신만의 버킷 배열을 갖지만, 각 데이터의 주소값을 복사하기 때문에 원본과 **같은 연결 리스트를 참조**하여 원본과 복제본 모두 예기치 않게 동작할 가능성이 생긴다. 따라서, 아래와 같이 각 버킷을 구성하는 **연결 리스트 내용 자체를 복사**해야 한다.

```java
public class HashTable implements Cloneable{
  private Entry[] buckets = ...;
  
  private statis class Entry {
  	 ...
     //이 엔트리가 가르키는 연결 리스트를 재귀적으로 복사
    Entry deepCopy(){
       return new Entry(key, value,
       		next == null ? null : next.deepCopy());
    }
 }
 
 @Override public HashTable clone() {
	try{
    	HashTable result = (HashTable) super.clone();
        result.buckets = new Entry[buckets.length];  // 새로운 버킷 배열 할당
        for (int i=0 ; i < buckets.length; i++)
        	if (buckets[i] != null)
            	result.buckets[i] = buckets[i].deepCopy();   // 비지 않은 각 버킷에 대해 방어적 복사
        return result;
    } catch (CloneNotSupportedException e) {
            throw new AssertionError();
    } 

}
```

하지만, `buckets[i].deepCopy()` 와 같은 경우 재귀 호출로 스택 오버플로우를 일으킬 수 있으니 `deepCopy` 대신 반복자를 써서 순회하는 것이 좋다.

```java
Entry deepCopy(){
	Entry result = new Entry(key, value, next);
    for (Entry p = result; p.next != null; p = p.next)
    	p.next = new Entry(p.next.key, p.next.value, p.next.next);
    return result;
}
```

> **3) 복잡한 가변 객체 복제: 고수준 API 활용**

먼저, `super.clone()` 을 호출하여 얻은 객체의 모든 필드를 초기 상태로 설정 한다음 **원복 객체의 상태를 다시 생성**하는 고수준 메서드들을 호출한다.

>
ex) HashTable에서는 buckets 필드를 새로운 버킷 배열로 초기화 한 후 원본 테이블에 담긴 모든 키-값 쌍 각각에 복제본 테이블의 `put(key,value)` 메서드를 호출

하지만, 생성자와 마찬가지로 `clone()` 함수 내부에서도 하위 클래스에서 재정의될 수 있는 메서드를 호출하지 않아야 한다. 즉 하위 클래스의 오버라이딩을 막아야 하므로  `put(key,value)` 메서드는 `final` 이거나 `private` 이어야 한다.

### ☁️ 총 정리
clone 재정의 방법을 다시 한번 정리해보자.

1. Cloneable을 구현하는 모든 클래스는 clone을 재정의해야 한다.

접근 제한자는 `public`, 반환 타입은 클래스 자기 자신으로 변경한다.

2. 가장 먼저 `super.clone` 을 호출한 후, 필요한 필드를 전부 적절히 수정한다.

기본 타입 필드와 불변 객체 참조만 갖는 클래스라면, 여기서 끝내도 된다.(단, 일련번호나 고유 ID는 불변이어도 정해줘야 한다.)

가변 객체가 존재한다면 모든 가변 객체를 복사하고 복제본이 가진 객체 참조 모두가 복사된 객체들을 가리키게 한다. 이러한 내부 복사는 주로 clone을 재귀적으로 호출해 구현한다. 

### ☁️ Clone의 대안 : 복사 생성자와 복사 팩터리

현재까지 `Cloneable` 문제점을 정리하면 다음과 같다.

1. 기본 구현 `Object.clone()` 얕은 복사본을 반환한다. 
2. `Cloneable` 구현을 강요한다.
3. `CloneNotSupportedException` 과 같은 체크 예외를 발생시키므로 이에 대한 처리가 따로 필요하다.
4. `Object.clone()` 반환 Object 반환된 개체 참조를 형변환 해야 한다.

**복사 생성자(변환 생성자)와 복사 팩터리(변환 백터리)**는 이러한 문제점들을 모두 해결해준다. 단순히 복사 생성자란, 단순히 자신과 같은 클래스의 인스턴스를 인수로 받는 생성자를 말한다.


#### 복사 생성자
```java
class Student
{
    private String name;
    private int age;
    private Set<String> subjects;   // 가변 필드
 
    public Student(Student student)
    {
        this.name = student.name;
        this.age = student.age;
        this.subjects = new HashSet<>(student.subjects); // Deep Copy
    }
}
```

#### 복사 팩터리
```java
public static Student newInstance(Student student) {
        return new Student(student);
}
```

더불어 원본의 구현 타입에 얽매이지 않고 복제본의 타입을 적절히 선택도 가능하다.

https://www.techiedelight.com/ko/copy-constructor-factory-method-java/#:~:text=복사%20생성자는%20기존%20개체,것은%20매우%20좋은%20방법입니다.
