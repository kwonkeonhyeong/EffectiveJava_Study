## 스트림에서는 부작용 없는 함수를 사용하라

## **핵심정리**

스트림은 그저 또 하나의 API가 아닌, 함수형 프로그래밍에 기초한 패러다임이다. 

스트림 패러다임의 핵심은 계산은 일련의 변환(transformation)으로 재구성하는 부분이다. 

이때 각 변환 단계는 가능한 한 이전 단계의 결과를 받아 처리하는 순수 함수여야 한다. 

순수 함수란 오직 입력만이 결과에 영향을 주는 함수를 말한다. 다른 가변 상태를 참조하지 않고, 함수 스스로도 다른 상태를 변경하지 않는다. 이렇게 하려면 (중간 단계든 종단 단계든) 스트림 연산에 건네는 함수 객체는 모두 부작용(side effect) 없어야 한다. 

## 코드 46-1 스트림 패러다임을 이해하지 못한 채 API만 사용했다. - 따라 하지 말 것!
```java
Map<String, Long> freq = new HashMap<>();
try(Stream<String> words = new Scanner(file).tokens()) {
    word.forEach(word -> {
        freq.merge(word.toLowerCase(), 1L, Long::sum);
    });
}
```

스트림, 람다, 메서드 참조를 사용했고, 결과도 올바르다. 하지만 절대 스트림 코드라 할 수 없다. 

스트림 코드를 가장한 반복적 코드다. 스트림 API 이점을 살리지 못하여 같은 기능의 반복적 코드 보다 길고, 읽기 어렵고, 유지보수에도 좋지 않다. 

이 코드의 모든 작업이 종단 연산이 forEach에서 일어나는데, 이때 외부 상태(빈도표)를 수정하는 람다를 실행하면서 문제가 생긴다. 

## 코드 46-2 스트림 제대로 활용해 빈도표를 초기화한다.
```java
Map<String, Long> freq;
try(Stream<String> words = new Scanner(file).tokens()) {
    freq = words.collect(groupingBy(String::toLowerCase, counting()));
}
```
forEach 연산은 종단 연산 중 기능이 가장 적고 가장 '덜' 스트림 답다. 

대놓고 반복적이라서, 병렬화도 불가능하다. forEach 연산은 스트림 계산 결과를 보고할 때만 사용하자. 

위에 코드는 수집기(collector) 사용하는 데, 스트림을 사용하려면 꼭 배워야 하는 새로운 개념

java.util.stream.Collectors 클래스는 메서드를 무려 39개나 가지고 있고, 그중에는 타입 매개변수가 5개나 되는 것도 있다. 

다행히 복잡한 세부 내용을 잘 몰라도 이 API 장점을 대부분 활용할 수 있다. 

익숙해지기 전까지는 Collector 인터페이스 잠시 잊고, 그저 축소(reduction) 전략을 캡슐화한 블랙박스 객체라고 생각하기 바란다. 여기서 축소는 스트림의 원소들을 객체 하나에 취합한다는 뜻이다. 

수집기는 사용하면 스트림의 원소를 손쉽게 컬렉션으로 모을 수 있다. 수집기는 총 세 가지로, toList(), toSet(), toCollection(collectionFactory)가 그 주인공이다. 

이들은 차례로 리스트, 집합, 프로그래머가 지정한 컬렉션 타입을 반환한다. 지금까지 배운 지식을 활용해 빈도표에서 가장 흔한 단어 10개를 뽑아내는 스트림 파이프라인을 작성해보자. 

## 코드 46-3 빈도표에서 가장 흔한 단어 10개를 뽑아내는 파이프라인 
```java
List<String> topTen = freq.keySet().stream()
        .sorted(comparing(freq::get).reversed())
        .limit(10)
        .collect(toList());
```

**마지막 toList는 Collectors 메서드다. Collectors 멤버를 정적 임포트하여 쓰면 파이프라인 가독성이 좋아져 흔히들 이렇게 사용한다.**

코드에서 어려운 부분은 sorted에 넘긴 비교자, 즉 comparing(freq::get).reversed()뿐이다. comparing 메서드는 키 추출 함수를 받는 비교자 생성 메서드다.

그리고 한정적 메소드 참조이자, 여기서 키 추출 함수로 쓰인 freq::get은 입력받은 단어(키)를 빈도표를 찾아(추출) 그 빈도를 반환한다.

가장 흔한 단어가 오도록 비교자(comparing) 역순(reversed)으로 정렬한다(sorted).

Collectors 나머지 36개 메서드들도 알아보자. 이 중 대부분은 스트림을 맵으로 취합하는 기능으로, 진짜 컬렉션에 취합하는 것보다 훨씬 복잡하다. 

스트림의 각 원소는 키 하나와 값 하나에 연관되어 있다. 

가장 간단한 맵 수집기는 toMap(keyMapper, valueMapper) 보다시피 스트림 원소를 키에 매핑하는 함수와 값에 매핑하는 함수를 인수로 받는다. 

## 코드 46-4 toMap 수집기를 사용하여 문자열을 열거 타입 상수에 매핑한다. 
```java
private static final Map<String, Operation> stringToEnum = Stream.of(values()).collect(toMap(Object::toString, e -> e));
```

toMap 형태는 스트림의 각 원소가 고유한 키에 매핑되어 있을 때 적합하다. 스트림 원소 다수가 같은 키를 사용한다면 파이프라인이 IlleagalStateException 던지며 종료 

더 복잡한 형태의 toMap이나 groupingBy는 이런 충돌을 다루는 다양한 전략을 제공한다. 

toMap에 키 매퍼와 값 매퍼는 물론 병합(merge) 함수까지 제공할 수 있다. 병합 함수의 형태는 BinaryOperator<U> 이며, 여기서 U는 해당 맵의 값 타입이다. 

같은 키를 공유하는 값들은 이 병합 함수를 사용해 기존 값에 합쳐진다. 예컨대 병합 함수가 곱셈이라면 키가 같은 모든 값(키/값 매퍼가 정한다)을 곱한 결과를 얻는다. 

인수 3개를 받는 toMap은 어떤 키와 그 키에 연관된 원소들 중 하나를 골라 연관 짓는 맵을 만들 때 유용하다. 예컨대 다양한 음악가의 앨범들을 담은 스트림을 가지고, 음악가와 그 음악가의 베스트 앨범을 연관 짓고 싶다고 해보자. 

## 각 키와 해당 키의 특정 원소를 연관 짓는 맵을 생성하는 수집기
```java
Map<Artist, Album> topHits = albums.collect(toMap(Album::artist, a-> a, maxBy(comparing(Album::sales))));
```
비교자로는 BinaryOperator에서 정적 임포트한 maxBy라는 정적 팩터리 메서드를 사용

maxBy는 Comparator<T> 입력받아 BinaryOperator<T>를 돌려준다. 이 경우 비교자 생성 메서드인 comparing maxBy인 넘겨줄 비교자를 반환 

자신의 키 추출 함수로는 Album::sales 받았다. 

앨범 스트림을 맵으로 바꾸는데, 이 맵은 각 음악가와 그 음악가의 베스트 앨범을 짝지은 것이다. 

인수가 3개인 toMap은 충돌이 나면 마지막 값을 취하는(last-write-wins) 수집기를 만들 때도 유용하다. 많은 스트림의 결과가 비결정적이다.

하지만 매핑 함수가 키 하나에 연결해준 값들이 모두 같을 때, 혹은 값이 다르더라도 모두 허용되는 값일 때

## 코드 46-7 마지막에 쓴 값을 취하는 수집기
```java
toMap(keyMapper, valueMapper, (oldVal, newVal) -> newVal)
```
세 번째이자 마지막 toMap 네 번째 인수로 맵 팩토리를 받는다. 

이 인수로는 EnumMap이나 TreeMap처럼 원하는 특정 맵 구현체를 직접 지정할 수 있다. 

세 가지 toMap에는 변종이 있다. 그중 toConcurrentMap은 병렬 실행된 후 결과로 ConcurrentMap 인스턴스를 생성한다. 

이번에는 Collectors가 제공하는 또 다른 메서드인 groupingBy를 알아보자. 

이 메서드는 입력으로 분류 함수(classifier)받고 출력으로는 원소들을 카테고리별로 모아 놓은 맵을 담은 수집기를 반환한다. 분류 함수는 입력받은 원소가 속하는 카테고리를 반환한다. 

분류 함수는 입력받은 원소가 속하는 카테고리를 반환한다. 그리고 이 카테고리가 해당 원소의 맵 키로 쓰인다. 

다중정의된 groupingBy 중 형태가 가장 간단한 것은 분류 함수 하나를 인수로 받아 맵을 반환

반환된 맵에 담긴 각각의 값은 해당 카테고리에 속하는 원소들을 모두 담은 리스트다. 

```java
words.collect(groupingBy(word -> alphabetize(word)))
```

groupingBy가 수집기가 리스트 외의 값을 갖는 맵을 생성하게 하려면, 분류 함수와 함께 다운스트림(downstream) 수집기도 명시해야 한다. 

다운 스트림 수집기의 역할은 해당 카테고리의 모든 원소를 담은 스트림으로부터 값을 생성하는 일이다. 이 매개변수를 사용하는 가장 간단한 방법은 toSet()을 넘기는 것이다. 

groupingBy는 원소들의 리스트가 아닌 집합(Set)을 값으로 갖는 맵을 만들어낸다. 
toSet() 대신 toCollection(collectionFactory)를 건네는 방법도 있다. 

```java
Map<String, Long> freq = words.collect(groupingBy(String::toLowerCase, countin()));
```

groupingBy 세 번째 버전은 다운스트림 수집기에 더해 맵 팩터리도 지정할 수 있게 해준다. 
참고로 이 메서드는 점층적 인수 목록 패턴(telescoping argument list pattern)에 어긋난다. 

즉, mapFactory 매개변수가 downStream 매개변수보다 앞에 놓인다. 이 버전의 groupingBy를 사용하면 맵과 그 안에 담긴 컬렉션의 타입을 모두 지정할 수 있다. 

예컨대 값이 TreeSet인 TreeMap 반환하는 수집기를 만들 수 있다.
총 세 가지 groupingBy 각각에 대응하는 groupingByConcurrent 메서드들도 볼 수 있다. 

이름에서 알 수 있듯 대응하는 메서드의 동시 수행 버전으로 ConcurrentHashMap 인스턴스를 만들어준다. 
많이 쓰이진 않지만 groupingBy의 사촌격인 partitionBy도 있다. 분류 함수 자리에 프레디키트(predicate)를 받고 키가 Boolean인 맵을 반환한다. 

Collectors의 마지막 메서드는 joining이다. 이 메서드는 (문자열 등의) CharSequence 인스턴스의 스트림에만 적용할 수 있다. 이 중 매개변수가 없는 joining은 단순히 원소들을 연결(concatenate)하는 수집기를 반환한다. 

한편 인수 하나짜리 joining CharSequence 타입의 구분문자(delimiter) 매개변수로 받는다. 연ㄱ녈 부위에 이 구분문자를 삽입하는 데, 구분문자로 쉼표(,)를 입력하면 CSV 형태의 문자열을 만들어준다. 
