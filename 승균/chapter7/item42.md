## 익명 클래스보다는 람다를 사용하라

## **핵심정리**
예전 자바에서 함수 타입을 표현할 때 추상 메서드를 하나만 담은 인터페이스(드물게는 추상 클래스)를 사용했다. 이런 인터페이스의 인스턴스를 함수(function object)라고 하여, 특정 함수나 동작을 나타내는 데 썻다.

1997년 jdk 1.1이 등장하면서 함수 객체를 만드는 주요 수단은 익명 클래스(item24)가 되었다.

다음 코드를 예로 살펴보자. 

문자열을 길이순으로 정렬하는데, 정렬을 위한 비교 함수로 익명 클래스 사용한다.

**익명 클래스의 인스턴스를 함수 객체로 사용 - 낡은 기법이다!**

```java
import java.util.Collections;

Collections.sort(word, new Comparator<String>(){
    public int compare(String s1, String s2) {
        return Integer.compare(s1.length(), s2.length());
    }
});
```

전략 패턴처럼, 함수 객체를 사용하는 과거 객체 지향 디자인 패턴에는 익명 클래스면 충분했다.

이 코드에서 Comparator 인터페이스가 정렬을 담당하는 추상 전략을 뜻하며, 문자열을 정렬하는 구체적인 전략을 익명 클래스로 구현.

익명 클래스 방식은 코드가 너무 길기 때문에 자바는 함수형 프로그래밍에 적합하지 않았다.

자바 8에 와서 추상 메서드 하나짜리 인터페이스는 특별한 의미를 인정받아 특별한 대우를 받게 되었다. 

지금은 함수형 인터페이스라 부르는 이 인터페이스들의 인스턴스를 람다식(lambda expression)을 사용해 만들 수 있게 된 것이다.

람다는 함수나 익명 클래스를 사용한 앞의 코드를 람다 방식으로 바꾼 모습이다.

**람다식을 함수 객체로 사용 - 익명 클래스 대체**
```java
Collections.sort(words, (s1, s2) -> Integer.compare(s1.length(), s2.length()));
```
여기서, 람다 매개변수(s1, s2) 반환값의 타입은 각각 (Comparator<String>), String, int 지만 코드에서는 언급이 없다.

우리 대신 컴파일러가 문맥을 살펴 타입을 추론해준 것이다. 상황에 따라 컴파일러가 타입을 결정하지 못할 수도 있는데, 그럴 떄는 프로그래머가 직접 명시해야 한다.

**타입을 명시해야 코드가 더 명확할 때만 제외하고는, 람다의 모든 매개변수 타입은 생략하자**

람다 자리에 비교자 생성 메서드를 사용하면 이 코드를 더 간결하게 만들 수 있다.

```java
Collections.sort(words, comparingInt(String::length));
```

더 나아가 자바 8 때 List 인터페이스에 추가된 sort 메서드를 이용하면 더욱 짧아진다.

word.sort(comparingInt(String:length));

람다를 언어 차원에서 지원하면서 기존에는 적합하지 않았던 곳에서도 함수 객체를 실용적으로 사용할 수 있게 되었다. 

아이템 34의 Operation 열거 타입을 예로 들어보자. apply 메서드의 동작이 상수마다 달라야 해서 상수별 클래스 몸체를 사용해 각 상수에서 apply 메서드를 재정의한 것이 생각나는가?

**코드 42-3 상수별 클래스 몸체와 데이터를 사용한 열거 타입(코드 34-6)**
```java
public enum Operation {
    PLUS("+") {
        public double apply(double x, double y) { return x + y; }
    },
    MINUS("-") {
        public double apply(double x, double y) { return x + y; }
    },
    TIMES("*") {
        public double apply(double x, double y) { return x * y; }
    },
    DIVIDE("/") {
        public double apply(double x, double y) { return x / y; }
    };
    
    private final String symbol;
    
    Operation(String symbol) { this.symbol = symbol; }
    
    @Override public String toString() { return symbol; }
    public abstract double apply(double x, double y);
}
```

상수별 클래스 몸체를 구현하는 방식보다는 결거 타입에 인스턴스 필드를 두는 편이 낫다고 했다.

람다를 이용하면 상수별로 다르게 동작하는 코드를 쉽게 만들 수 있다.

생성자는 이 람다를 인스턴스 필드로 저장해둔다. 

**코드42-4 함수 객체(람다)를 인스턴스 필드에 저장해 상수별 동작을 구현한 열거 타입**

```java
import java.util.function.DoubleBinaryOperator;

public enum Operation {
    PLUS("+", (x, y) -> x + y),
    MINUS("-", (x, y) -> x - y),
    TIMES("*", (x, y) -> x * y),
    DIVIDE("/", (x, y) -> x / y);

    private final String symbol;
    private final DoubleBinaryOperator op;
    
    Operation(String symbol, DoubleBinaryOperator op) {
        this.symbol = symbol;
        this.op = op;
    }
    
    @Override public String toString() { return symbol; }
    
    public double apply(double x, double y) {
        return op.applyAsDouble(x, y);
    }
}
```

DoubleBinaryOperator는 java.util.function 패키지가 제공하는 다양한 함수 인터페이스중 하나로, double 타입 인수 2개를 받아 double 타입 결과를 돌려준다.

**람다는 이름이 없고 문서화도 못 한다. 따라서 코드 자체로 동작히 명확히 설명되지 않거나 코드 줄 수가 많아지면 람다를 쓰지 말아야함.**

람다는 한 줄 일 때 가장 좋고 길어야 세 줄 안에 끝내는 게 좋다. 세 줄을 넘어가면 가독성이 심하게 나빠짐

람다가 길거나 일기 어렵다면 더 간단히 줄여보거나 람다를 쓰지 않는 쪽으로 리팩터링

열거 타입 생성자에 넘겨지는 인수들의 타입도 컴파일타임에 추론된다. 따라서 열거 타입 생성자 안의 람다는 열거 타입의 인스턴스 멤버에 접근할 수 없다.

람다를 많이 쓰지만, 람다로 대체할 수 없는 곳도 많음

추상 클래스의 인스턴스를 만들 때, 람다를 쓸 수 없으니, 익명 클래스를 써야 한다. 

비슷하게 추상 메서드가 여러 개인 인터페이스의 인스턴스를 만들 떄도 익명 클래스를 쓸 수 있다. 마지막으로, 람다는 자신을 참조할 수 없다. 

람다에서의 this 키워드는 바깥 인스턴스를 가리킨다. 반면 익명 클래스에서의 this 키워드는 자기자신 인스턴스를 가리킨다.

그래서 함수 객체가 자신을 참조해야 한다면 반드시 익명 클래스를 써야 한다.

람다도 익명 클래스처럼 직렬화 형태가 구현별로 다를 수 있다. 따라서 **람다를 직렬화하는 일은 극히 삼가야 한다.**



