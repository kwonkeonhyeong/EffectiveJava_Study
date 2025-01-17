## int 상수 대신 열거 타입을 사용하라

## **핵심정리**

열거 타입은 일정 개수의 상수 값을 정의한 다음, 그 외의 값은 허용하지 않는 타입이다.

**34-1 정수 열거 패턴 - 상당히 취약하다**

```java
public static final int APPLE_FUJI = 0;
public static final int APPLE_PIPPIN = 1;
public static final int APPLE_GRANNY_SMITH = 2;

public static final int ORANGE_NAVEL = 0;
public static final int ORANGE_TEMPLE = 1;
public static final int ORANGE_BLOOD = 2;
```

정수 열거 패턴(int enum pattern) 기업에는 단점이 많다.

타입 안전은 보장할 방법이 없으며 표현력도 좋지 않다.

오렌지를 건네야 할 메서드에 사과를 보내고 동등 연산자(==)로 비교하더라도 컴파일러는 아무런 경고 메시지를 출력하지 않는다.

int i = (APPLE_FUJI - ORANGE_TEMPLE) / APPLE_PIPPIN;

사과용 상수의 이름은 모두 APPLE_로 시작하고, 오렌지용 상수는 ORANGE_로 시작한다.

자바가 정수 열거 패턴을 위한 별도 **이름공간(namespace)** 지원하지 않기 때문에 어쩔 수 없이 접두어를 써서 이름 충돌을 방지.

정수 열거 패턴을 사용한 프로그램은 깨지기 쉽다.

평범한 상수를 나열한 것뿐이라 컴파일하면 그 값이 클라이언트 파일에 그대로 새겨진다.

컴파일이 반드시 이루어져야하며, 다시 컴파일 되지 않으면 엉뚱한 동작을 한다.

정수 대신 문자열 상수를 사용하는 변형 패턴도 있다.

문자열 열거 패턴(string enum pattern)이라 하는 변형은 더 나쁘다.

상수의 의미를 출력할 수 있다는 점은 좋지만, 경험이 부족한 프로그래머가 문자열 상수의 이름 대신 문자열 값을 그대로 하드코딩하게 만들기 때문이다.

이렇게 하드 코딩한 문자열에 오타가 있어도 컴파일러는 확인할 길이 없으니 자연스럽게 런타임 버그가 생긴다.

다행히 자바는 열거 패턴의 단점을 말끔히 씻어주는 동시에 여러 장점을 안겨주는 열거 타입이라는 것이 있다.


**가장 단순한 열거 타입**
```java
public enum Apple { FUJI, PIPPIN, GRANNY_SMITH }
public enum Orange { NAVEL, TEMPLE, BLOOD }
```

열거 타입 자체는 클래스이며, 상수 하나당 자신의 인스턴스를 하나씩 만들어 public static final 필드로 공개

열거 타입은 밖에서 접근 할 수 있는 생성자 생성 x 사실상 final 

따라서 클라이언트가 인스턴스를 직접 생성하거나 확장 할 수 없으니 열거 타입 선언으로 만들어진 인스턴스들은 딱 하나씩만 존재함이 보장된다.

싱글턴은 원소가 하나뿐인 열거 타입이라 할 수 있고, 거꾸로 열거 타입은 싱글턴을 일반화한 형태라고 볼 수 있다.

**열거 타입은 컴파일 타임 타입 안정성을 제공**

열거 타입은 값을 바꾸거나 새로운 값을 추가하여도 다시 컴파일 x 

열거 타입은 toString으로 출력하기 좋은 문자열 제공

열거 타입은 임의의 메서드나 필드 추가할 수 있고, 임의의 인터페이스를 구현 할 수 있음.

**데이터와 메서드를 갖는 열거 타입**
```java
public enum Planet {
    MERCURY(3.302e+23, 2.439e6),
    VENUS(4.869e+24, 6.052e6),
    EARTH(5.975e+24, 6.378e6),
    MARS(6.419e+23, 3.393e6),
    JUPITER(1.899e+27, 7.149e7),
    SATURN(5.684e+26, 6.027e7),
    URANUS(8.683e+25, 2.556e7),
    NEPTUNE(1.024e+26, 2.477e7);
    
    private final double mass; // 질량
    private final double radius; // 반지름
    private final double surfaceGravity; // 표면 장력
    
    // 중력 상수
    private static final double G = 6.67300E-11;
    
    Planet(double mass, double radius) {
        this.mass = mass;
        this.radius = radius;
        surfaceGravity = G * mass / (radius * radius);
    }
    
    public double mass() { return mass; }
    public double radius() { return radius; }
    public double surfaceGravity() { return surfaceGravity; }
    
    public double surfaceWeight(double mass) {
        return mass * surfaceGravity; // F = ma
    }
}
```
열거 타입 상수 각각을 특정 데이터와 연결지으려면 생성자에서 데이터를 받아 인스턴스 필드에 저장하면 된다.

필드를 private로 두고 public 접근자 메서드를 만드는게 좋다.

Planet 생성자에서 표면중력을 계산해 저장한 이유는 단순히 최적화를 위해서이다.

```java
public class WeightTable {
    public static void main(String[] args) {
        double earthWeight = Double.parseDouble(args[0]);
        double mass = earthWeight / Planet.EARTH.surfaceGravity();
        for(Planet p : Planet.values())
            System.out.printf("%s에서의 무게는 %f이다. %n", p, p.surfaceWeight(mass));
    }
}
```

열거 타입은 자신 안에 정의된 상수들의 값을 배열에 담아 반환하는 정적 메서드인 values 제공한다. 

각 열거 타입 값의 toString 메서드는 상수 이름을 문자열로 반환하므로 println과 printf로 출력하기에 안성 맞춤이다.

패키지에서만 유용한 기능은 private이나 package-private 메서드로 구현한다.

그 기능을 클라이언트에 노출해야 할 합당한 이유가 없다면 private, 혹은 (필요하다면) package-private로 선언하라

널리 쓰이는 열거 타입은 톱레벨 클래스로 만들고, 특정 톱레벨 클래스에서만 쓰인다면 해당 클래스의 멤버 클래스로 만든다.

예를 들어 소수 자릿수의 반올림 모드를 뜻하는 열거 타입인 Java.math.RoundingMode는 BigDecimal 사용한다.

그런데 반올림 모드는 BigDecimal과 관련 없는 영역에서도 유용한 개념이라 자바 라이브러리 설계자는 RoundingMode를 톱레벨로 올렸다. 

이 개념을 많은 곳에서 사용하여 다양한 API가 더 일관된 모습을 갖출 수 있도록 장려한 것이다. 


**코드 34-4 값에따라 분기하는 열거 타입 - 이대로 만족하는가?**

```java
public enum Operation {
    PLUS, MINUS, TIMES, DIVIDE;
    
    public double apply(double x, double y) {
        switch(this) {
            case PLUS: return x + y;
            case MINUS: return x - y;
            case TIMES: return x * y;
            case DIVIDE: return x / y;
        }
        throw new AssertionError("알 수 없는 연산: " + this);
    }
}
```

동작은 하지만 그리 예쁘지 않다. 마지막의 throw 문은 실제로는 도달할 일이 없지만 기술적으로는 도달할 수 있기 생략하면 컴파일조차 되지 않는다. 

더 나쁜 점은 깨지기 쉬운 코드라는 사실이다. 

예컨대 새로운 상수를 추가하면 해당 case문 추가해야 한다. 

*"알 수 없는 연산"*이라는 런타임 오류를 내며 프로그램이 종료된다. 

다행이 열거 타입은 상수별로 다르게 동작하는 코드를 구현

열거 타입에 apply라는 추상 메서드를 선언하고 각 상수별 클래스 몸체(constant-specific class body), 즉 각 상수에서 자신에 맞게 재정의하는 방법이다.

**상수별 메서드 구현을 활용한 열거 타입**
```java
public enum Operation {
    PLUS { public double apply(double x, double y) { return x + y;}}, 
    MINUS { public double apply(double x, double y) { return x- y;}}, 
    TIMES { public double apply(double x, double y) { return x * y;}},
    DIVIDE { public double apply(double x, double y) { return x / y;}};
    
    public abstract double apply(double x, double y);
}
```

apply 메서드가 상수 선언 바로 옆에 붙어 있으니 새로운 상수를 추가 할 때 apply 재정의해야 한다는 사실을 깜빡하기도 어려울 것이다. 

그뿐만 아니라 apply가 추상 메서드이므로 재정의하지 않았다면 컴파일 오류로 알려준다. 

상수별 메서드 구현을 상수별 데이터와 결합할 수도 있다. 

**코드34-6 상수별 클래스 몸체(class body)와 데이터를 사용한 열거 타입**
```java
public enum Operation {
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
    };
    
    private final String symbol;
    
    Operation(String symbol) { this.symbol = symbol; }
    
    @Override public String toString() { return symbol; }
    
    public abstract double apply(double x, double y);
}
```

```java
public static void main(String[] args) {
    double x = Double.parseDouble(args[0]);
    double y = Double.parseDouble(args[1]);
    for ( Operation op : Operation.values()) {
        System.out.printf("%f %s %f = %f%n", x, op, y, op.apply(x, y));
    }
}
```

열거 타입에는 상수 이름을 입력받아 그 이름에 해당하는 상수를 반환해주는 valueOf(String) 메서드가 자동으로 생성된다. 
열거 타입의 toString()메서드를 정의하려 한다면, toString 반환하는 문자열을 해당 열거 타입 상수로 변환해주는 fromString 메서드도 함께 제공하는 걸 고려해보자. 

**34-7 열거 타입용 fromString메서드 구현하기**
```java
private static final Map<String, Operation> stringToEnum = Stream.of(values()).collect(toMap(Object::toString, e -> e));

// 저장한 문자열에 해당하는 Operation을 반환한다. 
public static Optional<Operation> fromString(String symbol) {
    return Optional.ofNullable(stringToEnum.get(symbol));
}
```

Operation 상수가 stringToEnum 맵에 추가되는 시점은 열거 타입 상수 생성 후 정적 필드가 초기화될 때다. 

앞의 코드는 values 메서드가 반환하는 배열 대신 스트림 참조 

열거 타입의 정적 필드 중 열거 타입의 생성자에 접근 할 수 있는 것은 상수 변수뿐이다.

Optional<Operation>은 주어진 문자열이 가리키는 연산이 존재하지 않을 수도 있음을 클라이언트에 알림. 

그 상황을 클라이언트가 대처하도록 함. 

한편 상수별 메서드 구현에는 열거 타입 상수끼리 코드를 공유하기 어렵다는 단점이 있다. 

급여명세서에서 쓸 요일을 표현하는 열거 타입을 예로 생각해보자. 

**34-8 값에 따라 분기하여 코드를 공유하는 열거 타입 - 좋은 방법인가?**
```java
enum PayrollDay {
    MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY, SATURDAY, SUNDAY;
    
    private static final int MINS_PER_SHIFT = 8 * 60;
    
    int pay(int minutesWorked, int payRate) {
        int basePay = minutesWorked * payRate;
        
        int overtimePay;
        
        switch (this){
            case SATURDAY: case SUNDAY: //주말
              overtimePay = basePay / 2;
              break;
            default: // 주중
                overtimePay = minutesWorked <= MINS_PER_SHIFT ? 0 : (minutesWorked - MINS_PER_SHIFT) * payRate / 2;
        }
        
        return basePay + overtimePay;
    }
}
```

해당 코드는 위험한 코드이다 왜냐면 새로운 값이 들어오면 항상 switch 문, ENUM TYPE 추가 해주어야 한다.

예를 들어, 생일 휴가를 주는 회사라면, 생일 휴가를 추가해야하고, 만약 뭔가를 추가해야하는 것을 실수로 switch case 넣어주지 않는다면 잔업수당이 제외된다. 

어떻게 개선하는게 좋은가? 가장 깔끔한 방법은 새로운 상수를 추가할 때, 잔업 수당 '전략'을 선택하도록 하는 것이다. 

잔업 수당 계산을 private 중첩 열거 타입(다음 코드의 PayType)으로 옮기고 PayrollDay 열거 타입의 생성자에서 이중 적당한 것을 선택한다. 

그러면 PayrollDay 열거 타입은 잔업수당 계산을 그 전략 열거 타입에 위임하여, switch 문이나 상수별 메서드 구현이 필요 없게 된다. 

**코드 34-9 전략 열거 타입 패턴**
```java
enum PayrollDay {
    MONDAY(WEEKDAY), TUESDAY(WEEKDAY), WEDNESDAY(WEEKDAY),
    THURSDAY(WEEKDAY), FRIDAY(WEEKDAY), 
    SATURDAY(WEEKEND), SUNDAY(WEEKEND);
    
    private final PayType payType;
    
    PayRollDay(PayType payType) { this.payType = payType; }
    
    int pay(int minutesWorked, int payRate) {
        return payType.pay(minutesWorked, payRate);
    }
    
    enum PayType {
        WEEKDAY {
            int overtimePay(int minsWorked, iny payRate) {
                return minsWorked <= MINS_PER_SHIFT ? 0 : (minsWorked - MINS_PER_SHIFT) * payRate / 2;
            }
        }, 
        WEEKEND {
            int overtimePay(int minsWorked, int payRate) {
                return minsWorked * payRate / 2;
            }
        };
        
        abstract  int overtimePay(int mins, int payRate);
        private static final int MINS_PER_SHIFT = 8 * 60;
        
        int pay(int minsWorked, int payRate) {
            int basePay = minsWorked * payRate;
            return basePay + overtimePay(minsWorked, payRate);
        }
    }
}
```

switch문은 열거 타입의 상수별 동작을 구현하는 데 적합하지 않음. 

하지만 기존 열거 타입에 상수별 동작을 혼합ㅂ해 넣을 때는 switch문이 좋은 선택일 수 있다. 

**34-10 switch문을 이용해 원래 열거 타입에 없는 기능을 수행한다.**
```java
public static Operation inverse(Operation op) {
    switch(op) {
        case PLUS: return Operation.MINUS;
        case MINSU: return Operation.PLUS;
        case TIMES: return Operation.DIVIDE;
        case DIVIDE: return Operation.TIMES;
        
        default: throw new AssertionError("알 수 없는 연산: " + op);
    }
}
```

추가하려는 메서드가 의미상 열거 타입에 속하지 않는다면 직접 만든 열거 타입이라도 이 방식을 적용하는 게 좋다. 

종종 쓰이지만 열거 타입 안에 포함할만큼 유용하지는 않은 경우도 마찬가지다. 

대부분의 경우 열거 타입의 성능은 정수 상수와 별반 다르지 않다. 열거 타입을 메모리에 올리는 공간과 초기화하는 시간이 들긴 하지만 체감될 정도는 아니다. 

필요한 원소를 컴파일타임에 다 알 수 있는 상수 집합이라면 항상 열거 타입을 사용하자. 

그래서 열거 타입을 과연 언제 쓰란 말인가? **필요한 원소를 컴파일타임에 다 알 수 있는 상수 집합이라면 항상 열거 타입을 사용하자**

태양계 행성, 한 주의 요일, 체스 말처럼 본질적으로 열거 타입인 타입은 당연히 포함된다. 

그리고 메뉴 아이템, 연산 코드, 명령줄 플래그 등 허용하는 값 모두를 컴파일 타임에 이미 알고 있을 때도 쓸 수 있다. 
