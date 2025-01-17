## 📖 스트림은 주의해서 사용하라

### 핵심 내용

스트림을 사용해야 멋지게 처리할 수 있는 일이 있고 반복 방식이 더 알맞은 일도 있다.

수많은 작업은 이 둘을 조합했을 때 가장 멋지게 해결된다.

어느 쪽을 선택하는 확고부동한 규칙은 없지만 참고할 만한 지침 정도는 있다.

**스트림과 반복 중 어느 쪽이 나은지 확신하기 어렵다면 둘 다 해보고 더 나은 쪽을 택하라**


## 💡 주요 내용 정리 & 🛠️ 실습 코드

스트림 API는 다량의 데이터 처리 작업(순차적이든 병렬적이든)을 돕고자 자바 8에 추가되었다.

1. 스트림은 데이터 원소의 유한 혹은 무한 시퀀스를 뜻한다.
2. 스트림 파이프라인은 이 원소들로 수행하는 연산 단계를 표현하는 개념이다.

스트림의 원소들은 어디로부터든 올 수 있다.

스트림 안의 데이터 원소들은 객체 참조나 기본 타입 값이다. (기본 타입 값으로는 int, long, double) 이렇게 세 가지를 지원한다.

스트림 파이프라인은 소스 스트림에서 시작해 종단 연산(terminal operation)으로 끝나며, 그 사이에 하나 이상의 중간 연산 (intermediate operation)이 있을 수 있다.

중산 연산들은 모두 한 스트림을 다른 스트림으로 변환하는데, 변환된 스트림의 원소 타입은 변환 전 스트림의 원소 타입과 같을 수도 있고 다를 수도 있다.

종단 연산은 마지막 중산 연산이 내놓은 스트림에 최후의 연산을 가한다.

---

### 스트림 파이프라인은 지연 평가 (lazy evaluation)된다.

평가는 종단 연산이 호출될 때 이뤄지먀, 종단 연산에 쓰이지 않는 데이터 원소는 계산에 쓰이지 않는다.

종단 연산이 없는 스트림 파이프라인은 아무 일도 하지 않는 명령어인 no-op과 같으니, 종단 연산을 빼먹는 일이 절대 없도록 하자!


사전 하나를 훑어 원소 수가 많은 아나그램 그룹들을 출력한다.
```java
public class Anagrams {
    public static void main(String[] args) throws IOException {
        File dictionary = new File(args[0]);
        int minGroupSize = Integer.parseInt(args[1]);
        
        Map<String, Set<String>> groups = new HashMap<>();
        try (Scanner s = new Scanner(dictionary)) {
            while (s.hasNext()) {
                String word = s.next();
                groups.computeIfAbsent(alphabetize(word), (unused) -> new TreeSet<>().add(word)
                );
            }
        }
        
        for (Set<String> group : groups.values())
            if (group.size() >= minGroupSize)
                System.out.println(group.size() + ": " + group);
    }

    private static String alphabetize(String string) {
        char[] a = s.toCharArray();
        Arrays.sort(a);
        return new String(a);
    }
}
```

스트림을 과하게 사용했다 - 따라 하지 말 것!
```java
public class Anagrams {
    public static void main(String[] args) throws IOException {
        Path dictionary = Paths.get(args[0]);
        int minGroupSize = Integer.parseInt(args[1]);

        try (Stream<String> words = Files.lines(dictionary)) {
            words.collect(
                            groupingBy(word -> word.chars().sorted()
                                    .collect(StringBuilder::new,
                                            (sb, c) -> sb.append((char) c),
                                            StringBuilder::append
                                    ).toString()))
                    .values.stream()
                    .filter(group -> group.size() >= minGroupSize)
                    .map(group -> group.size() + ": " + group)
                    .forEach(System.out::println);
        }
    }
}
```

**스트림을 과용하면 프로그램이 읽거나 유지보수하기 어려워진다.**

스트림을 적절히 활용하면 깔끔하고 명료해진다.
```java
public class Anagrams {
    public static void main(String[] args) throws IOException {
        Path dictionary = Paths.get(args[0]);
        int minGroupSize = Integer.parseInt(args[1]);

        try (Stream<String> words = Files.lines(dictionary)) {
            words.collect(groupingBy(word -> alphabetsize(word)))
                    .values().stream()
                    .filter(group -> group.size() >= minGroupSize)
                    .forEach(g -> System.out.println(g.size() + ": " + g));
        }
        
        // alphabetsize 메서드 코드 45-1과 같다.
    }
}
```

--- 

**람다 매개변수의 이름은 주의해서 정해야 한다.**

람다에서는 타입 이름을 자주 생략하므로 매개변수 이름을 잘 지어야 스트림 파이프라인의 가독성이 유지된다.

도우미 메서그를 적절히 활용하는 일의 중요성을 일반 반복코드에서보다는 스트림 파이프라인에서 훨씬 크다.

---

--- 

**기존 코드는 스트림을 사용하도록 리펙터링하되, 새 코드가 더 나아 보일 때만 반영하자**

---

스트림 파이프라인은 되풀이되는 계산을 함수 객체로 표현한다.

반면 반복 코드에서는 코드 블록을 사용해 표현한다.

#### **함수객체로는 할 수 없지만 코드 블록으로는 할 수 있는 일들**

- 코드 블록에서는 범위 한의 지역변수를 읽고 수정할 수 있다.

- 하지만 람다에서는 final이거나 사실상 final인 변수만 읽을 수 있고, 지역변수를 수정하는 건 불가능하다.

- 코드 블록에서는 return 문을 사용해 메서드에서 빠져나가거나, break나 continue 문으로 바깥의 반복문을 종료하거나 반복을 한 번 건너뛸 수 있다.

- 메서드 선언에 명시된 검사 예외를 던질 수 있다.

- 하지만 람다로는 이 중 어떤 것도 할 수 없다.

계산 로직에서 이상의 일들을 수행해야 한다면 스트림과는 맞지 않는 것이다.

#### **스트림이 안성 맞춤인 경우**

- 원소들의 시퀀스를 일관되게 변환한다.
- 원소들의 시퀀스를 필터링한다.
- 원소들의 시퀀스를 하나의 연산을 사용해 결합한다. (더하기, 연결하기, 최솟값 구하기 등)
- 원소들의 시퀀스를 컬렉션에 모은다. (아마도 공통된 속성을 기준으로 묶어가며)
- 원소들의 시퀀스에서 특정 조건을 만족하는 원소를 찾는다.

이러한 일 중 하나를 수행하는 로직이라면 스트림을 적용하기에 좋은 후보다.

---

데카르트 곱 계산을 반복 방식으로 구현
```java
private static List<Card> newDeck() {
    List<Card> result = new ArrayList<>();
    for (Suit suit : Suit.values())
        for (Rank rank : Rank.values())
            result.add(new Card(suit, rank));
    return result;
}
```

데카르트 곱 계산을 스트림 방식으로 구현
```java
private static List<Card> newDeck() {
    return Stream.of(Suit.values())
            .flatMap(suit ->
                    Stream.of(Suit.values())
                            .map(rank -> new Card(suit, rank)))
            .collect(toList());
}
```
중간 연산으로 사용한 flatMap은 스트림의 원소 각각을 하나의 스트림으로 매핑한 다음 그 스트림들을 다시 하나의 스트림으로 합친다 => 평탕화(flattening)


## 추가 강의 내용 정리

## 🤔 생각 정리
- 궁금한 점, 논의하고 싶은 내용

