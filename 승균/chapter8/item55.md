## **item55: 옵셔널 반환은 신중히 하라**

## **핵심정리**

자바 8 전에는 메서드가 특정 조건에서 값을 반환할 수 없을 때 취할 수 있는 선택지가 두 가지 있었다. 예외를 던지거나,(반환 타입이 객체 참조라면)null을 반환하는 것이다. 두 가지 방법 모두 허점이 있다. 

null 반환하면 이런 문제가 생기지 않지만, 그 나름의 문제가 있다. 

null을 반환할 수 있는 메서드를 호출할 떄는, (null이 반환될 일이 절대 없다고 확신하지 않는 한) 별도의 null 처리 코드를 추가해야 한다.

자바 버전이 8로 올라가면서 또 하나의 선택지가 생겼다. 그 주인공인 Optional<T>는 null이 아닌 T타입 참조를 하나 담거나, 혹은 아무것도 담지 않을 수 있다. 

반대로 어떤 값을 담은 옵셔널은 '비지 않았다'고 한다.

옵셔널은 원소를 최대 1개 가질 수 있는 '불변' 컬렉션이다. 

Optional<T>가 Collection<T> 구현하지는 않았지만, 원친적으로 그렇다는 말이다.

보통 T 반환해야 하지만 특정 조건에서는 아무것도 반환하지 않아야 할 때 T 대신 Optional<T> 반환하도록 선언하면 된다. 

유효한 반환값이 없을 떄는 빈 결과를 반환하는 메서드가 만들어진다.

옵셔널을 반환하는 메서드는 예외를 던지는 메서드보다 사용하기 쉬우며, null 반환하는 메서드보다 오류 가능성이 작다.

## 코드 55-1 컬렉션에서 최댓값을 구한다
```java
public static <E extends Comparable<E>> E max(Collection<E> c) {
    if(c.isEmpty())
        throw new IllegalArgumentException("빈 컬렉션");
    
    E result = null;
    for(E e : c) 
        if(result == null || e.compareTo(result) > 0)
            result = Objects.requireNonNull(e);
    
    return result;
}
```

빈 컬렉션을 건네면 IllegalArgumentException 던진다.

## 코드 55-2 컬렉션에서 최댓값을 구해 Optional<E>로 반환한다.

```java
import java.util.Objects;

public static <E extends Comparable<E>> Optional<E> max(Collection<E> c) {
    if (c.isEmpty())
        return Optional.empty();

    E result = null;
    for (E e : c)
        if (result == null || e.compareTo(result) > 0)
            result = Objects.requireNonNull(e);
    return Optional.of(result);
}
```

옵셔널을 반환하도록 구현하기는 어렵지 않다. 적절한 정적 팩터리를 사용해 옵셔널을 생성해주기만 하면 된다. 

빈 옵셔널은 Optional.empty() 만들고, 값이 든 옵셔널은 Optional.of(value) 생성했다. Optional.of(value) null 넣으면 NullPointerException 던지니 주의하자. 

null 값도 허용하는 옵셔널을 만들려면 Optional.ofNullable(value) 사용하면 된다.

**옵셔널을 반환하는 메서드에서는 절대 null을 반환하지 말자.**


## 코드 55-3 컬렉션에서 최댓값을 구해 Optional<E>로 반환한다. 스트림 버전

```java
import java.util.Comparator;

public static <E extends Comparable<E>> Optional<E> max(Collection<E> c) {
    return c.stream().max(Comparator.naturalOrder());
}
```

**옵셔널은 검사 예외와 취지가 비슷하다**

비검사 예외를 던지거나 null 반환한다면 API 사용자가 사실을 인지하지 못해 끔찍한 결과로 이어질 수 있다. 하지만 검사 예외를 던지면 클라이언트에서는 반드시 이에 대처하는 코드를 작성해넣어야 한다.

## 코드 55-4 옵셔널 활용 1 - 기본값을 정해둘 수 있다.
```java
String lastWordInLexicon = max(words).orElse("단어 없음...");
```

다음 코드에서 실제 예외가 아니라 예외 팩터리를 건넨 것에 주목하자. 이렇게하면 예외가 실제로 발생하지 않는한 예외 생성 비용은 들지 않는다. 

## 코드 55-5 옵셔널 활용 2 - 원하는 예외를 던질 수 있다.
```java
Toy myToy = max(toys).orElseThrow(TemperTantrumException::new);
```

## 코드 55-6 옵셔널 활용 3- 항상 값이 채워져있다고 가정한다.
```java
Element lastNobleGas = max(Element.NOBLE_GASES).get();
```

Supplier<T> 인수로 받는 orElseGet 사용하면, 값이 처음 필요할 때 Supplier<T> 사용해 생성하므로 초기 설정 비용을 낮출 수 있다. 

더 특별한 쓰임에 대비한 메서드도 준비되어있다. 바로 filter, map, flatMap, ifPresent 다. 앞서의 기본 메서드로 처리하기 어려워 보인다면 API 문서를 참조해 고급 메서드들이 문제를 해결해줄 수 있을지 검토해보자.

적합한 메서드를 찾지 못했다면 isPresent 메서드를 살펴보자. 안전 밸브 역할의 메서드로, 옵셔널이 채워져 있으면 true를, 비어 있으면 false 반환한다. 

이 메서드로는 원하는 모든 작업을 수행할 수 있지만 신중히 사용해야 한다. 

실제로 isPresent 쓴 코드 중 상당수는 앞서 언급한 메서드들로 대체할 수 있으며, 그렇게 하면 더 짧고 명확하고 용법에 맞는 코드가 된다. 

```java
Optional<ProcessHandle> parentProcess = ph.parent();
System.out.println("부모 PID: " + (parentProcess.isPresent() ? String.valueOf(parentProcess.get().pid()) : "N/A"));
```

코드는 Optional의 map 사용하여 다음처럼 다듬을 수 있다. 

```java
System.out.println("부모 PID: " + ph.parent().map(h -> String.valueOf(h.pid))).orElse("N/A"));
```

스트림을 사용한다면 옵셔널들을 Stream<Optional<T>>로 받아서, 그중 채워진 옵셔널들에서 값을 뽑아 Stream<T> 건넨 담아 처리하는 경우가 드물지 않다. 

```java
streamOfOptionals.filter(Optional::isPresent).map(Optional::get);
```

옵셔널에 값이 있다면 (Optional::isPresent) 값을 꺼내 Optional::get 스트림에 매핑한다. 

```java
streamOfOptionals.flatMap(Optional::stream)
```

컬렉션, 스트림, 배열, 옵셔널 같은 컨테이너 타입은 옵셔널로 감싸면 안 된다.

결과가 없을 수 있으며, 클라이언트가 이 상황을 특별하게 처리해야 한다면 Optional<T> 반환한다.

박싱된 기본 타입을 담는 옵셔널은 기본 타입 자체보다 무거울 수밖에 없다. 
자바 API 설계자는 int, long, double 전용 옵셔널 클래스들을 준비해놨다.
OptionalInt, OptionalLong, OptionalDouble이다.

Optional<T> 제공하는 메서드 거의 다 제공한다.

박싱된 기본 타입을 담은 옵셔널을 반환하는 일은 없도록 하자.

덜 중요한 기본 타입용인 Boolean, Byte, Character, Short, Float 예외일 수 있다.

옵셔널 맵의 값으로 사용하면 절대 안됨. 만약 그리 한다면 맵 안에 키가 없다는 사실을 나타내는 방법이 두 가지가 된다.

옵셔널을 컬렉션의 키, 값, 원소나 배열의 원소로 사용하는 게 적절한 상황은 거이 없다.

NutritionFacts 인스턴스 필드 중 상당수는 필수가 아니다. 
