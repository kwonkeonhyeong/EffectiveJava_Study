## 스트림은 주의해서 사용하라

## **핵심 정리**

스트림 API는 다량의 데이터 처리 작업(순차적이든 병렬적이든)을 돕고자 자바 8에 추가되었다.

이 API가 제공하는 추상 개념 중 핵심은 두 가지다. 

그 첫번째인 스트림(stream)은 데이터 원소의 유한 혹은 무한 시퀀스(sequence)를 뜻한다.

두 번째인 스트림 파이프라인(stream pipeline)은 이 원소들로 수행하는 연산 단계를 표현하는 개념이다.

스트림의 원소들은 어디로부터든 올 수 있다. 대표적으로는 컬렉션, 배열, 파일, 정규표현식 패턴 매처(matcher), 난수 생성기, 혹은 다른 스트림이 있다.

스트럼 안의 데이터 안의 데이터 원소들은 객체 참조나 기본 타입 값이다. 기본 타입 값으로는 int, long, double 이렇게 세 가지를 지원한다.

스트림 파이프라인은 소스 스트림에서 시작해 종단 연산(terminal operation)으로 끝나며, 그 사이에 하나 이상의 중간 연산(intermediate operation)이 있을 수 있다.

각 중간 연산은 스트림을 어떠한 방식으로 변환(transform)한다. 각 원소에 함수를 적용하거나 특정 조건을 만족 못하는 원소를 걸러낼 수 있다.

중간 연산들은 모두 한 스트림을 다른 스트림으로 변환하는데, 변환된 스트림의 원소 타입은 변환 전 스트림의 원소 타입과 같을 수도 있고 다를 수도 있다.

중단 연산은 마지막 중간 연산이 내놓은 스트림에 최후의 연산을 가한다. 

원소를 정렬해 컬렉션에 담거나, 특정 원소 하나를 선택하거나, 모든 원소를 출력하는 식이다.

스트림 파이프라인은 지연 평가(lazy evaluation)된다. 평가는 종단 연산이 호출될 때 이루어지며, 종단 연산에 쓰이지 않는 데이터 원소는 계산에 쓰이지 않는다.

지연 평가가 무한 스트림을 다룰 수 있게 해주는 열쇠다. 종단 연산이 없는 스트림 파이프라인은 아무 일도 하지 않는 명령어인 no-op과 같으니, 종단 연산을 뺴먹는 일이 없도록 하자.

스트림 API는 메서드 연쇄를 지원하는 플루언트 API(fluent API)다. 즉, 파이프라인 하나를 구성하는 모든 호출을 연결하여 단 하나의 표현식으로 완성할 수 있다.

파이프라인 여러 개를 연결해 표현식 하나로 만들 수도 있다. 기본적으로 스트림 파이프라인은 순차적으로 수행된다. 

파이프라인을 병렬로 실행하려면 파이프라인을 구성하는 스트림 하나에서 parallel 메서드를 호출해주기만 하면 되나, 효과를 볼 수 있는 상황을 많지 않다.

스트림 API 다재다능하여 사실상 어떠한 계산이라도 해낼 수 있다. 

하지만 할 수 있다는 뜻이지, 해야 한다는 뜻은 아니다. 스트림 중 제대로 사용하면 프로그램이 짧고 깔끔해지지만, 잘못 사용하면 읽기 어렵고 유지보수도 힘들어 진다.

스트림을 언제 써야 하는지를 규정하는 확고부동한 규칙은 없지만, 참고할 만한 노하우는 있다. <br/>
이 프로그램은 사전 파일에서 단어를 읽어 사용자가 지정한 문턱값보다 원소 수가 많은 아나그램(anagram) 그룹을 출력한다. <br/>
아나그램이란 철자를 구성하는 알파벳이 같고 순서만 다른 단어를 말한다. 이 프로그램은 사용자가 명시한 사전 파일에서 각 단어를 읽어 맵에 저장한다. <br/>
맵의 키는 그 단어를 구성하는 철자들을 알파벳순으로 정렬한 값이다. 즉 "staple"의 키는 "aelpst" , "petals"가 된다. 

따라서 이 두 단어는 아나그램이고, 아나그램끼리는 같은 키를 공유한다. 

맵의 값은 같은 키를 공유한 단어들을 담은 집합이다. 사전 하나를 모두 처리하고 나면 각 집합은 사전에 등재된 아나그램들을 모두 담은 상태가 된다. <br/>
마지막으로 이 프로그램은 맵의 values() 메서드로 아나그램 집합들을 얻어 원소 수가 문턱값보다 많은 집합들을 출력한다.

```java
import java.io.IOException;
import java.util.Arrays;
import java.util.Scanner;
import java.util.TreeSet;

public clss Anagrams {

public static void main(String[] args) throws IOException {
    File dictionary = new File(args[0]);
    int minGroupSize = Integer.parseInt(args[1]);

    Map<String, Set<String>> groups = new HashMap<>();
    try (Scanner s = new Scanner(dictionary)) {
        while (s.hasNext()) {
            String word = s.next();
            groups.computeIfAbsent(alphabetize(word), (unused) -> new TreeSet<>().add(word));
        }
    }

    for (Set<String> group : groups.values())
        if (group.size() >= minGroupSize)
            System.out.println(group.size() + ": " + group);
}

    private static String alphabetize(String s) {
        char[] a = s.toCharArray();
        Arrays.sort(a);
        return new String(a);
    }
}
```

맵에 각 단어를 삽입할 때, 자바 8에서 추가된 computeIf 메서드 사용했다.

이 메서드는 삽입할 때 자바 8에서 추가된 computeIfAbsent 메서드를 사용했다.

이 메서드는 맵 안에 키가 있는지 찾은 다음, 있으면 단순히 그 키에 매핑된 값을 반환한다. 키가 없으면 건네진 함수 객체를 키에 적용하여 값을 계산해낸 다음 그 키와 값을 매핑해놓고, 계산된 값을 반환한다.

이처럼 computeIfAbsent 사용하면 각 키에 다수의 값을 매핑하는 맵을 쉽게 구현할 수 있다.

앞의 코드와 똑같지만, 스트림을 과하게 사용한다.

**스트림을 과하게 사용했다. - 따라 하지 말 것!**

```java
import java.util.stream.Stream;

public class Anagrams {
    public static void main(String[] args) {
        Path dictionary = Paths.get(args[0]);
        int minGroupSize = Integer.parseInt(args[1]);

        try (Stream<String> words = Files.lines(dictionary)) {
            words.collect(
                    groupingBy(word -> word.chars().sorted()
                                    .collect(StringBuilder::new, (sb, c) -> sb.append((char) c),
                                            StringBuilder::append).toString()))
                    .values().stream()
                    .filter(group -> group.size() >= minGroupSize)
                    .map(group -> group.size() + ": " + group)
                    .forEach(System.out::println);
        }
    }
}
```

**위에 코드처럼 스트림 코드를 과하게 사용하면 프로그램이 읽거나 유지보수하기 어려워진다.**

**코드 45-3 스트림을 적절히 활용하면 깔끔하고 명료해진다.**
```java
public class Anagrams {
    public static void main(String[] args) {
        Path dicionary = Paths.get(args[0]);
        int minGroupSize = Integer.parseInt(args[1]);
        
        try (Stream<String> words = Files.lines(dicionary)) {
            words.collect(groupingBy(word -> alphabetize(word)))
                    .values().stream()
                    .filter(group -> group.size() >= minGroupSize)
                    .forEach(g -> System.out.println(g.size() + ": "+ g));
        }
    }
}
```

스트림을 전에 본 적 없더라도 이 코드는 이해하기 쉬울 것이다. try-with-resources 블록에서 사전 파일을 열고, 파일의 모든 라인으로 구성된 스트림을 얻는다.

스트림 변수의 이름을 words로 지어 스트림 안의 각 원소가 단어(word)임을 명확히 했다.

이 스트림의 파이프라인에는 중간 연산은 없으며, 종단 연산에서는 모든 단어를 수집해 맵으로 모은다. 

이 맵은 단어들을 아나그램끼리 묵어놓은 것으로, 앞선 두 프로그램이 생성한 맵과 실질적으로 같다. 

그 다음으로 이 맵의 values()가 반환한 값으로부터 새로운 Stream<List<String>>스트림을 연다. 이 스트림의 원소는 물론 아나그램 리스트다.

**람다에서는 타입 이름을 자주 생략하므로 매개변수 이름을 잘 지어야 스트림 파이프라인의 가독성이 유지된다.**

연산에 적절한 이름을 지어주고 세부 구현을 주 프로그램 로직 밖으로 뺴내 전체적인 가독성을 높인 것이다.

**도우미 메서드를 적절히 활용하는 일의 중요성은 일반 반복코드에서보다는 스트림 파이프라인에서 훨씬 크다.**

alphabetize 메서드도 스트림을 사용해 다르게 구현할 수 있다. 하지만 그렇게하면 명확성이 떨어지고 잘못 구현할 가능성이 커진다.

심지어 느려질 수도 있다. 자바가 기본 타입인 char 용 스트림을 지원하지 않기 때문이다.

그렇게 하는 건 불가능했다. 문제를 직접 보여주는 게 나을 것 같아 char 값들을 스트림으로 처리하는 코드를 준비해보았다.

```java
"Hello world!".chars().forEach(System.out::println);
```

Hello world! 출력 안됨

**char 값들을 처리할 때 스트림을 삼가는 편이 낫다.**

기존 코드는 스트림을 사용하도록 리팩터링하되, 새코드가 더 나아 보일 때만 반영하자.

- 코드 블록에서는 범위 안의 지역변수를 읽고 수정할 수 있다. 하지만 람다에서는 final 이거나 사실상 final인 변수만 읽을 수 있고, 지역변수를 수정하는 건 불가능하다.
- 코드 블록에서는 return 문을 사용해 메서드에서 빠져나가거나, break나 continue문으로 블록 바깥의 반복문을 종료하거나 반복을 한 번 건너뛸 수 있다. 또한 메서드 선언에 명시된 검사 예외를 던질 수 있다.

계산 로직에서 이상의 일들을 수행해야 한다면 스트림과는 맞지 않는 것이다.

반대로 다음 일들에는 스트림이 아주 안성맞춤이다.
- 원소들의 시퀀스를 일관되게 변환한다.
- 원소들의 시퀀스를 필터링한다.
- 원소들의 시퀀스를 하나의 연산을 사용해 결합한다.(더하기, 연결하기, 최솟값 구하기등)
- 원소들의 시퀀스를 컬렉션에 모은다.
- 원소들의 시퀀스에서 특정 조건을 만족하는 원소를 찾는다.

스트림으로 처리하기 어려운 일 한 데이터가 파이프라인의 여러 단계(stage)를 통과할 때 이 데이터의 각 단계에서의 값들에 동시에 접근하기는 어려운 경우다.

스트림 파이프라인은 일단 한 값을 다른 값에 매핑하고 나면 원래의 값은 잃는 구조이기 때문이다. 원래 값과 새로운 값의 쌍을 저장하는 객체를 사용해 매핑하는 우회 방법도 있지만, 그리 만족스러운 해법은 아닐 것이다.

매핑 객체가 필요한 단계가 여러 곳이라면 특히 더 그렇다. 이런 방식은 코드 양도 많고 지저분하여 스트림을 쓰는 주목적에서 완전히 벗어난다.

예시, 메르센 소수를 출력하는 프로그램을 작성해보자.

```java
static Stream<BigInteger> primes() {
    return Stream.iterate(TWO, BigInteger::nextProbablePrime);
}
```

메서드 이름 primes는 스트림의 원소가 소수임을 말해준다. 

스트림을 반환하는 메서드 이름은 원소의 정체를 알려주는 복수 명사로 쓰기를 강력히 추천한다.

스트림 파이프라인이 가독성이 크게 좋아질 것이다. 이 메서드가 이용하는 Stream.iterate라는 정적 팩터리는 매개변수를 2개 받는다.

첫 번째 매개변수는 스트림의 첫 번째 원소이고, 두 번째 매개변수는 스트림에서 다음 원소를 생성해주는 함수다.

```java
public static void main(String[] args) {
    primes().map(p-> TWO.pow(p.intValueExact()).subtract(ONE))
            .filter(mersenne -> mersenne.isProbablePrime(50))
            .limit(20)
            .forEach(System.out::println);
}
```

소수들을 사용해 메르센 수를 계산하고, 결괏값이 소수인 경우만 남긴 다음, 결과 스트림의 원소 수를 20개로 제한해놓고, 작업이 끝나면 결과를 출력한다.

이제 우리가 각 메르센 소수의 앞에 지수(p)를 출력하길 원한다고 해보자. 이 값은 초기 스트림에만 나타나므로 결과를 출력하는 종단 연산에서는 접근할 수 없다.

하지만 다행히 첫 번째 중간 연산에서 수행한 매핑을 거꾸로 수행해 메르센 수의 지수를 쉽게 계산할 수 있다.

지수는 단순히 숫자를 이진수로 표현한 다음 몇 비트인지를 세면 나오므로, 종단 연산을 다음처럼 작성하면 원하는 결과를 얻을 수 있다.

```java
.forEach(mp -> System.out.println(mp.bitLength() + ": " + mp));
```

스트림과 반복 중 어느 쪽을 써야 할지 바로 알기 어려운 작업도 많다. 카드 덱을 초기화하는 작업을 생각해보자. 카드는 숫자(rank)와 무늬(suit) 묶은 불변 값 클래스이고, 숫자와 무늬는 모두 열거타입이라 하자.

이 작업은 두 집합의 원소들로 만들 수 있는 가능한 모든 조합을 계산하는 문제다. 수학자들은 이를 두 집합의 데카르트 곱이라고 부른다. 다음은 for-each 반복문을 중처해서 구현한 코드로, 스트림에 익숙하지 않은 사람에게 친숙한 방식일 것이다.

**코드 45-4 데카르트 곱 계산을 반복 방식으로 구현**
```java
private static List<Card> newDeck() {
    List<Card> result = new ArrayList<>();
    for(Suit suit : Suit.values()) 
        for(Rank rank : Rank.values())
            result.add(new Card(suit, rank));
    return result;
}
```

다음은 스트림으로 구현한 코드다 중간 연산으로 사용한 flatMap은 스트림 원소 각각을 하나의 스트림으로 매핑한 다음 그 스트림들을 다시 하나의 스트림으로 합친다. 이를 평탄화(flattening)라고도 한다.

**코드 45-5 데카르트 곱 계산을 스트림 방식으로 구현**
```java
private static List<Card> newDeck() {
    return Stream.of(Suit.values())
            .flatMap(suit -> 
                Stream.of(Rank.values())
                        .map(rank -> new Card(suit, rank)))
            .collect(toList());
}
```

어느 newDeck이 좋아 보이는가? 결국은 개인 취향과 프로그래밍 환경의 문제다.
