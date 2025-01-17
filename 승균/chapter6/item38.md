## 확장할 수 있는 열거 타입이 필요하면 인터페이스를 사용하라

## **핵심정리**
타입 안전 열거 패턴은 확장할 수 있으나 열거 타입은 그럴 수 없다는 점

대부분 상황에서 열거 타입 확장 좋지 못한 생각이다.

확장한 타입의 원소는 기반 타입의 원소로 취급하지만, 그 반대는 성립하지 않으면 리스코프 치환법칙에 어긋남.

확장할 수 있는 열거 타입이 어울리는 쓰임이 최소한 하나는 있음.
연산 코드 (Operation code 혹은 opcode)다.

API가 제공하는 기본 연산외 사용자 확장 연산을 추가할 수 있도록 열워줘야 함.

**코드 38-1 인터페이스를 이용해 확장 가능 열거 타입을 흉내 냈다.**
```java
public interface Operation {
    double apply(double x, double y);
}

public enum BasicOperation implements Operation {
    PLUS("+") {
        public double apply(double x, double y) { return x + y; }
    }, 
    MINUS("-") {
        public double apply(double x, double y) { return x - y; }
    },
    TIMES("*") {
        public double apply(double x, double y) { return x * y; }
    },
    DIVIDE("/") {
        public double apply(double x, double y) { return x / y; }
    }
    
    private final String symbol;
    
    BasicOperation(String symbol) {
        this.symbol = symbol;
    }
    
    @Override public String toString() {
        return symbol;
    }
}
```

열거 타입인 BasicOperation은 확장할 수 없지만 인터페이스인 Operation 확장할 수 있고, 이 인터페이스를 연산의 타입으로 사용하면 된다.

이렇게 하면 Operation 구현한 또 다른 열거 타입을 정의해 기본 타입인 BasicOperation을 대체 할 수 있다.

지수 연산과 나머지 연산 추가 예시)
**코드 38-2 확장 가능 열거 타입**
```java
public enum ExtendedOperation implements Operation {
    EXP("^") {
        public double apply(double x, double y) {
            return Math.pow(x, y);
        }
    },
    REMAINDER("%") {
        public double apply(double x, double y) {
            return x % y;
        }
    };
    
    private final String symbol;
    
    ExtendedOperation(String symbol) {
        this.symbol = symbol;
    }
    
    @Override public String toString() {
        return symbol;
    }
}
```

새로 작성한 연산은 기존 연산을 쓰던 곳이면 어디든 쓸 수 있다.

(Basic Operation 아닌) Operation 인터페이스를 사용하도록 작성되어 있기만 하면 된다.

apply가 인터페이스(Operation) 선언되어 있으니 열거 타입에 따로 추상 메서드로 선언하지 않아도 된다.

코드 34-5 상수별 메서드 구현과 다른점

```java
public static void main(String[] args) {
    double x = Double.parseDouble(args[0]);
    double y = Double.parseDouble(args[1]);
    test(ExtendedOperation.class, x, y);
}

private static <T extends Enum<T> & Operation> void test(Class<T> opEnumType, double x, double y) {
    for (Operation op : opEnumType.getEnumConstans())
        System.out.printf("%f %s %f = %f%n", x, op, y, op.apply(x, y));
}
```

main 메서드는 test 메서드에 ExtendedOperation의 class 리터럴을 넘겨 확장된 연산들이 무엇인지 알려준다.

여기서 class 리터럴은 한정적 타입 토큰 역할을 한다.

opEnumType 매개변수의 선언(<T extends Enum<T> & Operation> Class<T>) 는 열거 타입인 동시에 Operation 하위 타입이어야 한다는 의미이다.

두 번째 대안은 class 객체 대신 한정적 와일드카드 타입 Collection<? extends Operation> 넘기는 방법이다.
```java
public static void main(String[] args) {
    double x = Double.parseDouble(args[0]);
    double y = Double.parseDouble(args[1]);
    
    test(Arrays.asList(ExtendedOperation.values()), x, y);
}

private static void test(Collection<? extends Operation> opSet, double x, double y) {
    for(Operation op : opSet) {
        System.out.printf("%f %s %f = %f%n", x, op, y, op.apply(x, y));
    }
}
```

덜 복잡하고 test 메서드가 조금 더 유연해짐
특정 연산에서 EnumSet, Enum Map 사용 불가능

4.000000 ^ 2.000000 = 16.000000

4.000000 % 2.000000 = 0.000000

인터페이스를 이용해 확장 가능한 열거 타입을 흉내 내는 방식에도 한 가지 사소한 문제가 있다.

바로 열거 타입끼리 구현을 상속할 수 없는 점

아무 상태에도 의존하지 않는 경우에만 디폴트 구현을 이용해 인터페이스에 추가하는 방법이 있다.

Operation  예는 연산 기호를 저장하고 찾는 로직이 BasicOperation 과 ExtendedOperation 모두에 들어가야만 한다.

이 경우에는 중복량이 적으니 문제되진 않지만, 공유하는 기능이 많다면 그 부분을 별도의 도우미 클래스나 정적 도우미 메서드로 분리하는 방식으로 코드 중복을 없앨 수 있다.


