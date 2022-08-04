ITEM1
# 생성자 대신 정적 팩터리 메서드를 고려하라

### 장점
1. 이름을 가질 수 있다.
2. 호출 때마다 인스턴스를 새로 생성하지 않아도 된다.
3. 반환 타입의 하위 타입 객체를 반환할 수 있는 능력이 있다.
4. 입력 매개변수에 따라 매번 다른 클래스의 객체를 반환할 수 있다.
5. 정적팩터리 메서드를 작성하는 시점에는 반환할 객체의 클래스가 존재하지 않아도 된다.

### 단점
1. 상속을 하려면 pubilc이나 protected 생성자가 필요하니 정적 팩터리 메서드만 제공하면 하위 클래스를 만들 수 없다.
2. 정적 팩터리 메서드는 프로그래머가 찾기 어렵다

<hr>

### 1. 이름을 가질 수 있다.
생성자에 넘기는 매개변수와 생성자 자체로 반환될 객체의 특성을 잘 설명하지 못하는 경우 정적 팩터리 메서드를 사용하면 반환될 **객체의 특성을 명확히 나타낼 수 있다.**

```java
public class myServcie{

	private String name;
	private boolean open;
	private boolean mine;

	public myService(){}

	public static isOpenedMyService(String name){
			myService ms = new myService();
			ms.name = name;
			ms.open = true;
			ms.mine = false;
	}

	public static isMineMyService(String name){
		  myService ms = new myService();
		  ms.name=  name;
		  ms.open = false;
		ms.mine = true;
	    }
}
```
- 어떤 객체를 만들 것인지 메소드의 이름에  명시해 사용하기 편리하게 유도하는 것이 가능
- 메소드를 사용해 해당 객체를 만드는데 있어서 편하게 인지 하고 사용이 가능

#### 하나의 시그니쳐로 여러가지 객체를 생성할 수 있다.
매개변수의 타입과 갯수가 같은 생성자를 여러개 만들 수 없다.

```java
public class Book {
    private String name;
    private String author;
    private String publisher;

    public Book(String name) {
        this.name = name;
    }
    
    // 불가능
    public Book(String author) {
        this.author = author;
    }
}
```

하지만 아래와 같이 정적 팩토리 매서드에서는 가능하다.

```java
public class Book {
    private String name;
    private String author;
    private String publisher;

    public static Book createBookWithName(String name) {
        Book book = new Book();
        book.name = name;
        return book;
    }

    public static Book createBookWithAuthor(String author) {
        Book book = new Book();
        book.author = author;
        return book;
    }
}
```

매개변수의 타입과 갯수가 같은 생성자를 여러개 만들 수 없다.

### 2. 호출 할 때마다 인스턴스를 새로 생성하지 않아도 된다.
static 으로 사용되기 때문에 따로 객체를 생성해서 사용할 필요없이 선언해둔 것을 통해서 사용하는 것이 가능
-   인스턴스를 미리 만들어 놓거나 새로 생성한 인스턴스를 캐싱하여 재활용하는 식으로 불필요한 객체 생성을 피할 수 있다.
-   (특히 생성비용이 큰)같은 객체가 자주 요청되는 상황이라면 성능을 상당히 끌어올려 준다.

```java
public final class Boolean implements java.io.Serializable,Comparable<Boolean> {

    public static final Boolean TRUE = new Boolean(true);

    public static final Boolean FALSE = new Boolean(false);

    public static Boolean valueOf(boolean b) {
        return (b ? TRUE : FALSE);
    }
}
```
-   (매번 생성하지 않기 때문에 인스턴스 통제가 가능라고 표현한다) 인스턴스 통제가 가능하다 -> 싱글톤으로 만들 수 있다.
-   불변 값 클래스에서 동치인 인스턴스가 단 하나뿐임을 보장할 수 있다(a == b일 때만, a.equals(b)가 성립)

### 3. 반환 타입의 하위 타입 객체를 반환할 수 있는 능력이 있다.

정적 팩터리 메서드의 리턴 타입을 인터페이스로 할 경우 하위 타입 객체를 반환 할 수 있다.  
이는 주로 동반 클래스에서 볼 수 있다.

> 동반 클래스 : 자바 1.8 이전에는 인터페이스에 정적 메서드를 선언할 수 없었다. 따라서 인터페이스에 기능을 추가하기 위해서는 `동반 클래스`라는 것을 만들어 그 안에 정적 메서드를 추가했다.

```
인터페이스 Collection의 동반 클래스 -> Collections
```

다음은 Collections 클래스의 정적 팩토리 메소드 unmodifiableList의 모습이다.

```java
public static <T> List<T> unmodifiableList(List<? extends T> list) {
        return (list instanceof RandomAccess ?
                new UnmodifiableRandomAccessList<>(list) :
                new UnmodifiableList<>(list));
}
```

-   이와 같이 팩터리 메서드를 통해 `메소드의 반환 타입은 List이지만 실제로는 List의 하위 객체를 반환`시킬 수 있다.
-   이럴 경우의 장점은, 사용자들이 해당 인터페이스의 구현체를 일일이 알아 볼 필요가 없다는 것이다.

---

### 4. 입력 매개변수에 따라 매번 다른 클래스의 객체를 반환할 수 있다.

다음은 EnumSet의 정적 팩터리 메서드이다.  
메서드 내부에서 `universe.length 따라 리턴 타입을 다르게` 반환 한다.
```java
public static <E extends Enum<E>> EnumSet<E> noneOf(Class<E> elementType) {
        Enum<?>[] universe = getUniverse(elementType);
        if (universe == null)
            throw new ClassCastException(elementType + " not an enum");

        if (universe.length <= 64)
            return new RegularEnumSet<>(elementType, universe);
        else
            return new JumboEnumSet<>(elementType, universe);
    }
```

-   이와 같이 팩터리 메서드를 통해 `사용자가 내부의 구현에 대해 알 필요 없이` 원하는 반환 값을 전달 받을 수 있다.
-   또한 메서드의 내부 구현이 변경되어도, 반환타입만 같다면 사용자에게 영향을 끼치지 않는다.

## 5. 정적 팩터리 메소드를 작성하는 시점에는 반환할 객체의 클래스가 존재하지 않아도 된다

책에서는 예시를 JDBC를 들었다 -> 실제 JDBC를 연동하는 코드를 구현해 놓았는데, 만약 다른 JDBC가 들어오면 코드를 바꿔야하나?  
만약 정적 팩토리 메소드를 사용해서 구현을 해 두었으면 **내부코드에 의존적이지 않다는 것**이 장점이다


이외에도 DTO <-> Entity을 진행하는데 있어서 아주 유용하다고 한다 -> 이건 나도 생각했던건데 실제로 그렇다고 하니까 한번 적용해보자  
//static으로 빼긴 뺐는데... 생성자도 한꺼번에 처리해버리고 싶지만 dto같은 경우에는 generator를 사용하기 때문에 이거쓰면 엔티티 전용으로밖애 못만들구나

### 흔히 사용하는 명명방식들

-   from : 매개변수를 하나 받아서 해당 타입의 인스턴스를 반환
    -   Date d = Date.from(instant);
-   of : 여러 매개변수를 받아 적합한 타입의 인스턴스를 반환
    -   Set faceCards = EnumSet.of(JACK, QUEEN, KING);
-   valueOf : from과 of의 더 자세한 버전
    -   BigInteger prime = BigInteger.valueOf(Integer.MAX_VALUE);
-   instance || getInstance : (매개변수를 받는다면) 매개변수로 명시한 인스턴스를 반환하지만, 같은 인스턴스임을 보장하지 않는다.
    -   StackWalker luke = StackWalker.getInstance(options);
-   create || newInstance : intacne 혹은 getInstance와 같지만, 매번 새로운 인스턴스를 생성해 반환한다.
    -   Object newArray = Array.newInstance(classObject, arrayLen);
-   getType : getInstance와 같으나, 생성할 클래스가 아닌 다른 클래스에 팩토리 메소드를 정의할 때 쓴다.
    -   FileStore fs = Files.getFileStore(path);
-   newType : newInstance와 같으나, 생성할 클래스가 아닌 다른 클래스에 팩토리 메소드를 정의할 때 쓴다.
    -   BufferedReader br = Files.newBufferedReader(path);
-   type : getType과 newType의 간결한 버전
    -   List litany = Collections.list(legacyLitany);





## 단점

상속을 하려면 public, protected를 사용해야하는데, 생성자가 없고 정적 팩토리 메소드만 존재하면 상속이 불가능


정적 팩토리 메소드만 사용하게 되면 → 자바독스와 같은 것으로 분석하는 경우에는 찾기가 힘들어진다


그래서 제목에도 잘 나와있다 -> 정적 팩터리 메소드를 사용하는 것을 **고려하라**



## Enum

이늄이라고도 부르고, 이건 상수 목록을 담을 수 있는 데이터 타입이다  
이늄을 사용하게 되면 값을 지정해줄 때 선언해둔 값들로만 제한해서 사용하게 할 수 있다, 즉 Type-Safety를 보장해줄 수 있다  
이것을 활용해서 싱글턴 패턴 구현에도 자주 사용하곤 한다


특정 enum 타입이 가질 수 있는 모든 값을 순회하며 출력하는 방법 -> Enum.values();  
enum도 자바 클래스처럼 생성자, 메소드, 필드를 가질 수 있는가  
enum을 비교하는데 있어서 == 을 사용해서 동일성을 비교하는 것이 가능한가  
-> equals, == 모두 사용이 가능한데, equals이라는 메소드는 null-safe한 메소드가 아니기 때문에 null 체크를 거치고 확인해줘야하는데 == 을 사용하게되면 null에 대한 고민을 하지 않아도 되기 때문에 == 을 기본적으로 권장

enum을 키로 사용하는 map을 정의하거나 enum을 가지고 있는 set을 구현해보라 -> EnumMap, EnumSet을 사용



## [](https://github.com/kyu9/My-Library/blob/cf41554aa16775334cc30e9661dd43c87aa30b92/Effective_Java/EJ_item1.md#%ED%94%8C%EB%9D%BC%EC%9D%B4%EC%9B%A8%EC%9D%B4%ED%8A%B8-%ED%8C%A8%ED%84%B4)플라이웨이트 패턴

객체를 재사용하는 방법 중에 하나임

-   객체를 가볍게 만들어 메모리 사용을 줄이는 패턴
-   자주 변하는 속성 또는 외적인 속성과 변하지 않는 속성을 분리해서 재사용해서 메모리 사용을 줄이는 것이 가능




변경되지 않는 것을 플라이 웨이트 팩토리에 모아두고 가져와서 사용한다



## [](https://github.com/kyu9/My-Library/blob/cf41554aa16775334cc30e9661dd43c87aa30b92/Effective_Java/EJ_item1.md#%EC%9D%B8%ED%84%B0%ED%8E%98%EC%9D%B4%EC%8A%A4%EC%99%80-%EC%A0%95%EC%A0%81-%EB%A9%94%EC%86%8C%EB%93%9C)인터페이스와 정적 메소드

자바 8, 9 이후부터는 인터페이스에 직접 메소드를 정의하는 것이 가능해졌다  
default 키워드나 static 키워드가 들어가 있어야 메소드를 정의할 수 있다  
default 메소드는 인터페이스에서 메소드의 선언 뿐만 아니라, 기본적인 구현체까지 제공하는 것이 가능하다 또한 기존의 인터페이스를 구현하는 클래스에 새로운 기능을 추가하는 것이 가능하다  
static 메소드는 default에서는 불가능 했던(기본으로 public이기 때문에) private 메소드를 사용할 수 있다는 특징이 있다



## [](https://github.com/kyu9/My-Library/blob/cf41554aa16775334cc30e9661dd43c87aa30b92/Effective_Java/EJ_item1.md#%EC%84%9C%EB%B9%84%EC%8A%A4-%EC%A0%9C%EA%B3%B5%EC%9E%90-%ED%94%84%EB%A0%88%EC%9E%84%EC%9B%8C%ED%81%AC)서비스 제공자 프레임워크

서비스 제공자 인터페이스(Service Provider Interface), 서비스 제공자, 서비스 제공자 등록 API, 서비스 접근 API  
서비스 제공자 인터페이스란 그냥 원하는 서비스를 정의할 인터페이스를 지칭  
구현체는 어디에 구현되어있던 상관은 없다 -> 중요한 건 구현체에 의존적이지 않게 되기 위해서이다  
스프링부트에서도 빈이라는 개념을 통해서 서비스를 제공해주고, DI을 통해서 원하는 서비스를 가져와서 사용하는 것이 가능하다  
빈등록을 통해서 서비스를 등록하고, getBean이나 @Autowired을 통해서 서비스를 가져와서 사용하는 것이 가능

자바를 기준으로 확인하는 방법으로는 ServiceLoader가 예시로 있다


브릿지 패턴의 예시를 잘 들어주셨는데, 롤로 따지면 스킨과 챔피언이 나누어져 있는 상황에서 만약 스킨이 추가되게 되면 모든 챔피언에도 적용이 되어야 하지만  
챔피언과 스킨 사이에 브릿지, delegate클래스를 두고 해당에서만 작업해주면 원하는 스킨으로 바꿔끼면서 챔피언을 사용하는 것이 가능해진다  
이 브릿지 패턴이 스프링의 PSA(Portable Service Abstraction)에도 적용되어있다는 점


## [](https://github.com/kyu9/My-Library/blob/cf41554aa16775334cc30e9661dd43c87aa30b92/Effective_Java/EJ_item1.md#%EB%A6%AC%ED%94%8C%EB%A0%89%EC%85%98)리플렉션

클래스로더를 통해서 읽어온 클래스 정보를 사용하는 기술이다  
옛날에 했던 것을 리마인드 해보면, 리플렉션을 통해서 객체를 생성하는 것이 가능했었으며, 이외에도 메소드나 필드의 이름을 가져오거나 메소드에 선언되어 있는 애노테이션등을 가져오는 것이 가능했다

[참고 깃허브](https://github.com/kyu9/My-Library/blob/cf41554aa16775334cc30e9661dd43c87aa30b92/Effective_Java/EJ_item1.md)

[참고깃허브2](https://github.com/Meet-Coder-Study/book-effective-java/blob/main/2%EC%9E%A5/1_%EC%83%9D%EC%84%B1%EC%9E%90_%EB%8C%80%EC%8B%A0_%EC%A0%95%EC%A0%81_%ED%8C%A9%ED%84%B0%EB%A6%AC_%EB%A9%94%EC%84%9C%EB%93%9C%EB%A5%BC_%EA%B3%A0%EB%A0%A4%ED%95%98%EB%9D%BC_%EA%B9%80%EC%9E%AC%EC%A4%80.md)