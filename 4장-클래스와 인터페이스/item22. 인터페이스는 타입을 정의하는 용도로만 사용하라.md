# Item22. 인터페이스는 타입을 정의하는 용도로만 사용하라.

- 인터페이스는 자신을 구현한 클래스의 인스턴스를 참조할 수 있는 타입 역할을 한다.
- 클래스가 어떤 인터페이스를 구현한다는 것은 자신의 인스턴스로 무엇을 할 수 있는지 클라이언트에게 알려주는 것
- 인터페이스는 오직 이 용도로만 사용해야 한다.
```java
public interface Walkable {
    void walk();
}

public class Dog implements Walkable {

    @Override
    public void walk() {
        ...
    }
}
```

## 지침에 맞지 않는 상수 인터페이스(안티패턴)
아래 코드는 인터페이스를 잘못 사용한 예
```java
public interface PhysicalConstants {

    static final double AVOGADROS_NUMBER   = 6.022_140_857e23;
    
    static final double BOLTZMANN_CONSTANT = 1.380_648_52e-23;
    
    static final double ELECTRON_MASS      = 9.109_383_56e-31;

}
// 숫자 리터럴에 사용한 밑줄(_) => 자바 7부터 허용되는 이 밑줄은 숫자 리터럴의 값에는 아무런 영향을 주지 않고 가독성이 좋아진다.
// 5자리 이상의 수라면 밑줄을 사용하는 걸 고려해보자.
```
- 사용하지 않을 수도 있는 상수를 포함하여 모두 가져오기 때문에 계속 가지고 있어야 함
- 다음 릴리즈에서 위 상수들을 안쓰더라도 바이너리 호환성을 위해 여전히 구현하고 있어야 함
- final이 아닌 클래스가 상수 인터페이스를 구현한다면 모든 하위 클래스의 이름공간이 그 인터페이스가 정의한 상수들로 오염되어 버림
- 상수 인터페이스를 구현하는 것은 이 내부 구현을 클래스의 API로 노출하는 행위
- 클래스가 어떠한 상수 인터페이스를 사용하든 이는 클라이언트에게 중요한 정보가 아니며, 오히려 혼란을 주기도 한다.
```java
public interface Constant {

    double PI = 3.1415;

}

public class ConstantImpl implements Constant{
    public static void main(String[] args) {
        double area = getArea(2);
        System.out.println("area = " + area);
    }

    private static double getArea(double r) {
        return PI * r * r;
    }
}
```

### 해결 방법
- 특정 클래스나 인터페이스와 강하게 연관된 상수라면, 그 클래스나 인터페이스 자체에 추가해야 한다.
- 대표적인 예로 Integer에 선언된 MIN_VALUE, MAX_VALUE 상수가 있다
- 열거 타입으로 나타내기 적합한 상수라면 열거 타입으로 만들어 공개하면 된다.
- 그것도 아니라면, 인스턴스화 할 수 없는 유틸리티 클래스에 담아 공개하자.
  - Item 4 인스턴스화를 막으려거든 private 생성자를 사용하라
```java
import static effectivejava.chapter4.item22.constantutilityclass.PhysicalConstants.*;

public class Test {
    double atoms(oduble mols) {
        return AVOGADROS_NUMBER * mols;
    }
    ...
    // PhysicalConstants를 빈번히 사용한다면 정적 임포트가 값어치를 한다.
}   
```


### 핵심 정리
인터페이스는 타입을 정의하는 용도로만 사용해야 함
상수 공개용 수단으로 사용하지 말자.