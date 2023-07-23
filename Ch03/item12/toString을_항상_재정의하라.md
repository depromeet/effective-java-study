# toString을 항상 재정의하라

### toString 메서드

``` java
public class Main {
    public static void main(String[] args) {
        Product product = new Product(1, "Toy");
        System.out.println(product);
    }
}
```

- `java.lang.Object` 클래스가 `toString` 메서드가 제공하지만 클래스의 이름과 @ 문자 기호와 16진수로 표현된 해시 코드가 붙은 문자열이 반환
- toString은 디버깅 용도이거나 사람이 읽기 위한 용도로 많이 활용
- 원하는 형식으로 변환해서 반환할 수 있도록 변경
- 포맷을 지정하면 그 포맷에 얽매이게 되기 때문에 유의
- Lombok이나 Intellij를 활용하여 생성하는 것이 가장 편리