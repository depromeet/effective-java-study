

#### 정적 유틸리티 클래스

```java
public class SpellChecker {
	private static final Lexicon dictionary = ...;
    
    private SpellChecker() {}
    
    public static boolean isVaild(String word) {...}
    public static List<String> suggestions(String typo) {...}
}

SpellChecker.isValid(word);
```

#### 싱글턴

```java
public class SpellChecker {
	private final Lexicon dictionary = ...;
    
    private SpellChecker() {}
    public static SpellChecker INSTANCE = new SpellChecker(...);
    
    public static boolean isVaild(String word) {...}
    public static List<String> suggestions(String typo) {...}
}

SpellChecker.INSTANCE.isValid(word);
```


위와 같이 구성할 경우, 사전을 단 하나로만 사용하지 못함 -> 개인사전, 한국어사전 등에 대해서 대응하기 어려움


### SpellChecker에서 여러개의 Dictionary를 넣어주는 방법

#### 1. Setter 클래스 추가
```java
public static void setDictionary(Lexicon new) {
    	dictionary = new;
    }
```
이 경우 다른 사전으로 교체가 가능하지만 **멀티스레드 환경에서 매우 위험, 언제 사전이 바뀌었는지 알기에 어려움**

#### 2. 인스턴스 생성시에 필요한 자원을 넘겨주는 방식 - 의존성 주입
```java
public class SpellChecker {
	private final Lexicon dictionary;
    
    private SpellChecker(Lexicon dictionary) {
	    this.dictionary = dictionary
    }
    
    public static boolean isVaild(String word) {...}
    public static List<String> suggestions(String typo) {...}
}

interface Lexicon {}


public class privateDictionary implements Lexicon {}

public class koreanDictionary implements Lexicon {}

Lexicon dic = new myDictionary();
SpellChecker spellChecker = new SpellChecker(dic);
spellChecker.isValid(word);
```



### 생성자에 자원 팩터리 넘겨주기

마인크래프트와 같은 게임을 만든다고 가정하면, GameMap이라는 클래스는 맵을 만드는 작업을 하는 클래스이고, 맵을 만들기 위해서는 Tile 종류의 클래스가 존재합니다.

이를 구현할 때에 자바 8의 ```Supplier<T>``` 함수형 인터페이스를 사용하면, 특정 타입(Tile)의 하위 타입이라면 무엇이든 생성가능한 팩터리를 넘길 수 있습니다. 

```java
class Tile {
    // 기본 타일 클래스
}

class GrassTile extends Tile {
    // 잔디 타일 클래스
}

class WaterTile extends Tile {
    // 물 타일 클래스
}

class MountainTile extends Tile {
    // 산 타일 클래스
}

```


```java
public class GameMap {
    private Supplier<? extends Tile> tileSupplier;

    public GameMap(Supplier<? extends Tile> tileSupplier) {
        this.tileSupplier = tileSupplier;
    }

    public void generateMap() {
    }
}

```

``` java
Supplier<Tile> grassTileSupplier = GrassTile::new; 
GameMap grassMap = new GameMap(grassTileSupplier);
grassMap.generateMap()
```