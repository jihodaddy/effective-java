# Item17. 변경 가능성을 최소화하라.

## 불변 클래스
인스턴스 내부 값을 수정할 수 없는 클래스. 
String, 기본 타입의 박싱된 클래스들, BigInteger, BigDecimal   
가변클래스보다 설계하고 구현하고 사용하기 쉬움, 오류 생길 여지도 적고 안전

### 불변 클래스 만드는 규칙
- 객체의 상태를 변경하는 메서드(변경자)를 제공하지 않음.
  - setter
- 클래스를 확장할 수 없도록 함.
  - final
- 모든 필드를 final로 선언.
  - 여러 스레드에서 접근해도 값이 안변함
- 모든 필드를 private로 선언.
- 자신외에 내부의 가변 컴포넌트에 접근할 수 없도록 함.

```java
public final class Complex{
    private final double re;
    private final double im;
    
    public Complex(double re, double im){
        this.re = re;
        this.im = im;
    }
    
    public double realPart(){ return re;}
    public double imagenaryPart(){ return im;}
    
    public Complex plus(Complex c){
        return new Complex(re + c.re, im + c.im);
    }
    
    public Complex minus(Complex c){
        return new Complex(re - c.re, im - c.im);
    }
    
    public  Complex times(Complex c){
        return new Complex(re * c.re, im * c.im);
    }
    
    public Complex divideBy(Complex c){
        double tmp = c.re * c.re + c.im * c.im;
        return new Complex(c.re * c.re + c.im * c.im / tmp ,
                c.re * c.re - c.im * c.im / tmp);
    }
    
    @Override
    public boolean equals(Object o){
        if (0 == this)
            return true;
        if (!(o instanceof Complex))
            return false;
        Complex c = (Complex)o;
        
        return Double.compare(c.re, re) == 0
                && Double.compare(c.im, c.im) == 0;
    }

    @Override
    public int hashCode(){
        return 31 * Double.hashCode(re) + Double.hashCode(im);
    }
    
    @Override
    public String toString(){
        return "(" + re + "+" + im + "i)";
    }
}

``` 

## 불변객체의 장점



1. 불변 객체는 근본적으로 스레드 안전하여 따로 동기화할 필요 없다.

   -> 객체의 변동이 없으니 여러 쓰레드에서 접근해도 절대 훼손되지 않는다.



2. 한번 만든 인스턴스는 최대한 재활용 하자.

   -> 정적 팩터리 (아이템1) 를 제공할 수 있다. (Ex: Biginteger.class)



3. 방어적 복사가 필요없다

   -> 불변 객체를 복사해도 항상 원본이랑 같기 때문에 굳이 clone 메서드나 복사생성자를 제공하지 말자.(불필요한 객체생성 방지)



4. 불변객체는 자유롭게 공유할 수있음은 물론, 불변 객체끼리는 내부 데이터를 공유할 수 있다.



5. 객체를 만들 때 다른 불변 객체들을 구성요소로 사용하면 이점이 많다.

   -> 유일 값이기 때문에 예를들어 (Set)의 원소로 쓰기 안성 맞춤이다. 즉 불변식을 유지할수 있다.



6. 불변 객체는 그 자체로 실패 원자성을 제공한다.(아이템76)

   -> 상태가 절대 변하지 않으니 잠깐이라도 불일치 상태에 빠질 가능성이 없다.





## 불변객체의 단점



1. 값이 다르면 반드시 독립된 객체로 만들어야 한다.

   -> 값의 가짓수가 많다면 이들을 많이 만드는데 많은 비용이 든다.



예시

   ```java
   public static void main(String[] args) {
       String count = "";
       for (int i = 0; i <10; i++) {
           count +=String.valueOf(i);
           System.out.println(count);
       }
   }
   ```

다음과 같은경우 String 의 인스턴스를 계속해서 만든다.

따라서 다음과 같은 경우 StringBuilder 가변 동반 클래스를 제공하자.





## 불변클래스를 만드는 설계방법



1. final 클래스로 선언
2. 모든 생성자를 private 혹은 package-private 으로 만들고 정적팩터리(아이템1) 을 제공하자 이 방법이 조금더 유연한 방법이다.



##    결론



1. getter 메서드가 있다고 해서 무조건 setter 를 만들지는 말자.

   -> 클래스는 꼭필요한 경우가 아니라면 불변이어야 한다.



2.  불변으로 만들 수없는 클래스라도 변경할 수 있는 부분을 최소한으로 줄이자.

-> 꼭 변경해야 할 필드를 뺀 나머지는 final로 선언하자.

-> 아이템 15의 조언과 종합하자면 **다른 합당한 이유가 없다면 모든 필드는 private final 이어야한다.**



3. 생성자는 불변식 설정이 모두 완료된 ,초기화가 완벽히 끝난 상태의 객체를 생성해야한다.

   ->확실한 이유가 없다면 생성자와 정적 팩터리 외에는 그 어떤 초기화 메서드도 public 으로 제공 하면 안된다.




