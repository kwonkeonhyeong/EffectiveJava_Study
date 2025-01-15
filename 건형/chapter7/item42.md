## 익명 클래스보다는 람다를 사용하라 

### 핵심 내용

자바 8부터 작은 함수 객체를 구현하는 데 적합한 람다가 도입되었다.

익명 클래스는 (함수형 인터페이스가 아닌) 타입의 인스턴스를 만들 때만 사용하라.

람다는 작은 함수 객체를 아주 쉽게 표현할 수 있어 (이전 자바에서는 실용적이지 않던) 함수형 프로그래밍의 지평을 열었다.


## 💡 주요 내용 정리 & 🛠️ 실습 코드

예전에 자바에서 함수 타입을 표현할 때 추상 메서드를 하나만 담은 인터페이스(드물게는 추상 클래스)를 사용했다.

이런 인터페이스의 인스턴스를 함수 객체라고 하여, 특정 함수나 동작을 나타내는 데 썼다.

이후 JDK 1.1 등장부터 함수 객체를 만드는 주요 수단은 익명 클래스가 되었다.

**익명 클래스의 인스턴스를 함수 객체로 사용 - 낡은 기법!**
```java
public static void main(String[] args) {
    Collections.sort(words, new Comparator<String>() {
        public int compare(String s1, String s2) {
            return Integer.compare(s1.length(), s2.length());
        }
    });
}
```

위 코드에서 Comparator 인터페이스가 정렬을 담당하는 추상 전략을 뜻하며, 문자열을 정렬하는 구체적인 전략을 익명 클래스로 구현했다.

하지만 익명 클래스 방식은 코드가 너무 길기 때문에 자바는 함수형 프로그래맹에 적합하지 않았다.

---

java 8에 와서는 함수형 인터페이스가 부르는 인터페이스들의 인스턴스를 람다식 (lamda expression)을 사용해 만들 수 있게 됐다.

람다는 함수나 익명 클래스와 개념은 비슷하지만 코드는 훨씬 간결하다.

**람다식을 함수 객체로 사용 - 익명 클래스 대체**
```java
public static void main(String[] args) {
    Collections.sort(words,
            (s1, s2) -> Integer.compare(s1.length(), s2.length()));

    Collections.sort(words, comparingInt(String::length));

    words.sort(comparingInt(String::length));
}
```

**타입을 명시해야 코드가 더 명확할 때만 제외하고는, 람다의 모든 매개변수 타입은 생략하자**

컴파일러가 "타입을 알 수 없다"는 오류를 낼때만 해당 타입을 명시하면 된다!

> 컴파일러가 타입을 추론하는 데 필요한 타입 정보 대부분을 제네릭에서 얻기 때문에 아래와 같은 조언이 중요해진다.
> 
> 1. 제네릭의 로 타입을 쓰지 말라
> 2. 제네릭을 쓰라
> 3. 제네릭 메서드를 쓰라
> 
> 컴파일러가 타입을 추론하는 데 필요한 타입 정보를 제공하지 않으면 컴파일러는 람다의 타입을 추혼할 수 없게 되어, 일일이 명시해야 한다.(코드가 너저분해진다.)
> 
> 좋은 예로 위 예시 코드의 words가 매개변수화 타입인 List<String>이 아니라 로 타입인 List였다면 컴파일 오류가 났을 것이다.

---

람다를 언어 차원에서 지원하면서 기존에는 적합하지 않았던 곳에서도 함수 객체를 실용적으로 사용할 수 있게 되었다.

```java
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
        return op.applyAsDouble(w,y);
    }
}

// DoubleBinaryOperator는 double 타입 인수 2개를 받아 double 타입 결과를 돌려준다.
```

---

### 람다를 사용하지 말아야 하는 상황!

1. **람다는 이름이 없고 문서화도 못 한다. 따라서 코드 자체로 동작이 명확히 설명되지 않거나 코드 줄 수가 많아지면 람다를 쓰지 말아야 한다.**

    - 람다는 한 줄일 때 가장 좋고 길어야 세 줄 안에 끝내는 게 좋다.

    - 람다가 길거나 읽기 어렵다면 더 간단히 줄여보거나 람다를 쓰지 않는 쪽으로 리팩터링하자.

2. **열거 타입 생성자에 넘겨지는 인수들의 타입도 컴파일타임에 추론된다.**
    
    - 열거 타입 생성자 안의 람다는 열서 타입의 인스턴스 멤버에 접근할 수 없다.( 인스턴스는 런타임에 만들어지기 때문이다.)
    - 따라서 상수별 동작을 단 몇 줄로 구현하기 어렵거나, 인스턴스 필드나 메서드를 사용해야만 하는 상황이라면 상수별 클래스 몸체를 사용해야 한다.
   
    
---

## 추가 강의 내용 정리

### 자바 8 그리고 람다.

람다는 함수형 인터페이스를 지향하고자 하였다

타입을 명시해야 코드가 더 명확할 떄를 제외하고는 람다의 모든 매개변수 타입은 생략한다.

즉 컴파일러가 타입을 알 수 없다고 에러를 낼 때를 제외하고는 항상 생략한다.

> 순수함수의 내부에서 순수함수 밖의 변수에 영향을 주게 되면 순수 함수로 인한 사이트 이펙트가 발생한다.
> 
> 순수하게 함수를 짜게 되면 그 내부에서만 소통이 되기 때문에 스레드 세이프하고 병렬 계산이 가능해진다.

---

### 익명 함수를 람다로 변환하는 모습

익명 함수
```java
public static void main(String[] args) {
   List<String> words = List.of("사과","배");
   Collections.sort(words, new Comparator<>() {
       public int compare(String o1, String o2) {
          return Integer.compare(o1.length(), o2.length());
       }
   });
}
```

람다
```java
public static void main(String[] args) {
   Collections.sort(words, Comparator.comparingInt(s -> s.length())); // 람다
   Collections.sort(words, Comparator.comparingInt(String::length)); // 메서드 참조
   words.sort(Comparator.comparingInt(String::length)); // 메서드 참조
}
```

### 정리

익명 클래스 보다는 람다를 사용하라

람다를 사용할 수 있는 자바 버전이면, 타입 추론 또한 가능하기 때문에 매개변수 타입은 가능하면 생략한다.

익명 함수는 이제 (함수형 인터페이스가 아닌) 타입으 인스턴스를 만들 때만 사용하라.

---

## 🤔 생각 정리
- 람다를 사용하게 되면서 익명 클래스를 사용하지 않게 되었고
- 확실히 타입을 자동으로 추론해주다 보니 코드를 작성하는 입장에서는 매우 편리하다고 생각했습니다.

