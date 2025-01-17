# 표준 함수형 인터페이스를 사용하라

## **핵심 정리**

자바가 람다를 지원하면서 API 작성하는 모범 사례도 크게 바뀌었다. 예컨대 상위 클래스의 기본 메서드를 재정의해 원하는 동작을 구현하는 템플릿 메서드 패턴의 매력이 크게 줄었다.

이를 대체하는 현대적인 해법은 같은 효과의 함수 객체를 받는 정적 팩터리나 생성자를 제공하는 것이다.

이 내용을 일반화해서 말하면 함수 객체를 매개변수로 받는 생성자와 메서드를 더 많이 만들어야 한다.

이때 함수형 매개변수 타입을 올바르게 선택해야 한다.

LinkedHashMap 생각해보자. 이 클래스의 protected 메서드인 removeEldestEntry 재정의하면 캐시로 사용할 수 있다.

맵에 새로운 키를 추가하는 put 메서드를 호출하여 true가 반환되면 맵에서 가장 오래된 원소를 제거한다. 예컨대 remoteEldestEntry 다음처럼 재정의하면 맵에 원소가 100개가 될 때까지 커지다가 , 그 이상되면 새로운키가 더해질 때마다 가장 오래된 원소를 하나씩 제거한다. 

```java
protected boolean removeEldestEntry(Mpa.Entry<K, V> eldest) {
    return size() > 100;
}
```

잘 동작하지만 람다를 사용하면 훨씬 잘 해낼 수 있음.

LinkedHashMap 오늘날 다시 구현한다면 함수 객체를 받는 정적 팩터리나 생성자로 제공할 것이다.

removeEldestEntry 선언을 보면 이 함수 객체는 Map.Entry<K, V> 받아 boolean 반환해야 할 것 같지만, 꼭 그렇지는 않다. removeEldestEntry size()를 호출해 맵 안의 원소 수를 알아내는데, 

removeEldestEntry가 인스턴스 메서드라 가능한 방식이다.

하지만 생성자에 넘기는 함수 객체는 이 맵의 인스턴스 메소드가 아니다.

팩터리나 생성자를 호출할 때는 맵의 인스턴스가 존재하지 않기 때문이다. 따라서 자기 자신도 함수 객체에 건네줘야 한다.

이를 반영한 함수형 인터페이스는 다음처럼 선언할 수 있다.

**불필요한 함수형 인터페이스 - 대신 표준 함수형 인터페이스를 사용하라**
```java
@FunctionalInterface
interface EldestEntryRemovalFunction<K, V> {
    boolean remove(Map<K, V> map, Map.Entry<K, V> eldest);
}
```

**java.util.function 필요한 용도에 맞는 게 있다면, 직접 구현하지 말고 표준 함수형 인터페이스를 활용하라.**

예컨대 Predicate 인터페이스는 프레디키트(predicate)들을 조합 메서드 제공

LinkedHashMap 예에서는 직접 만든 EldestEntryRemovalFunction 대신 표준 인터페이스인 BiPredicate<Map<K, V>>, Map.Entry<K, V> 사용할 수 있다.

java.util.function 패키지에는 총 43개의 인터페이스가 담겨 있다.

전부 기억하긴 어렵겠지만, 기본 인터페이스만 6개만 기억하면 나머지를 충분히 유추해낼 수 있다.

기본 인터페이스들은 모두 참조 타입용이다.

Operator 인터페이스는 인수가 1개인 UnaryOperator 2개인 BinaryOperator 나뉘며, 반환값과 인수의 타입이 같은 함수를 뜻한다.

Predicate 인터페이스는 인수 하나를 받아 boolean 반환하는 함수를 뜻한다. Predicate 인터페이스는 인수 하나를 받아 boolean 반환하는 함수를 뜻하며, Function 인터페이스는 인수와 반환 타입이 다른 함수를 뜻한다.

Supplier 인스턴페이스는 인수를 받지 않고 값을 반환하는 함수를, Consumer 인터페이스는 인수를 하나 받고 반환값은 없는 함수를 뜻한다. 

다음은 이 기본 함수형 인터페이스들을 정리한 표다.

| 인터페이스 | 함수 시그니처 | 예 |
| --- | --- | --- |
| UnaryOperator<T> | T apply(T t) | String::toLowerCase |
| BinaryOperator<T> | T apply(T t1, T t2) | BigInteger::add |
| Predicate<T> | boolean test(T t) | Collection::isEmpty |
| Function<T,R> | R apply(T t) | Arrays::asList |
| Supplier<T> | T get() | Instant::now |
| Consumer<T> | void accept(T t) | System.out::println |

기본 인터페이스는 기본 타입인 int, long, double용으로 각 3개씩 변형이 생겨난다. 

그 이름도 기본 인터페이스의 이름 앞에 해당 기본 타입 이름을 붙여 지었다.

예컨대 int를 받는 Predicate는 IntPredicate가 되고 long을 받아 long을 반환하는 BinaryOperator는 LongBinaryOperator가 되는 식이다.

이 변형들 중 유일하게 Function 변형만 매개변수화됐다. 

정확히는 반환 타입만 매개변수화됐는데, 예를 들어 LongFunction<int[]>은 long 인수를 받아 int[]을 반환 

Function 인터페이스에는 기본 타입을 반환하는 변형이 총 9개가 더 있다.

인수와 같은 타입을 반환하는 함수는 UnaryOperator이므로, Function 인터페이스 변형은 입력과 결과의 타입이 항상 다르다.

입력 결과 타입이 모두 기본 타입이면 접두어로 SrcToResult 사용한다. 

예컨대 long을 받아 int 반환하면 LongToIntFunction 되는 식이다.

나머지는 입력이 객체 참조이고 결과가 int, long ,double 변형들로, 앞서와 달리 입력을 매개변수화하고 접두어로 ToResult 사용한다.

ToLongFunction<int[]> int[] 인수를 받아 long을 반환한다.

기본 함수형 인터페이스 중 3개에는 인수를 2개씩 받는 변형이 있다.

주인공은 BiPredicate<T, U> , BiFunction<T,U,R> , BiConsumer<T, U> 다. 

BiFunction 다시 기본 타입을 반환하는 세 변형 ToIntBiFunction<T, U> , ToLongBiFunction<T, U> , ToDoubleBiFunction<T, U>가 존재한다.

Consumer에도 객체 참조와 기본 타입 하나, 즉 인수를 2개 받는 변형인 ObjDoubleConsumer<T>, ObjIntConsumer<T>, ObjLongConsumer<T>가 존재한다.

마지막으로, BooleanSupplier 인터페이스는 boolean 반환하도록 한 Supplier의 변형이다. 

이것이 표준 함수형 인터페이스 중 boolean을 이름에 명시한 유일한 인터페이스지만, Predicate 그 변형 4개도 boolean 값을 반환할 수 있다.

앞서 소개한 42개의 인터페이스에 BooleanSuplier까지 더해서 표준 함수형 인터페이스는 총 43개다.

솔직히 다 외우기엔 수도 많고 규칙성도 부족하다. 

실무에서 자주 쓰이는 함수형 인터페이스 중 상당수를 제공하며, 필요할 때 찾아 쓸 수 있을 만큼은 범용적인 이름을 사용

표준 함수형 인터페이스 대부분은 기본 타입만 지원한다. 기본 함수형 인터페이스에 박싱된 기본 타입을 넣어 사용하지는 말자.

동작은 하지만 "박싱된 기본 타입 대신 기본 타입을 사용하라"라는 아이템 61의 조언을 위배한다.

매개변수 3개를 받는 Predicate라든가 검사 예외를 던지는 경우가 있을 수 있다.

Comparator<T> 인터페이스를 떠올려보자. 구조적 ToIntBiFunction<T,U> 동일한다.

심지어 자바 라이브러리에 Comparator<T> 추가할 당시 ToIntBiFunction<T, U>가 이미 존재했더라도 ToIntBiFunction<T,U>를 사용하면 안 됐다.

Comparator 독자적인 인터페이스로 살아남아야 하는 이유가 몇 개 있다. 

첫 번째, API에서 굉장히 자주 사용되는데, 지금의 이름이 그 용도를 아주 훌륭히 설명해준다. <br/>
두 번째, 구현하는 쪽에서 반드시 지켜야 할 규약을 담고 있다. <br/>
세 번째, 비교자들을 변환하고 조합해주는 유용한 디폴트 메서드들을 듬뿍 담고 있다. <br/>

전용 함수형 인터페이스를 작성하기로 했다면, 자신이 작성하는 게 다른 것도 아닌 '인터페이스'임을 명심해야 한다. 아주 주의해서 설계해야 한다는 뜻이다.(아이템 21).

코드 44-1 의 EldestEntryRemovalFunction 인터페이스에 @Functional Interface 애너테이션이 달려 있음에 주목하자.

첫 번쨰, 해당 클래스의 코드나 설명 문서를 읽을 이에게 그 인터페이스가 람다용으로 설계된 것임을 알려준다. <br/>
두 번째, 해당 인터페이스가 추상 메서드를 오직 하나만 가지고 있어야 컴파일되게 해준다. <br/>
세 번째, 그 결과 유지보수 과정에서 누군가 실수로 메서드를 추가하지 못하게 막아준다. <br/>

### **직접 만든 함수형 인터페이스에는 항상@FunctionalInterface 애너테이션을 사용하라.**

마지막으로, 함수형 인터페이스를 API에서 사용할 때의 주의점을 일러두겠다.

서로 다른 함수형 인터페이스를 같은 위치의 인수로 받는 메서드들을 다중 정의해서는 안 된다. 

클라이언트에게 불필요한 모호함만 안겨줄 뿐이며, 이 모호함으로 인해 실제로 문제가 일어나기도 한다.

ExecutorService submit 메서드는 Callable<T> 받는 것과 Runnable을 받는 것을 다중정의했다.

**다중정의는 주의해서 사용하라**
