## 매개변수가 유효한지 검사하라

## **핵심정리**

메서드와 생성자 대부분은 입력 매개변수의 값이 특정 조건을 만족하기를 바란다. 예컨대 인덱스 값은 음수이면 안되며, 객체 참조는 null이 아니어야 하는 식이다. 

이런 제약은 반드시 문서화해야 하며 메서드 몸체가 시작되기 전에 검사해야 한다. 이런 제약은 반드시 문서화해야 하며 메서드 몸체가 시작되기 전에 검사해야 한다. 

이는 "오류는 가능한 한 빨리(발생한 곳에서) 잡아야 한다"는 일반 원칙의 한 사례이기도 하다. 오류는 발생한 즉시 잡지 못하면 해당 오류를 감지하기 어려워지고, 감지하더라도 오류의 발생 지점을 찾기 어려워진다.

메서드 몸체가 실행되기 전에 매개변수를 확인한다면 잘못된 값이 넘어왔을 때 즉각적이고 깔끔한 방식으로 예외를 던질 수 있다.

매개변수 검사를 제대로 하지 못하면 몇 가지 문제가 생길 수 있다.

첫 번째, 메서드가 잘 수행되지만 잘못된 결과를 반환할 때다. 한층 더 나쁜 상황은 메서드는 문제없이 수행됐지만, 어떤 객체를 이상한 상태로 만들어 놓아서 이래의 알 수 없는 시점에 이 메서드와는 관련 없는 오류를 낼 때다. 

public과 protected 메서드는 매개변수 값이 잘못됐을 때 던지는 예외를 문서화해야 한다. 보통은 IllegalArgumentException, IndexOutOfBoundsException, NullPointerException 중 하나가 될 것이다.

```java
/**
 * 항상 음이 아닌 BigInteger 반환한다는 점에서 remainder 메서드와 다르다. 
 *
 * @param m 계수 (양수여야 한다.)
 * @return 현재 값 mod m 
 * @throws java.lang.ArithmeticException m 이 0보다 작거나 같으면 발생한다.
 */

import java.math.BigInteger;

public BigInteger mod(BigInteger m) {
    if(m.signum() <= 0)
        throw new ArithmeticException("계수(m)는 양수여야 합니다.", e);
}
```

이 메서드는 m이 null이면 m.signum() 호출 때 NullPointerException 던진다.

그런데 "m이 null 일 때, NullPointerException을 던진다"라는 말을 메서드 설명 어디에도 없다.

그 이유는 이 설명을 BigInteger 클래스 수준에서 기술 했다.

클래스 수준 주석은 그 클래스의 모든 public 메서드에 적용되므로 각 메서드에 일일이 기술하는 것보다 훨씬 깔끔한 방법이다. 

@Nullable이나 이와 비슷한 애너테이션을 사용해 특정 매개변수는 null이 될 수 있다고 알려줄 수도 있지만, 표준적인 방법은 아니다. 

자바 7에 추가된 java.util.Objects.requireNonNull 메서드는 유연하고 사용하기도 편하니, 더 이상 null 검사를 수동으로 하지 않아도 된다. 

**코드 49-1 자바의 null 검사 기능 사용하기**
```java
this.strategy = Objects.requireNonNull(strategy, "전략");
```

반환값은 그냥 무시하고 필요한 곳 어디서든 순수한 null 검사 목적으로 사용해도 된다. 

자바 9에서는 Objects 범위 검사 기능도 더해졌다. checkFromIndexSize, checkFromToIndex, checkIndex라는 메서드들인데, null 검사 메서드만큼 유연하지는 않다. 

예외 메시지를 지정할 수 없고, 리스트와 배열 전용으로 설계됐다. 또한 닫힌 범위(closed range; 양 끝단 값을 포함하는)는 다루지 못한다. 

그래도 이런 제약이 걸림돌이 되지 않는 상황에서는 아주 유용하고 편하다.

공개되지 않은 메서드라면 패키지 제작자인 여러분이 메서드가 호출되는 상황을 통제할 수 있다. 따라서 오직 유효한 값만이 메서드에 넘겨지리라는 것을 여러분이 보증할 수 있고 그렇게 해야함. 

public이 아닌 메서드라면 assert를 사용해 매개변수 유효성을 검증할 수 있다.
**코드 49-2 재귀 정렬용 private 도우미 함수**
```java
private static void sort(long a[], int offset, int length) {
    assert a != null;
    assert offset >= 0 && offset <= a.length;
    assert length >= 0 && length <= a.length - offset;
    ... // 계산 수행
}
```

단언문은 몇 가지 면에서 일반적인 유효성 검사와 다르다. 

첫 번째, 실패하면 AssertionError 던진다. 

두 번째, 런타임에 아무런 효과도, 아무런 성능 저하도 없다(단, java 실행할 때 명령줄에서 -ea 혹은 --enableassertions 플래그 설정하면 런타임에 영향을 준다)

메서드가 직접 사용하지는 않으나 나중에 쓰기 위해 저장하는 매개변수는 특히 더 신경 써서 검사해야 한다. 

정적 팩터리 메서드를 생각해보라. 입력받은 int 배열의 List 뷰(view) 반환하는 메서드였다. 이 메서드는 Objects.requireNonNull 이용해 null 검사를 수행하므로 클라이언트가 null을 건네면 NullPointerException 던진다. 만약 이 검사를 생략했다면 새로 생성한 List 인스턴스를 반환

이때가 되면 이 List 어디서 가져왔는지 추적하기 어려워 디버깅이 상당히 괴로워질 수 있다. 

메서드 몸체 실행 전에 매개변수 유효성을 검사해야 한다는 규칙에도 예외는 있다. 

유효성 검사 비용이 지나치게 높거나 실용적이지 않을 때, 혹은 계산 과정에서 암묵적으로 검사가 수행될 떄다. 

예를 들어 Collections.sort(List)처럼 객체 리스를 정렬하는 메서드를 생각해보자. 리스트 안의 객체들은 모두 상호 비교될 수 있어야 하며, 정렬 과정에서 이 비교가 이뤄진다. 

매개변수에 제약을 두는게 좋지 않다. 사실은 반대임 최대한 제약이 적을 수록 좋음. 
