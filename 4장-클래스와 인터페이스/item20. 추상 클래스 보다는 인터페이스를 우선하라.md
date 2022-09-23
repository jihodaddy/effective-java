# Item 20. 추상 클래스 보다는 인터페이스를 우선하라

## 정리

- 다중 구현 타입은 인터페이스가 더 적합하다 왜냐하면 다중 상속이 가능하기 때문이다
- 골격 구현이라 하면 클래스의 형태에서는 일단 기반 매소드를 인터페이스에 넣고 이는 공통 기능으로 정의한다
- 기반 매소드의 형태를 추상 클래스에 추상 매소드로서 활용하고 추상 관점에서 기반 매소드에 포함되지 못한 기능은 골격 구현 클래스의 고유 기능 매소드로 추가해서 넣는다
- 즉, 인터페이스의 메소드가 우선하며 골격에 추가로 필요한 매소드가 추상 클래스의 매소드가 된다 이는 public 이 아닌 private가 될 수도 있다

## 자바의 다중 구현 매커니즘을 지원하는 2개의 요소

>- Interface
>- Abstract Class (Base Class)

언어가 발전하면서 Interface에 default 키워드로 구현된 매소드를 추가할 수 있으나 이를 사용하려면 주석이나 명세에 필수적으로 기록하고 모든 기능이 동일하게 동작할 수 있어야 한다

## 추상 클래스의 단점

다중 상속이 되지 않는다 즉 이것 떄문에 확장에 용이하지 않다 (상위/하위로서만 구조적으로 존재)

## 인터페이스가 확장에 용이한 이유

1. 기존 레거시 클래스에도 손쉽게 확장 인터페이스를 구현해 넣을 수 있다
- java 버전이 올라가면서 추가된 기능이 있는데 이런 표준 라이브러리들의 기능을 끼워 넣으려면 추상 클래스로는 되지 않는다 즉, 자바의 표준 라이브러리의 확장은 인터페이스로 되어 있다

2. 인터페이스는 Mixin 정의가 가능하다
- 믹스인이 구현된 클래스는 원래 "주" 타입 형태가 있으나 상황에 따라 선택적인 기능을 추가로 가질 수 있다 이는 다중 인터페이스 상속이 가능하기 떄문에 믹스인이라는 선택적 기능을 사용할 수 있다는 것이다
- 추상 클래스는 계층 구조가 두 부모를 섬길 수 없고 상속에 상속을 이용하면 의도치 않은 중간 추상 클래스로 인해 기존 기능에 영향을 끼칠 수 있다

3. 인터페이스는 계층구조가 없는 타입 프레임워크를 만들 수 있다
- 이 얘기는 유연한 확징으로 현실 세계에서 하나로 정의되지 않는 타입에 대해 상/하의 상속구조를 조합구조로 가져갈 수 있다는 얘기이다 (책에서는 가수(Singer)와 작곡가(SongWiter)를 들었다 이는 클래스로 확정하면 동시에 여러가지를 포함하기 애매)
- 계층구조로 많은 기능이 추상화 되면 슈퍼 클래스 즉 거대한 클래스가 되어 추상화 관점을 쪼개어 생각하기 어렵다
- 계층 보다는 Mixin(믹스인)을 고려하는 부분과도 겹치는 내용으로 보인다

## default 매소드를 Interface에서 사용하려면 잘 사용하라

- 개념적으로 하나로 확정하기 어려운 기능은 default 매소드를 제공하지 말라는 얘기이다
- 공통으로 사용되는 기능이 하나의 구현으로 책임질 상황에서만 사용하고 이를 주석이나 명세로 다른 사람에게 전달 될 수 있어야 한다는 주의점을 상기시키고 있다

## 추상골격 구현과 인터페이스를 같이 고려하라

추상 클래스와 인터페이스를 둘 다 사용하라는 관점이다
- 디자인 패턴으로는 템플릿 매소드 패턴이다

참고한 코드 - 신선영님 코드
- 구현한 Phone 클래스에서 process() 를 호출했을 때 만든 Phone 에 따라 결과가 달라진다

```java
// 인터페이스
public interface Phone {
  void booting();
  void greeting();
  void shutdown();
  void process();
}
```

```java
// 추상골격 구현 클래스 (보통 Abstract~의 네이밍을 사용한다)
public abstract class AbstractPhone implements Phone {

	// 같은 동작을 하는 메소드를 여기에 정의한다.
  @Override
  public void booting() {
    System.out.println("booting ...");
  }

  @Override
  public void shutdown() {
    System.out.println("shut down ...");
  }

  @Override
  public void process() {
    booting();
    greeting();
    shutdown();
  }
}
```

```java
public class IPhone extends AbstractPhone implements Phone {

  @Override
  public void greeting() {
    System.out.println("I am iPhone");
  }
}
```

```java

public class GalaxyPhone extends AbstractPhone implements Phone {
  @Override
  public void greeting() {
    System.out.println("I am galaxy phone");
  }
}
```

이런 골격 구현에는 주석을 두어 동작방식에 대해 문서로 남겨야 한다

## 시뮬레이트한 다중 상속

상황에 따라 추상 골격 클래스를 직접 상속받지 못한다면 (상속은 하나만 가능하다)

이때 고려되는 방법으로 골격 클래스를 private한 내부 필드로 두고 인터페이스의 기반 메소드가 호출되면 내부 클래스의 매소드가 호출되도록 랩핑하는 것이다

참고한 코드 - 신선영님 코드
- InnerAbstractPhone 를 내부에 두어 랩핑되었다

```java
public class PhoneManufacturer {
  public void printManuFacturer() {
    System.out.println("Made by Apple");
  }
}
```

```java
// 골격 구현을 확장한 클래스
public class InnerAbstractPhone extends AbstractPhone {

  @Override
  public void greeting() {
    System.out.println("I am iPhone");
  }
}
```

```java
public class IPhone extends PhoneManufacturer implements Phone {
  InnerAbstractPhone innerAbstractPhone = new InnerAbstractPhone(); // 내부 클래스로 정의

  @Override
  public void booting() {
    innerAbstractPhone.booting();
  }

  @Override
  public void greeting() {
    innerAbstractPhone.greeting();
  }

  @Override
  public void shutdown() {
    innerAbstractPhone.shutdown();
  }

  @Override
  public void process() {
    printManuFacturer();
    innerAbstractPhone.process();
  }
}
```

## 결론

처음 결론에 추상 골격작성에 대해 언급했었는데 결국 인터페이스 기반의 확장을 사용하되 골격 클래스는 추상 클래스로 구현하는 방식을 권장하는 것으로 보인다

- 기반 매소드는 인터페이스로 정의
- 상황에 따라 default 매소드를 정의 (선택적)
- 추상 클래스에서 인터페이스의 메소드를 사용할 때는 abstract 매소드로 정의
- 기반 메소드는 아니지만 필요한 매소드는 추상 클래스의 매소드로 정의

추상 골격을 고려해서 확장이 필요하면 Mixin Interface를 사용하자로 마무리한다
