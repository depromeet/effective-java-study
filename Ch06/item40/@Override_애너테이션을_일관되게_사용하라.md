# @Override 애너테이션을 일관되게 사용하라

```java
public class Bigram {
    private final char first;
    private final char second;

    public Bigram(char first, char second) {
        this.first = first;
        this.second = second;
    }

    public boolean equals(Bigram bigram) {
        return bigram.first == first && bigram.second == second;
    }

    public int hashCode() {
        return 31 * first + second;
    }

    public static void main(String[] args) {
        Set<Bigram> s = new HashSet<>();
        for (int i = 0; i < 10; i++) {
            for (char ch = 'a'; ch <= 'z'; ch++) {
                s.add(new Bigram(ch, ch));
            }
        }
        System.out.println(s.size());
    }
}
```

- equals와 hashcode메서들르 함께 재정의 하였음
- 하지만, equals를 재정의한 것이 아니라 다중 정의 하였음
- Object의 eqauls를 재정의 하려면 매개변수 타입을 Object로 해야함
- Object의 equals를 사용해 같은 소문자 바이그램이 각각 다른 객체로 인식
- @Override 어노테이션을 통해서 재정의 의도를 표시해야함
- 상위 클래스를 재정의 할때는 무조건 @Override 어노테이션을 표기할 것