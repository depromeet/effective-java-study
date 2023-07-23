> 싱글톤고 같이 불변식을 지키려고 한다면, 열거 타입을 사용하자.
`readResolve` 를 사용한다면 반드시 모든 참조 타입 인스턴스 필드를 `transient` 로 선언해야 한다.

#### 싱글턴 클래스 직렬화 문제
싱글턴 클래스를 직렬화 하려고 `Seriablizable` 를 구현하면, 더 이상 싱글턴이 아니게 된다. 어떤 `readObject` 를 사용하던 클래스가 **초기화될 때 만들어진 인스턴스와는 별개의 인스턴스가 생성**되기 때문이다

`readObject` 메서드는 역직렬화 때 자동으로 호출되기 때문에, 막을 수 없다.


```java
private void readObject(ObjectInputStream s) {
	Object object = s.defaultReadObject();  // 인스턴스 객체 생성
    return (Class) object;
}
```

> 싱글턴 객체는 JVM 내에서 하나여야만 한다!

#### singleton test


```java
public class SerializableTest {

    public byte[] serialize(Object object) {   // 객체 -> byte
        try (
                ByteArrayOutputStream bos = new ByteArrayOutputStream();
                ObjectOutputStream oos = new ObjectOutputStream(bos)
        ) {
            oos.writeObject(object);
            return bos.toByteArray();
        } catch (Exception ex) {
            ex.printStackTrace();
        }
        return null;
    }

    public Object deSerialize(byte[] data) throws IOException {  // byte -> 객체
        try (
                ByteArrayInputStream bis = new ByteArrayInputStream(data);
                ObjectInputStream ois = new ObjectInputStream(bis)
        ) {
            return ois.readObject();
        } catch (Exception ex) {
            ex.printStackTrace();
        }
        return null;
    }
}
```


```java
public class SingletonPerson implements Serializable {

    public static final SingletonPerson INSTANCE = new SingletonPerson(1, "name", 100);

    private SingletonPerson(Integer id, String name, Integer height){
        this.id = id;
        this.name = name;
        this.height = height;
    }

    private final transient Integer id;
    private final String name;
    private final Integer height;
}
```


![](https://velog.velcdn.com/images/semi-cloud/post/b097f18b-ab07-47f9-a1f4-ae6157775254/image.png)


테스트 결과를 보면, 두 객체의 해시 코드 값이 다른 것을 확인할 수 있다.
![](https://velog.velcdn.com/images/semi-cloud/post/44c0fa58-f77f-43a8-870a-51879a7fb7c1/image.png)


#### 싱글턴 클래스 직렬화 문제 해결

`readResolve` 메서드를 통해, 기존 객체의 참조를 반환하면 된다.

`readResolve` 메서드는 **역직렬화 중에 생성된 객체를 다른 객체로 대체**하는 데 사용된다.

따라서 만일 역직렬화 과정에서 자동으로 호출되는 `readObject` 메서드가 있더라도 `readResolve` 메서드에서 반환한 인스턴스로 대체되기 때문에, `readObject` 메서드를 통해 만들어진 인스턴스는 더이상 유지되지 않아 가비지 컬렉션 대상이 되어 사라져 싱글톤을 보장할 수 있다.

![](https://velog.velcdn.com/images/semi-cloud/post/d0e02564-40c4-4f7d-97ed-4b01f6a1cbae/image.png)


https://www.baeldung.com/java-serialization-readobject-vs-readresolve


```java
public class SingletonPerson implements Serializable {
	...
    private Object readResolve() {     // 역직렬화 객체 대신 클래스 초기화 때 만들어진 인스턴스 반환
        return INSTANCE;
    }
}
```

![](https://velog.velcdn.com/images/semi-cloud/post/17718893-97ef-4752-a18d-3bce184ba93e/image.png)

#### readResolve 접근 제한자
1. `private` : `final` 클래스인 경우 사용
1. `package-private` : 같은 패키지에 속한 하위 클래스만 사용 가능
2. `protected / public` : 모든 하위 클래스에서 사용 가능하지만, 재정의 하지 않은 경우 하위 클래스의 인스턴스를 직렬화하면서 상위 클래스의 인스턴스를 생성해 `ClassCastException` 을 일으킬 수 있다.


### 공격
만약 싱글턴이 `non-transient` 참조를 가지고 있다면, 해당 필드의 내용은 `readResolve` 메서드가 **수행되기 이전에 역직렬화**된다. 따라서 그 시점에 역직렬화된 인스턴스의 참조를 가져와서 공격에 이용할 수 있다.

```java
public class Elvis {
    public static final Elvis INSTANCE = new Elvis();
    private Elvis();
    
    private String[] favoriteSongs =
        { "Hound Dog", "Heartbreak Hotel" };
        
    public void printFavorites() {
        System.out.println(Arrays.toString(favoriteSongs));
    }
    
    private Object readResolve() {
    	return INSTANCE;
    }
}
```

```java
public class ElvisImpersonator {
    // 진짜 Elvis 인스턴스로는 만들어질 수 없는 바이트 스트림
    private static final byte[] serializedForm = {
            -84, -19, 0, 5, 115, 114, 0, 20, 107, 114, 46, 115,
          ...
            32, 72, 111, 116, 101, 108
    };

    public static void main(String[] args) {
        Elvis elvis = (Elvis) deserialize(serializedForm);
        Elvis impersonator = ElvisStealer.impersonator;

        elvis.printFavorites();
        impersonator.printFavorites();
    }
}
```
1. 싱글턴이 도둑을 포함하므로 싱글턴이 역질렬화될 때 도둑의 `readResolve` 메서드가 먼저 호출된다.

2. 도둑의 `readResolve` 메서드가 수행될 때 도둑의 인스턴스 필드에는 역직렬화 도중인(그리고 readResolve가 수행되기 전인) 싱글턴의 참조가 담겨 있게 된다.

3. 도둑의 `readResolve` 메서드는 이 인스턴스 필드가 팜조한 값을 정적 필드로 복사하여 `readResolve`가 끝난 후에도 계속 참조할 수 있도록 한다.

4. 그런 다음 이 메서드는 도둑이 숨긴 `transient`가 아닌 필드의 원래 타입에 맞는 값을 반환한다.

5. 이 과정을 생략하면 직렬화 시스템이 도둑의 참조를 이 필드에 저장하려 할 때 VM이 `ClassCastException`을 던진다.

https://github.com/NW-study/effective-java/issues/91
https://stackoverflow.com/questions/37660696/elvisstealer-from-effective-java

### 열거 타입으로 전환하기

`transient` 를 붙이면 문제를 해결할 수 있지만, 열거 타입이 훨씬 나은 방안이다. 해당 `enum` 제외 다른 객체는 존재하지 않음을 자바가 보장해주기 때문에 안전하다.

```java
public enum Elvis {
    INSTANCE;
    private String[] favoriteSongs =
        { "Hound Dog", "Heartbreak Hotel" };
        
    public void printFavorites() {
        System.out.println(Arrays.toString(favoriteSongs));
    }
}
```
