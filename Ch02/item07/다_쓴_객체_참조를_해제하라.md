### 자바는 가비지 컬렉터가 있으니 다 쓴 객체 참조를 해제하지 않아도 되나?
C, C++과 같은 언어는 메모리를 직접 관리해야 하기 때문에 다 쓴 객체 참조를 해제해야 한다. </br>
하지만 자바는 가비지 컬렉터가 있기 때문에 다 쓴 객체 참조를 해제하지 않아도 된다. </br>
그렇지만 메모리 관리에 더 이상 신경쓰지 않아도 된다고 오해할 수 있는데, 그렇지 않다.

### 메모리 누수가 일어나는 경우
```java
public class Stack {
    private Object[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;
	
    public Stack() {
        elements = new Object[DEFAULT_INITIAL_CAPACITY];
    }
	
    public void push(Object e) {
        ensureCapacity();
        elements[size++] = e;
    }
	
    public Object pop() {
        if (size == 0) {
            throw new EmptyStackException();
        }
        return elements[--size];
    }
	
    private void ensureCapacity() {
        if (elements.length == size) {
            elements = Arrays.copyOf(elements, 2 * size + 1);
        }
    }
}
```

책에서 총 3가지 메모리 누수가 일어나는 경우를 설명한다.
1. 자체 메모리 관리
2. 캐시
3. 리스너 혹은 콜백

### 1.자체 메모리 관리
위 코드에서 pop을 할 떄 elements에서 값을 꺼내주기만 하지,</br>
객체는 여전히 존재한다.</br>
이렇게 되면 가비지 컬렉터가 이 객체를 회수하지 않는다.

그렇다면 어떻게 해결 할 수 있을까?
```java
public Object pop() {
    if (size == 0) {
        throw new EmptyStackException();
    }
    Object result = elements[--size];
    elements[size] = null; // 다 쓴 참조 해제
    return result;
}
```
위와 같이 다 쓴 객체를 참조 해제하면 가비지 컬렉터가 회수할 수 있다.

### 2.캐시
캐시 역시 메모리 누수를 일으키는 주범이다. </br>
캐시는 자주 사용하는 객체를 임시로 저장해놓는다. </br>
하지만 이 객체를 사용하지 않는다면 캐시에서 제거해야 한다. </br>
하지만 캐시를 구현할 때 이를 고려하지 않는 경우가 많다.

외부에서 키(key)를 참조하는 동안만 엔트리가 살아있는 캐시가 필요한 상황이라면 <br>
WeakHashMap을 사용하면 된다. </br>

WeakHashMap이 뭘까?
```
WeakHashMap은 WeakReference의 특성을 이용하여
HashMap의 Element를 자동으로 제거, GC 해버린다.
(Key에 해당하는 객체가 더이상 사용되지 않는다고 판단되면 제거한다는 의미이다)
```
이런 설명보다는 코드로 보는게 직관적이라 예제를 가져왔다.

```java
public class CacheKey {
    private Integer id;
    private LocalDateTime created;
	
    public CacheKey(Integer id) {
        this.id = id;
        this.created = LocalDateTime.now();
    }
}

public class PostRepository {
    private Map<CacheKey, Post> cache;

    public PostRepository() {
        this.cache = new HashMap<>();
    }
	
    public Post getPostById(Integer id) {
        CacheKey key = new CacheKey(id);
        if (cache.containsKey(key)) {
                return cache.get(key);
        } else {
            // DB에서 읽거나 REST API를 통해 읽어온다.
            Post post = new Post();
            cache.put(key, post);
            return post;
        }
    }

    public Map<CacheKey, Post> getCache() {
        return cache;
    }
}
```
위와 같은 코드가 있을때 아래의 테스트는 성공할까 실패할까?

```java
@Test
void cache() throws InterruptedException {
    PostRepository postRepository = new PostRepository();
    Integer p1 = 1;
    Post post = postRepository.getPostById(p1);
	
    assertFalse(postRepository.getCache().isEmpty()); // 성공
    
    // p1 = null; 주석을 풀어도 실패한다.
    
        
    // TODO run gc
    System.out.println("run gc");
    System.gc();
    System.out.println("wait");
    Thread.sleep(3000L);

    assertTrue(postRepository.getCache().isEmpty()); // 실패
}
```
gc가 실행되어도 cache가 비어있지 않다. </br>
p1의 값을 null로 만들어도 해당 테스트는 실패하게 된다.



그렇다면 HashMap을 WeakHashMap으로 바꾸면 어떻게 될까?
WeakHashMap는 외부에서 키를 참조하지 않으면 바로 GC가 되기 때문에 </br>
위의 테스트는 성공하게 된다.

그렇다면 CacheKey가 외부에서 존재한다면 어떨까?
```java
public class CacheKey {
    private Integer id;
    private LocalDateTime created;
	
    public CacheKey(Integer id) {
        this.id = id;
        this.created = LocalDateTime.now();
    }
}

public class PostRepository {
    private Map<CacheKey, Post> cache;
	
    public PostRepository() {
        this.cache = new HashMap<>();
    }
	
    public Post getPostById(CacheKey key) {
        if (cache.containsKey(key)) {
            return cache.get(key);
        } else {
            // DB에서 읽거나 REST API를 통해 읽어온다.
            Post post = new Post();
            cache.put(key, post);
            return post;
        }
    }

    public Map<CacheKey, Post> getCache() {
        return cache;
    }
}
```

```java
@Test
void cache() throws InterruptedException {
    PostRepository postRepository = new PostRepository();
    CacheKey key1 = new CacheKey(1);
    postRepository.getPostById(key1);
	
    assertFalse(postRepository.getCache().isEmpty()); // 성공

    // key1 = null; // 이 경우에는 참조가 끊어지기 때문에 GC가 된다.
        
    // TODO run gc
    System.out.println("run gc");
    System.gc();
    System.out.println("wait");
    Thread.sleep(3000L);

    assertTrue(postRepository.getCache().isEmpty()); // 실패 (key1 = null; 주석을 풀면 성공)
}
```
위 처럼 테스트를 하면 어떻게 될까?
위에서 Key는 외부에서 참조하고 있기 때문에 GC가 되지 않는다. </br>
즉 메소드가 끝나기 전까지 CacheKey는 GC가 되지 않는다. </br>

### 3.리스너 혹은 콜백
리스너 혹은 콜백을 사용할 때도 메모리 누수가 발생할 수 있다. </br>
리스너는 콜백을 등록만 하고 명확히 해지하지 않는다면 발생한다. </br>

```java
import java.lang.ref.WeakReference;
import java.util.Objects;

public class ChatRoom {
    private List<WeakReference<User>> users;

    public ChatRoom() {
        this.users = new ArrayList<>();
    }

    public void addUser(User user) {
        this.users.add(new WeakReference<>(user));
    }

    public void sendMessage(String message) {
        users.forEach(wr -> Objects.requireNonNull(wr.get()).receive(message));
    }

    public List<WeakReference<User>> getUsers() {
        return users;
    }
}
```
위와 같은 코드가 있다고 가정하자. </br>
message가 왔을 떄 리스너의 목록을 순회하면서 receive를 호출한다. </br>

위 코드에서도 리스너나 콜백을 제거해주는 기능이 없으면,
유저가 떠났다면 컬렉션은 그대로 남아있을 것이다.
하지만 WeakReference를 사용하면 제거가 된다. </br>


#### 핵심정리
메모리 누수는 겉으로 잘 드러나지 않아 시스템에 수년간 잠복하는 사례도 있다. </br>
이런 누수는 철저한 코드 리뷰나 힙 프로파일러 같은 디버깅 도구를 동원해야만 발견되기도 한다. </br>
그래서 이런 종류의 문제는 예방법을 익혀두는 것이 매우 중요하다.

#### 참고자료
[[인프런]이펙티브자바 1부 - 백기선](https://www.inflearn.com/course/%EC%9D%B4%ED%8E%99%ED%8B%B0%EB%B8%8C-%EC%9E%90%EB%B0%94-1)