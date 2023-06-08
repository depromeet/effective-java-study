

### 용어정리
- 공변(convariance) : T1이 T의 하위 타입이면, T대신 T1를 사용할 수 있는 성질 (조상 -> 자식)
- 반공변(contravariance) : T1이 T의 하위 타입이면, T1대신 T를 사용할 수 있는 성질 (자식 -> 조상)
- 무공변, 불공변 :  관계가 없음


클래스는 상하위 관계가 존재, 제너릭은 상하관계가 존재 하지 않음

클래스의 상하위 관계 - 상속
```java
public class Electronics {
}

public class Tv extends Electronics {
    private String title;
    public Tv(String title) {
        this.title = title;
    }
    public String getTitle() {
        return title;
    }
}

public class Radio extends Electronics {
    private String name;
    public Radio(String name) {
        this.name = name;
    }
    public String getName() {
        return name;
    }
}
```

제너릭을 사용할 경우
```java
Tv tv = new Tv("티비"); 
Radio radio = new Radio("라디오"); RemoteController<Electronics> tvRemoteController = new RemoteController<Tv>(tv); //compile error 
RemoteController<Electronics> radioRemoteController = new RemoteController<Radio>(radio); //compile error
```

**제네릭의 불공변** 때문에 `RemoteController <Electronics>`와 `RemoteController <Tv>`는 전혀 상관없는 타입으로 인정


그럼 불공변성 때문에, 상하위 관계에 대해서 사용하기 어려운 문제가 존재!!!

자바는 이 부분을 해결하기 위해서 와일드 카드를 제공


### 와일드 카드 정리
##### 비 한정적 와일드 카드
`Generic type<?>`   : 타입 제한 X

```java
Tv tv = new Tv("티비"); 
Radio radio = new Radio("라디오"); 
RemoteController<?> tvRemoteController = new RemoteController<Tv>(tv); 
RemoteController<?> radioRemoteController = new RemoteController<Radio>(radio);
```


##### 한정적 와일드카드
`Generic type<? extend T>`  : T또는 T1를 상속받은 타입만 사용가능

```java
Tv tv = new Tv("티비"); 
Radio radio = new Radio("라디오");
RemoteController<? extends Electronics> tvRemoteController = new RemoteController<Tv>(tv); 
RemoteController<? extends Electronics> radioRemoteController = new RemoteController<Radio>(radio);
```


`Generic type<? super T>`  : T또는 T1의 조상 타입만 사용가능

```java
Electronics electronics = new Electronics(); 
RemoteController<? super Tv> remoteController = new RemoteController<Electronics>(electronics);
```



### PECS(Producer extends, Consumer super)
Producer - 데이터를 생산해내는(데이터 조회)
Consumer - 데이터를 소비하는(데이터 저장, 수정)

Produce <-> Extends
Consumer <-> Super


##### Max 예시
- 

## 타입 매개변수와 와일드 카드

```java
public static <E> void swap(List<E> list, int i, int j);
public static void swap(List<?> list, int i, int j);
```

- 어떤게 좋은 코드 일까? : Public ->외부에서 제너릭에 대해서 전혀 신경쓸게 없음(추상화)
- 메서드 선언에 타입 매개변수가 한 번만 나오면 와일드 카드로 대체하라.
- 이때 비한정적 타입 매개변수라면 비한정적 와일드카드로, 한정적 타입 매개변수라면 한정적 와일드카드를 사용하라.

```java
public static void swap(List<?> list, int i, int j){
	swapHelper(list, i, j);
}

private static <E> void swapHelper(List<E> list, int i, int j) {
	list.set(i, list.set(j, list.get(i)));
}

```

### 참고

https://dev.gmarket.com/28
