## 📖 스트림 병렬화는 주의해서 적용하라

### 핵심 내용

계산도 올바로 수행하고 성능도 빨라질 거라는 확신 없이는 스트림 파이프라인 병렬화는 시도조차 하지 말라.

스트림을 잘못 병렬화하면 프로그램을 오동작하게 하거나 성능을 급격히 떨어뜨린다.

병렬화하는 편이 낫다고 믿더라도, 수정 후의 코드가 여전히 정확한지 확인하고 운영 환경과 유사한 조건에서 수행해보며 성능지표를 유심히 관찰하라.

그래서 계산도 정확하고 성능도 좋아졌음이 확실해졌을 때, 오직 그럴때만 병렬화 버전 코드를 운영 코드에 반영하라.


## 💡 주요 내용 정리 & 🛠️ 실습 코드

동시성 프로그래밍을 할 때는 안전성과 응답 가능 상태를 유지하기 위해 애써야 하는데, 병렬 스트림 파이프라인 프로그래밍에서도 다를 바 없다.

환경이 아무리 좋더라도 데이터 소스가 Stream.iterate거나 중간 연산으로 limit를 쓰면 파이프라인 병렬화로는 성능 개선을 기대할 수 없다.

---

**대체로 스트림의 소스가 ArrayList, HashMap, HashSet, ConcurrentHashMap의 인스턴스거나 배열, int 배열, long 범위일 때 병렬화의 효과가 가장 좋다.**

이 자료구조들은 모두 데이터를 원하는 크기로 정확하고 손쉽게 나눌 수 있어서 일을 다수의 스레드에 분배하기에 좋다는 특징이 있다.

나누는 작업은 Spliterator가 담당하며, Spliterator 객체는 Stream 이나 Iterable의 spliterator 메서드로 얻어올 수 있다.

위 자료구조들의 중요한 공통점은 원소들을 순차적으로 실행할 때 참조 지역성이 뛰어나다는 것이다.

이웃한 원소의 참조들이 메모리에 연속해서 저장되어 있다는 뜻이다.

하지만 참조들이 가리키는 실제 객체가 메모리에서 서로 떨어져 있을 수 있는데, 그러면 참조 지역성이 나빠진다.

참조 지역성이 낮으면 스레드는 데이터가 주 메모리에서 캐시 메모리로 전송되어 오기를 기다리며 대부분 시간을 멍하니 보내게 된다.

따라서 참조 지역성은 다량의 데이터를 처리하는 벌크 연산을 병렬화할 때 아주 중요한 요소로 작용한다.

참조 지역성이 가장 뛰어난 자료구조는 기본 타입의 배열이다.

기본 타입 배열에서는 데이터 자테가 메모리에 연속해서 저장되기 때문이다.

---

**스트림 파이프라인의 종단 연산의 동작 방식 역시 병렬 수행 효율에 영향을 준다.**

종단 연산에서 수행하는 작업량이 파이프라인 전체 작업에서 상당 비중을 차지하면서 순차적인 연산이라면 파이프라인 병럴 수행의 효과는 제한될 수 밖에 없다.

종단 연산 중 병렬화에 가장 적합한 것은 축소다.

축소는 파이프라인에서 만들어진 모든 원소를 하나로 합치는 작업이다.

Stream의 reduce 메서드 중 하나, 혹은 min,max,count,sum 같이 완성된 형태로 제공되는 메서드 중 하나를 선택해 수행한다.

anyMatch, allMatch, noneMatch 처럼 조건에 맞으면 바로 반환되는 메서드도 병렬화에 적합니다.

반면, 가변 축소를 수행하는 Stream의 collect 메서드는 병렬화에 적합하지 않다. 

컬렉션들을 합치는 부담이 크기 때문이다.

---

직접 구현한 Stream, Iterable, Collection이 병렬화의 이점을 제대로 누리게 하고 싶다면 spliterator 메서드를 반드시 재정의하고 결과 스트림의 변렬화 성능을 강도 높게 테스트하라.

**스트림을 잘못 병렬화하면 (응답 불가를 포함해) 성능이 나빠질 뿐만 아니라 결과 가체가 잘못되거나 예상 못한 동작이 발생할 수 있다.**

보통은 병렬 스트림 파이프라인도 공통의 포크-조인 풀에서 수행되므로(즉, 같은 스레드 풀을 사용하므로), 잘못된 파이프라인 하나가 시스템의 다른 부분의 성능에까지 악영향을 줄 수 있음을 유념하자.



## 추가 강의 내용 정리

## 🤔 생각 정리
- 궁금한 점, 논의하고 싶은 내용

