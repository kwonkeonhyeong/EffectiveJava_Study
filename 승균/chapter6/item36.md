## 비트 필드 대신 EnumSet을 사용하라

## **핵심정리**

열거한 값들이 주로 (단독이 아닌) 집합으로 사용될 경우, 예전에는 각 상수에 서로 다른 2의 거듭제곱 값을 할당한 정수 열거 패턴

**코드36-1 비트 필드 열거 상수 - 구닥다리 기법!**
```java
public class Text {
    public static final int STYLE_BOLD = 1 << 0; // 1
    public static final int STYLE_ITALIC = 1 << 1; // 2
    public static final int STYLE_UNDERLINE = 1 << 2;
    public static final int STYLE_STRIKETHROUGH = 1 << 3;
    
    // 매개변수 styles는 0개 이상의 STYLE_ 상수를 비트별 OR한 값이다.
    public void applyStyles(int styles) { ... }
}
```

다음과 같은 식으로 비트별 OR 사용해 여러 상수를 하나의 집합으로 모을 수 있으며, 이렇게 만들어진 집합을 비트 필드(bit field)라 한다. 

```java
text.applyStyles(STYLE_BOLD | STYLE_ITALIC);
```

비트 필드를 사용하면 비트별 연산을 사용해 합집합과 교집합 같은 집합 연산을 효율적으로 수행할 수 있다. 

하지만 비트 필드는 정수 열거 상수의 단점을 그대로 지니며, 추가로 다음과 같은 문제까지 안고 있다. 

비트 필드 값이 그대로 출력되면 단순한 정수 열거 상수를 출력할 때보다 해석하기가 훨씬 어렵다. 

정수 상수보다 열거 타입을 선호하는 프로그래머 중에서도 상수 집합을 주고 받아야 할 때는 여전히 비트 필드를 사용하기도 한다. 

하지만 더 나은 대안이 있음. 

java.util 패키지의 EnumSet 클래스는 열거 타입 상수의 값으로 구성된 집합을 효과적으로 표현해준다. 

Set 인터페이스를 완벽히 구현하며, 타입 안전하고, 다른 어떤 Set 구현체와도 함께 사용할 수 있다. 

하지만 내부는 비트 벡터로 구현되었다. 원소가 총 64개 이하라면, 즉 대부분의 경우에 EnumSet 전체를 long 변수 하나로 표현하여 비트 필드에 비견되는 성능을 보여준다.

removeAll 과 retainAll 같은 대량 작업은 (비트 필드를 사용할 때 쓰는 것과 같은) 비트를 효율적으로 처리할 수 있는 산술 연산을 써서 구현했다.

**코드 36-2 비트 필드를 대체하는 현대적 기법**
```java
public class Text {
    public enum Style { BOLD, ITALIC, UNDERLINE, STRIKETHROUGH }
    
    // 어떤 Set 넘겨도 되나, EnumSet이 가장 좋음
    public void applyStyles(Set<Style> styles) { ... }
}
```

test.applyStyles(EnumSet.of(Style.BOLD, Style.ITALIC));

applyStyles 메서드가 EnumSet<Style>이 아닌 Set<Style>을 받은 이유를 생각해보자.

모든 클라이언트가 EnumSet을 건네리라 짐작되는 상황이라도 이왕이면 인터페이스로 받는 게 일반적으로 좋은 습관이다.
