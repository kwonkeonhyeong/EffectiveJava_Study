## ordinal 인덱싱 대신 EnumMap 사용하라

## **핵심정리**

```java
class Plant {
    enum LifeCycle { ANNUAL, PERENNIAL, BIENNIAL }
    
    final String name;
    final LifeCycle lifeCycle;
    
    Plant(String name, LifeCycle lifeCycle) {
        this.name = name;
        this.lifeCycle = lifeCycle;
    }
    
    @Override public String toString() {
        return name;
    }
}
```

심은 식물 배열 하나로 관리하고, 이들을 생애주기(한해살이, 여러해살이, 두해살이)별로 묶어보자.

이때 어떤 프로그래머는 집합들을 배열 하나에 넣고 생애주기의 ordinal 값을 그 배열의 인덱스로 사용

**코드 37-1 ordinal() 배열 인덱스로 사용 - 따라 하지 말 것!**
```java
Set<Plant>[] plantsByLifeCycle = (Set<Plant>) new Set[Plant.LifeCycle.values().length];

for (int i=0; i < plantsByLifeCycle.length; i++) 
    plantsByLifeCycle[i] = new HashSet<>();

for (Plant p : garden) 
    plantsByLifeCycle[p.lifeCycle.ordinal()].add(p);

for (int i = 0; i < plantsByLifeCycle.length; i++) {
    System.out.printf("%s: %s%n", Plant.LifeCycle.values()[i], plantsByLifeCycle[i]);
}
```

동작은 하지만 문제가 한가득이다.

배열은 제네릭과 호환되지 않으나 비검사 형변환을 수행해야 하고 깔끔히 컴파일되지 않을 것이다.

배열은 각 익덱스의 의미를 모르니 출력 결과에 직접 레이블을 달아야 한다.

가장 심각한 정확한 정수 값을 사용한다는 것을 우리가 보증해야 한다.

잘못된 값을 사용해도 묵묵히 동작하거나 ArrayIndexOutOfBoundsException 던짐 <= 운이 좋은 경우임.

훨씬 멋진 해결책이 있다. 여기서 배열을 실질적으로 열거 타입 상수를 값으로 매핑하는 역할 

Map 사용 가능하다. 열거 타입을 키로 사용하도록 설계한 아주 빠른 Map 구현체가 존재하는 데, 바로 EnumMap이 그 주인공

**코드 37-2 EnumMap을 사용해 데이터와 열거 타입을 매핑한다.**
```java
Map<Plant.LifeCycle, Set<Plant>> plantsByLifeCycle = new EnumMap<>(Plant.LifeCycle.class);
for (Plant.LifeCycle lc : Plant.LifeCycle.values())
    plantsByLifeCycle.put(lc, new HashSet<>());
for (Plant p : garden) 
    plantsByLifeCycle.get(p.lifeCycle).add(p);
System.out.println(plantsByLifeCycle);
```

안전하지 않은 형변환은 쓰지 않고, 맵의 키인 열거 타입이 그 자체로 출력용 문자열을 제공하니 출력 괄겨에 직접 레이블을 달 일도 없다.

EnumMap의 성능이 ordinal 쓴 배열에 비견되는 이유는 그 내부에서 배열을 사용하기 때문이다.

EnumMap 생성자가 받는 키 타입의 Class 객체는 한정적 타입 토큰으로 런타임 제네릭 타입 정보를 제공한다.

**스트림을 사용한 코드 1 - EnumMap을 사용하지 않는다.!**
```java
System.out.println(Arrays.stream(garden).collect(groupingBy(p -> p.lifeCycle)));
```

EnumMap이 아닌 고유한 맵 구현체를 사용했기 때문에 EnumMap을 써서 얻은 공간과 성능 이점이 사라진다는 문제

매개변수 3개짜리 Collectors.groupingBy 메서드는 mapFactory 매개변수에 원하는 맵 구현체를 명시해 호출

**코드 37-4 스트림을 사용한 코드 2 - EnumMap을 이용해 데이터와 열거 타입을 매핑했다.**
```java
System.out.println(Arrays.stream(garden).collect(groupingBy(p-> p.lifeCycle, () -> new EnumMap<>(LifeCycle.class), toSet())));
```
스트림을 사용하면 EnumMap만 사용했을 때와 살짝 다르게 동작한다.

EnumMap 버전은 언제나 식물의 생애주기당 하나씩의 중첩 맵을 만들지만, 스트림 버전에서는 해당 생애주기에 속하는 식물이 있을 때만 만든다.

예컨대 정원에 한해살이와 여러해살이 식물만 살고 두해살이는 없다면, EnumMap 버전에서는 맵을 3개 만들고 스트림 버전에서는 2개만 만든다.

두 열거 타입 값들을 매핑하느라 ordinal을 (두 번이나) 쓴 배열들의 배열을 본 적이 있을 것이다. 

다음은 이 방식을 적용해 두 가지 상태(Phase)를 전이(Transition)와 매핑하도록 구현한 프로그램이다.

예컨대 액체(LIQUID)에서 고체(SOLID)로의 전이는 응고(FREEZE)가 되고, 액체에서 기체(GAS)로의 전이는 기화(BOIL)가 된다.

**37-5 배열들의 배열의 인덱스에 ordinal()을 사용 - 따라 하지 말 것!**
```java
public enum Phase {
    SOLID, LIQUID, GAS;
    
    public enum Transition {
        MELT, FREEZE, BOIL, CONDENSE, SUBLIME, DEPOSIT;
        
        private static final Transition[][] TRANSITIONS = {
                {null, MELT, SUBLIME},
                {FREEZE, null, BOIL},
                {DEPOSIT, CONDENSE, null}
        };
        
        public static Transition from(Phase from, Phase to) {
            return TRANSITIONS[from.ordinal()][to.ordinal()];
        }
    }
}
```

해당 코드는 컴파일러는 ordinal 배열 인덱스의 관계를 알 수 없음. 

Phase or Phase.Transaction 잘못 수정하면 런타입 오류 발생

예외 던지지 않을 수도 있고, 운이 좋으면 예외가 던져짐. 절대 사용 x

무조건 EnumMap사용하기!!

**코드 37-6 중첩 EnumMap으로 데이터와 열거 타입 쌍을 연결했다.**
```java
public enum Phase {
    SOLID, LIQUID, GAS;
    
    public enum Transition {
        MELT(SOLID, LIQUID), FREEZE(LIQUID, SOLID),
        BOIL(LIQUID, GAS), CONDENSE(GAS, LIQUID),
        SUBLIME(SOLID, GAS), DEPOSIT(GAS, SOLID)
        
        private final Phase from;
        private final Phase to;
        
        Transition(Phase from, Phase to) {
            this.from = from;
            this.to = to;
        }
        
        // 상전이 맵을 초기화 한다.
        private static final Map<Phase, Map<Phase, Transition>> m = Stream.of(values()).collect(groupingBy(t -> t.from, () -> new EnumMap<>(Phase.class), toMap(t -> t.to, t -> t, (x,y) -> y, () -> new EnumMap<>(Phase.class))));
        
        public static Transition from(Phase from, Phase to) {
            return m.get(from).get(to);
        }
    }
}
```

Map<Phase, Map<Phase, Transition>>은 "이전 상태에서 '이후 상태에서 전이로의 맵'에 대응시키는 맵"이라는 뜻

맵의 초기화를 위해 수집기 (java.util.stream.Collector) 2개를 차례로 사용.

첫 번째 수집기인 groupingBy에서는 전이를 이전 상태를 기준으로 묶고, 두 번째 수집기인 toMap에서는 이후 상태를 전이 대응시키는 EnumMap 생성한다.

두 번째 수집기의 병합 함수임 (x,y) -> y는 선언만 하고 실제로 쓰이지 않는데, 이는 단지 EnumMap을 얻으려면 맵 팩터리가 필요하고 수집기들은 점층적 팩터리를 제공하기 때문이다.

새로운 상태인 플라스마 추가. 

이 상태와 연결된 전이 2개. 플라스마로 변하는 이온화(IONIZE)이고, 둘째는 플라스마에서 기체로 변하는 탈이온화(DEIONIZE)다. 

배열로 만든 37-5 phase에 1개 , Phase.Transaction에 2개 추가 해야함.
9개짜리인 배열을 원소 16개짜리로 교체

반면에 EnumMap 버전에서는 상태 목록에 PLSAMA 추가하고, 전이 목록에 IONIZE(GAS, PLASMA와 DEIONZE(PLASMA, GAS)만 추가하면 끝이다.

**코드 37-7 EnumMap 버전에 새로운 상태 추가하기**
```java
public enum Phase {
    SOLID, LIQUID, GAS, PLASMA;
    
    public enum Transition {
        MELT(SOLID, LIQUID), FREEZE(LIQUID, SOLID),
        BOIL(LIQUID, GAS), CONDENSE(GAS, LIQUID),
        SUBLIME(SOLID, GAS), DEPOSIT(GAS, SOLID),
        IONIZE(GAS, PLASMA), DEIONIZE(PLASMA, GAS);
    }
}
```

잘못 수정할 가능성이 지극히 적음
