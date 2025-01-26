## 반환 타입으로는 스트림보다 컬렉션이 낫다.

## **핵심정리**

컬렉션 인터페이스는 for-each 문에서만 쓰이거나, 반환된 원소 시퀀스가 주로 contains(Object) 같은 일부 Collection 메서드를 구현할 수 없을 때는 Iterable 인터페이스를 썼다. 그런데 자바 8이 스트림이라는 개념을 들고 오면서 이 선택이 아주 복잡한 일이 되어버렸다. 

원소 시퀀스를 반환할 때는 당연히 스트림을 사용해야 한다는 이야기..?

스트림은 반복(iteration) 지원하지 않는다. 

따라서 스트림과 반복을 알맞게 조합해야 좋은 코드가 나온다.  API 스트림만 반환하도록 짜놓으면 반환된 스트림을 for-each반복하길 원하는 사용자는 당연히 불만을 토로한다. 

Stream 인터페이스는 Iterable 인터페이스가 정의한 방식대로 동작한다. 그럼에도 for-each 스트림을 반복할 수 없는 까닭은 바로 Stream이 Iterable 확장(extend)하지 않아서다. 

## 코드 47-1 자바 타입 추론의 한계로 컴파일 되지 않는다. 
```java
for(ProcessHandle ph : ProcessHandle.allProcesses()::iterator) {
    // 프로세스 처리한다.    
}
```

이 오류를 바로잡으려면 메소드 참조를 매개변수화된 Iterable로 적절히 형변환해줘야 한다. 

## 코드 47-2 스트림을 반복하기 위한 '끔찍한' 우회 방법
```java
for(ProcessHandle ph : (Iterable<ProcessHandle> ProcessHandle.allProcesses()::iterator)) {
    // 프로세스를 처리한다. 
}
```

작동은 하지만 실전에 쓰기에는 너무 난잡하고 직관성이 떨어진다. 다행히 어댑터 메서드를 사용하면 상황이 나아진다. 자바는 이런 메서드를 제공하지 않지만 다음 코드와 같이 쉽게 만들어낼 수 있다. 

## 코드 47-3 Stream<E>를 Iterable<E>로 중개해주는 어댑터
```java
public static <E> Iterable<E> iterableOf(Stream<E> stream) {
    return stream::iterator;
}
```

## 코드 47-4 Iterable<E>를 Stream<E>로 중개해주는 어댑터

```java
import java.util.stream.StreamSupport;

public static <E> Stream<E> streamOf(Iterable<E> iteralbe) {
    return StreamSupport.stream(iterable.spliterator(), false);
```

이 메서드가 오직 스트림 파이프라인에서만 사용한다면 안심하고 반환 

반대로 반환된 객체들이 반복문에서 쓰인다면 Iterable을 반환하자. 

공개된 API를 작성할 때는 스트림 파이프라인을 사용하는 사람과 반복문에서 쓰려는 사람 모두를 배려해야 한다. 사용자 대부분이 한 방식만 사용할 거라는 그럴싸한 근거가 없다면 말이다. 

**원소 시퀀스를 반환하는 공개 API의 반환 타입에는 Collection이나 그 하위 타입을 쓰는게 일반적으로 최선이다.**

Arrays 역시 Arrays.asList 와 Stream.of 메서드로 손쉽게 반복과 스트림을 지원 

**단지 컬렉션을 반환한다는 이유로 덩치 큰 시퀀스를 메모리에 올려서는 안 된다.**

반환할 시퀀스가 크지만 표현을 간결하게 할 수 있다면 전용 컬렉션을 구현하는 방안을 검토해보자.

## 코드 47-5 입력 집합의 멱집합을 전용 컬렉션을 담아 반환한다.

```java
import java.util.AbstractList;

public class PowerSet {
    public static final <E> Collection<Set<E>> of(Set<E> s) {
        List<E> src = new ArrayList<>();
        if (src.size() > 30)
            throw new IllegalArgumentException("집합에 원소가 너무 많습니다(최대 30개).: " + s);

        return new AbstractList<Set<E>>() {
            @Override public int size() {
                return 1 << src.size();
            }
            
            @Override public boolean contains(Object o) {
                return o instanceof Set && src.containsAll((Set)o);
            }
            
            @Override public Set<E> get(int index) {
                Set<E> result = new HashSet<>();
                for(int i = 0; index != 0; i++, index >>= 1)
                    if((index & 1) == 1) 
                        result.add(src.get(i));
                return result;
            }
        }
    }
}
```

## 코드 47-6 입력 리스트의 모든 부분리스트를 스트림으로 반환한다.

```java
import java.util.Collections;
import java.util.stream.IntStream;

public class SubLists {
    public static <E> Stream<List<E>> of(List<E> list) {
        return Stream.concat(Stream.of(Collections.emptyList())),prefixes(list).flatMap(SubLists::suffixes);
    }

    private static <E> Stream<List<E>> prefixes(List<E> list) {
        return IntStream.rangeClosed(1, list.size())
                .mapToObj(end -> list.subList(0, end));
    }
    
    private static <E> Stream<List<E>> suffixes(List<E> list) {
        return IntStream.range(0, list.size())
                .mapToObj(start -> list.subList(start, list.size()));
    }
}
```

Stream.concat 메서드는 반환되는 스트림에 빈 리스트를 추가하며, flatMap 메서드(아이템 45)는 모든 프리픽스의 모든 서픽스로 구성된 하나의 스트림을 만든다. 

마지막으로 프리픽스들과 서픽스들의 스트림은 IntStream.range와 IntStream.rangeClosed 반환하는 연속된 정수값들을 매핑해 만들었다.

for(int start = 0; start < src.size(); start++)
    for(int end = start + 1; end <= src.size(); end++)
        System.out.println(src.subList(start, end));

위에 반복문은 그대로 스트림으로 변환

## 코드 47-7 입력 리스트의 모든 부분리스트를 스트림으로 반환한다.

```java
import java.util.stream.IntStream;

public static <E> Stream<List<E>> of(List<E> list) {
    return IntStream.range(0, list.size())
            .mapToObj(start -> IntStream.rangeClosed(start + 1, list.size())
                    .mapToObj(end -> list.subList(start, end)))
            .flatMap(x -> x);
}
```

바로 앞의 for 반복문처럼 이 코드도 빈 리스트는 반환하지 않는다. 이 부분을 고치려면 concat 사용하거나 rangeClosed 호출 코드의 1을 (int) Math.signum(start)로 고쳐주면 된다. 