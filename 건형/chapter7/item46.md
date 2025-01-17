## 📖 스트림에서는 부작용 없는 함수를 사용하라

### 핵심 내용

스트림 파이프라인 프로그래밍의 핵심은 부작용 없는 함수 객체에 있다.

스트림뿐 아니라 스트림 관련 객체에 건네지는 모든 함수 객체가 부작용이 없어야 한다.

종단 연산 중 forEach는 스트림이 수행한 계산 결과를 보고할 때만 이용해야 한다.

계산 자체에는 이용하지 말자.

스트림을 올바로 사용하려면 수집기를 잘 알아둬야 한다.

가장 중요한 수집기 팩터리는 toList, toSet, toMap, groupingBy, joining 이다.


## 💡 주요 내용 정리 & 🛠️ 실습 코드

스트임 패러다임의 핵심은 계산을 일련의 변환(transformation)으로 재구성하는 부분이다.

이때 각 변환 단계는 가능한 한 이전 단계의 결과를 받아 처리하는 순수 함수여야 한다.

순수 함수란 오직 입력만이 결과에 영향을 주는 함수를 말한다.

다른 가변 상태를 참조하지 않고, 함수 스스로도 다른 상태를 변경하지 않는다.

이렇게 하려면 (중간 단계든 종단 단계든) 스트림 연산에 건네는 함수 객체는 모두 부작용(side effect)가 없어야 한다.


스트림 패러다임을 이해하지 못한 채 API만 사용했다 - 따라하지 말 것!
```java
public static void main(String[] args) {
    Map<String, Long> frep = new HashMap();
    try (Stream<String> words = new Scanner(file).tokens()) {
        words.forEach(word -> {
            frep.merge(word, toLowerCase(), 1L, Long::sum);
        });
    }
}
```

위 코드의 모든 작업이 종단 연산인 forEach에서 일어나는데, 이때 외부 상태를 수정하는 람다를 실행하면서 문제가 생긴다.

forEach가 그저 스트림이 수행한 연산 결과를 보여주는 일 이상을 하는 것을 보니 나쁜 코드의 냄새가 난다.

스트림을 제대로 활용해 빈도표를 초기화한다.
```java
public static void main(String[] args) {
    Map<String, Long> freq;
    try (Stream<String> words = new Scanner(file).tokens()) {
        freq = words
                .collect(groupingBy(String::toLowerCase, counting()));
    }
}
```

**forEach 연산은 스트림 계산 결과를 보고할 때만 사용하고, 계산하는 데는 쓰지 말자!**

물론 가끔은 스트림 계산 결과를 기존 컬렉션에 추가하는 등의 다른 용도로도 쓸 . 있다.

---

수집기를 사용하면 스트림의 원소를 쉽게 컬렉션으로 모을 수 있다.

toList(), toSet(), toCollection(collectionFactory)

이들은 차례로 리스트, 집합, 프로그래머가 지정한 컬렉션 타입을 반환한다.

빈도표에서 가장 흔한 단어 10개를 뽑아내는 파이프라인
```java
List<String> topTen = freq.keySet().stream()
        .sorted(comparing(freq::get).reversed())
        .limit(10)
        .collect(toList());
```

- 마지막 toList는 Collectors의 메서드다. 이처럼 Collectors의 멤버를 정적 임포트하여 쓰면 스트림 파이프라인 가독성이 좋아져, 흔히들 이렇게 사용한다.

---


???


## 추가 강의 내용 정리

## 🤔 생각 정리
- 궁금한 점, 논의하고 싶은 내용

