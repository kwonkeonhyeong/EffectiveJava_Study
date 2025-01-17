## 📖 반환 타입으로는 스트림보다는 컬렉션이 낫다.

### 핵심 내용

원소 시퀀스를 반환하는 메서드를 작성할 때는, 

이를 스트림으로 처리하기를 원하는 사용자와 반복으로 처리하길 원하는 사용자가 모두 있을 수 있음을 떠올리고,

양쪽을 다 만족시키려 노력하자.

컬렉션을 반환할 수 있다면 그렇게 하라.

반환 전부터 이미 원소들을 컬렉션에 담아 관리하고 있거나 컬렉션을 하나 더 만들어도 될 정도로 원소 개수가 적다면 ArrayList 같은 표준 컬렉션에 담아 반환하라.

그렇지 않으면 앞서의 멱집합 예처럼 전용 컬렉션을 구현할지 고민하라.

컬렉션을 반환하는 게 불가능하면 스트림과 Iterable 중 더 자연스러운 것을 반환하라.

만약 나중에 Stream 인터페이스가 Iterable을 지원하도록 자바가 수정된다면, 그때는 안심하고 스트림을 반환하면 될 것이다. (스트림 처리와 반복 모두에 사용할 수 있으니)


## 💡 주요 내용 정리 & 🛠️ 실습 코드

어탭터 메서드를 사용하면 자바의 타입 추론이 문맥을 잘 파악하여 어댑터 메서드 안에서 따로 형변환하지 않아도 된다.

Stream<E>를 Iterable<E>로 중개해주는 어댑터
```java
public static <E> Iterable<E> iterableOf(Stream<E> stream) {
    return stream::iteraor;
}

public static void main(String[] args) {
    // 어댑터를 사용하면 어떤 스트림도 for-each 문으로 반복할 수 있다.
    for (ProcessHandle p : iterableOf(ProcessHandle.allProcesses())) {
        // 프로세스를 처리한다.
    }
}
```

Iterable<E>를 Stream<E>로 중개해주는 어댑터
```java
public static <E> Stream<E> streamOf(Iterable<E> iterable) {
    return StreamSupport.stream(iterable.spliterator(), false);
}
```

---

객체 시퀀스를 반환하는 메서드를 작성하는데, 이 메서드가 오직 스트림 파이프라인에서만 쓰일 걸 안다면 마음 놓고 스트림을 반환하게 해주자.

반대로 반환된 객체들이 반복문에서만 쓰일 걸 안다면 Iterable을 반환하자.

하지만 공개 API를 작성할 때는 스트림 파이프라인을 사용하는 사람과 반복문에서 쓰여는 사람 모두를 배려해야 한다.

---

원소 시퀀스를 반환하는 공개 API 반환 타입에는 Collection이나 그 하위 타입이고 stream 메서드도 제공하니 반복과 스트림을 동시에 지원한다.

원소 시퀀스를 반환하는 공개 API의 반환 타입에는 Collection이나 그 하위 타입을 쓰는게 일반적으로 최선이다.

단지 컬렉션을 반환한다는 이유로 덩치 큰 시퀀스를 메모리에 올려서는 안 된다.


## 추가 강의 내용 정리

## 🤔 생각 정리
- 궁금한 점, 논의하고 싶은 내용

