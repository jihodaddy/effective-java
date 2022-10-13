# Item21. 인터페이스는 구현하는 쪽을 생각해 설계하라.


현재 자바 8에서 Collection Interface에 추가된 removeIf 메서드를 찾아보자.

RemoveIf는 predicate가 true를 반환하는 모든 원소를 제거한다.

### RemoveIf
```java
default boolean removeIf(Predicate<? super E> filter) {  
    Objects.requireNonNull(filter);  
    boolean removed = false;  
    final Iterator<E> each = iterator();  
    while (each.hasNext()) {  
        if (filter.test(each.next())) {  
            each.remove();  
            removed = true;  
        }  
    }  
    return removed;  
}
```
### RemoveIf 사용 예제
```java
    List<String> list = new ArrayList<>();  
    list.add("1");  
    list.add("2");  
    list.add("1");  
    list.add("2");  
    list.add("3");  
      
    System.out.println("list.size() = " + list.size());   // 5
      
    list.removeIf(number -> number.equals("1"));  
      
    System.out.println("list.size() = " + list.size()); // 3
```

Collection의 RemoveIf는 현재 범용적으로 사용을 하고 있지만, 아직까지 모든 Collection 구현체와 잘 어울리는 게 아니라고 한다.

책에서는 SynchronizedCollection을 예를 들었고, 해당 SynchroizedCollection을 사용했을 때 RemoveIf를 사용하게 되면 **ConcurrentModificationException**을 발생시키거나, 다른 결과를 보내준다고 한다.



## 그러면 이렇게 Collection의 RemoveIf처럼  default Method로 사용이 되고 있고,  해당 Interface를 사용하고 있는 구현체 중에 default Method에 대해서 사용 유무에 대한 처리를 어떻게 했을까?

바로 구현한 인터페이스의 디폴트 메서드를 재정의하고, 다른 메서드에서는 디폴트 메서드를 호출하기 전에 필요한 작업을 수행하도록 했다.

하지만, 해당 플랫폼에 속하지 않는 것들은, 수정될 기회가 없고, 그중 일부는 여전히 수정되고 있지 않다고 한다.



## 그러면 Default Method를 잘 사용하려면 어떻게 해야하는데요?

1. 인터페이스를 설계할 때는 여전히 세심한 주의를 기울여야 한다.
    - Default Method로 기존 interface에 새로운 method를 추가하면 커다란 위험도 딸려온다.
2. 인터페이스를 릴리스한 후라도 결함을 수정하는 게 가능한 경우도 있겠지만, 절대 그 가능성에 기대서는 안 된다.
    - 새로운 인터페이스라면 릴리스 전에 반드시 테스트를 거쳐야하며, 다른 방식으로 최소한 3가지로 구현을 해봐야 한다.
    - 각 인터페이스의 인스턴스를 다양한 작업에 활용하는 클라이언트도 여러 개 만들어봐야 한다.
