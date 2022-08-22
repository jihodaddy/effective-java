# 불필요한 객체 생성을 피하라

[백기선님 강의](https://www.youtube.com/watch?v=0yUxPUXS1pM&list=PLfI752FpVCS8e5ACdi5dpwLdlVkn0QgJJ&index=6)

[code1](https://github.com/keesun/study/blob/master/effective-java/item6.md)

[code2](https://github.com/WegraLee/effective-java-3e-source-code/tree/master/src/effectivejava/chapter2/item6)

--- 

## 1. 재사용성에 대한 관점
똑같은 기능의 객체를 매번 생성하기보다는 객체 하나를 재사용하는 편이 좋다

특히 불변 객체는 언제든 재사용할 수 있다

아래와 같이 생성자를 통해 String 객체를 매번 만들지 말아라
```java
String s = new String("bikini"); // 따라 하지 말 것 
String s = "bikini";
```

생성자를 통해 String 객체를 만들게 되면 쓸데없이 String 인스턴스를 반복해서 만들게 된다

Java의 가상머신은 똑같은 문자열 리터럴에 대해서는 동일 코드를 사용하는 재사용성이 보장된다

생성자를 통해 생성하는 것 대신에 정적 팩토리 매소드를 제공하는 불변 클래스에서는 재사용 할 수 있는 객체 생성을 할 수 있다

차이점이라면 생성자는 매번 새로운 객체를 만들지만 팩토리 메소드는 클래스 내부에 한번 만들어서 캐싱해놓고 사용할 수 있다 (미리 만들어 놓은게 있다는 부분)

```java
public class Main {
    /**
     * The {@code Boolean} object corresponding to the primitive
     * value {@code true}.
     */
    public static final Boolean TRUE = Boolean.TRUE;

    /**
     * The {@code Boolean} object corresponding to the primitive
     * value {@code false}.
     */
    public static final Boolean FALSE = Boolean.FALSE;
    
    public static void main(String[] args) {
        Boolean result = Main.valueOf(true);
        System.out.println(result);
    }
    
    public static Boolean valueOf(boolean b) {
        return b ? Main.TRUE : Main.FALSE;
    }
}

```

생성 비용이 비싼 객체가 반복해서 사용하게 된다면 캐싱해서 재사용하는 걸 권장한다

생성 비용이 비싸다라는 것은? (이전 작성자들 내용 참고)
1. 시스템의 자원을 많이 먹는 부분
    1. 메모리
    2. 디스크 사용랑
    3. 네트워크의 대역폭
2. 데이터의 크기가 크거나 객체 내부에 여러 객체들을 포함하는 경우나 단순 생성/소멸이 아닌 연관관계가 복잡한 부분
    1. 크기가 아주 큰 Array
    2. Database Connection
    3. I/O 작업을 필요로 하는 Object
3. (책에 나오는) Expression Object의 Pattern

```java
// 이렇게 하면 더 느리다
static boolean isRomanNumeralSlow(String s) {
    return s.matches("^(?=.)M*(C[MD]|D?C{0,3})"
            + "(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$");
}

// 미리 컴파일 해놓은 객체를 불러 사용하는 방식으로 개선했다 
private static final Pattern ROMAN = Pattern.compile(
    "^(?=.)M*(C[MD]|D?C{0,3})"
        + "(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$");

static boolean isRomanNumeralFast(String s) {
    return ROMAN.matcher(s).matches();
}
```

String.matches가 가장 쉽게 정규 표현식에 매치가 되는지 확인하는 방법이긴 하지만 성능이 중요한 상황에서 반복적으로 사용하기에 적절하지 않다

## 2. 불변 객체를 재사용할 때 위험한 부분

불변 객체인 경우에 안정하게 재사용하는 것이 매우 명확하다 하지만 몇몇 경우에 분명하지 않은 경우가 있다. 어댑터를 예로 들면, 어댑터는 인터페이스를 통해서 뒤에 있는 객체로 연결해주는 객체라 여러개 만들 필요가 없다.

Map 인터페이스가 제공하는 keySet은 Map이 뒤에 있는 Set 인터페이스의 뷰를 제공한다. keySet을 호출할 때마다 새로운 객체가 나올거 같지만 사실 같은 객체를 리턴하기 때문에 리턴 받은 Set 타입의 객체를 변경하면, 결국에 그 뒤에 있는 Map 객체를 변경하게 된다.

```java
public class UsingKeySet {
    public static void main(String[] args) {
        Map<String, Integer> menu = new HashMap<>();
        menu.put("Burger", 8);
        menu.put("Pizza", 9);

        Set<String> names1 = menu.keySet();
        Set<String> names2 = menu.keySet();

        // 재사용하는 전역에서 사용하는 Map일 경우 다른 쪽에도 영향을 줄 수 있다
        names1.remove("Burger");
        System.out.println(names2.size()); // 1
        System.out.println(menu.size()); // 1
    }
}
```

## 3. 아무생각없이 사용하면 문제되는 부분인 오토박싱/언박싱

불필요한 객체를 생성하는 또 다른 방법으로 오토박싱이 있다. 오토박싱은 프로그래머가 프리미티브 타입과 박스 타입을 섞어 쓸 수 있게 해주고 박싱과 언박싱을 자동으로 해준다.

오토박싱은 프리미티브 타입과 박스 타입의 경계가 안보이게 해주지만 그렇다고 그 경계를 없애주진 않는다.

```java
public class AutoBoxingExample {
    public static void main(String[] args) {
        long start = System.currentTimeMillis();
        Long sum = 0l;
        for (long i = 0 ; i <= Integer.MAX_VALUE ; i++) {
            sum += i;
        }
        System.out.println(sum);
        System.out.println(System.currentTimeMillis() - start);
    }
}
```
위 코드에서 sum 변수의 타입을 Long으로 만들었기 때문에 불필요한 Long 객체를 2의 31 제곱개 만큼 만들게 되고 대략 6초 조금 넘게 걸린다. 타입을 프리미티브 타입으로 바꾸면 600 밀리초로 약 10배 이상의 차이가 난다.

불필요한 오토박싱을 피하려면 박스 타입 보다는 프리미티브 타입을 사용해야 한다.

## 4. 프로그래머의 통찰력이 필요한 부분

객체생성은 비싸니 피해야 한다로 오해하면 안된다. 특히 JVM에서는 별다른 일을 하지 않는 작은 객체에 대해서는 큰 부담이 되지 않는다고 한다

프로그램의 명확성, 간결성, 기능을 위해서는 객체를 추가로 만들 수도 있어야 한다

또 객체 생성을 효율적으로 해보겠다고 사소한 것들도 다 캐싱하거나 자체 풀(pool)을 만들어서 유지보수하기 어려운 복잡한 프로그램을 만드는 것도 피해야 하는 부분이다 (JVM에게 위임할 부분은 위임해라 가비지 컬렉터를 신뢰하자)

[Item50]방어적 복사(defensive copy) 와 대비되는 내용이기 때문에 객체 생성을 해야하는 경우와 하지 않고 기존 것을 재사용해야 하는 부분은 개발자의 역량에 달려있다 (혹은 성능 테스트를 직접 해서 비교해보아라)

새로 만든 생성자가 필요한 경우와 재사용 가능한 불변 객체를 사용하는 구분을 할 수 있어야 한다