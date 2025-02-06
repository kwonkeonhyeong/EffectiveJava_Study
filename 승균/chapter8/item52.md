## item52 : 다중정의는 신중히 사용하라

## **핵심정리**

### 코드 52-1 컬렉션 분류기 - 오류!

```java
import java.util.ArrayList;
import java.util.HashSet;

public class CollectionClassifier {
    public static String classify(Set<?> s) {
        return "집합";
    }

    public static String classify(List<?> lst) {
        return "리스트";
    }
    
    public static String classify(Collection<?> c) {
        return "그 외";
    }
    
    public static void main(String[] args) {
        Collection<?>[] collections = {
                new HashSet<String>(),
                new ArrayList<BigInteger>(),
                new HashMap<String, String>().values()
        };
        
        for(Collection<?> c : collections)
            System.out.println(classify(c));
    }
}
```

"집합", "리스트", "그 외" 차례로 출력, 실제로 수행해보면 "그 외"만 세 번 연달아 출력한다. 

다중정의(overloading, 오버로딩)된 새 classify 중 어느 메서드를 호출할지가 컴파일타임에 정해지기 때문이다. 컴파일타임에는 for 문 안의 c는 항상 Collection<?> 타입이다. 

런타임에는 타입이 매번 달라지지만, 호출할 메서드를 선택하는 데는 영향을 주지 못한다.

이처럼 직관과 어긋나는 이유는 재정의한 메서드는 동적으로 선택되고, 다중정의한 메서드는 정적으로 선택되기 때문이다. 

**코드 52-2 재정의된 메서드 호출 메커니즘 - 이 프로그램은 무엇을 출력할까?**
```java
class Wine {
    String name() { return "포도주"; }
}

class SparklingWine extends Wine {
    @Override String name() { return "발포성 포도주"; }
}

class Champagne extends SparklingWine {
    @Override String name() { return "샴페인"; }
}

public class Overriding {
    public static void main(String[] args) {
        List<Wine> wineList = List.of(new Wine(), new SparklingWine(), new Champane());
        
        for(Wine wine : wineList)
            System.out.println(wine.name());
    }
}

```
Wine 클래스에 정의된 name 메서드는 하위 클래스인 SparklingWine과 Champagne에서 재정의된다. 

이 프로그램은 "포도주", "발포성 포도주", "샴페인"을 차례로 출력한다. 

for문에서의 컴파일타임 타입이 모두 Wine인 것에 무관하게 항상 '가장 하위에서 정의한' 재정의 메서드가 실행

다중정의된 메서드 사이에서는 객체의 런타임 타입은 전혀 중요치 않다.

코드 52-1 CollectionClassifier 예에서 프로그램의 원래 의도는 매개변수의 런타임 타입에 기초해 적절한 다중정의 메서드로 자동분배하는 것이었다. 


**다중정의가 혼동을 일으키는 상황을 피해야함.**

안전하고 보수적으로 가려면 매개변수 수가 같은 다중정의는 만들지 않는 편이 좋음

가변인수를 사용하는 메서드라면 다중정의를 아예 하지 말아야함. 

**다중정의하는 대신 메서드 이름을 다르게 지어주는 길도 항상 열려있다.**

ObjectOutputStream 클래스를 살펴보자. 이 클래스의 write 메서드는 모든 기본 타입과 일부 참조 타입용 변형을 가지고 있다.

writeBoolean(boolean), writeInt(int), wirteLong(long) 같은 식이다.

다중정의보다 나은 또 다른 점은 read 메서드의 이름과 짝을 맞추기 좋다. 
readBoolean(), readInt(), readLong() 같은 식이다. 

한편, 생성자는 이름을 다르게 지을 수 없으니 두 번째 생성자부터는 무조건 다중정의가 된다. 하지만 정적 팩터리라는 대안을 활용할 수 있는 경우가 많다. 

생성자는 재정의할 수 없으니 다중정의와 재정의가 혼용될 걱정은 넣어둬도 된다. 그래도 여러 생성자가 같은 수의 매개변수를 받아야하는 경우를 완전히 피해갈 수는 없을 테니, 그럴 떄를 대비해 안전 대책을 배워두면 도움이 될 것이다. 

매개변수 중 하나 이상이 "근본적으로 다르다(radically different)"면 헷갈릴 일이 없다. 

```java
public class SetList {
    public static void main(String[] args) {
        Set<Integer> set = new TreeSet<>();
        List<Integer> list = new ArrayList<>();
        
        for(int i = -3; i < 3; i++) {
            set.add(i);
            list.add(i);
        }
        
        for(int i = 0; i < 3; i++) {
            set.remove(i);
            list.remove(i);
        }
        System.out.println(set + " " + list);
    }
}
```
-3 부터 2까지의 정수를 정렬된 집합과 리스트에 각각 추가한 다음, 양쪽에 똑같이 remove 메서드를 세 번 호출한다. 

리스트에서는 홀수를 제거한 후 "[-3, -2, -1] [-2, 0, 2]"를 출력한다. 

set.remove(i)의 시그니처는 remove(Object)다. 다중정의된 다른 메서드가 없으니 기대한 대로 동작하여 집합에서 0 이상의 수들을 제거

list.remove(i) 다중정의된 remove(int index) 선택한다. 
그런데 이 remove는 '지정한 위치'의 원소를 제거하는 기능을 수행

메서드를 다중정의할 때, 서로 다른 함수형 인터페이스라도 같은 위치의 인수로 받아서는 안 된다.

서로 다른 함수형 인터페이스라도 서로 근본적으로 다르지 않다는 뜻이다. 

컴파일 할 때 명령줄 스위치로 -Xlint:overloads 지정하면 다중정의를 경고해준다. 

Object 클래스 타입과 배열 타입은 근본적으로 다르다. Serializable과 Cloneable 외의 인터페이스 타입과 배열 타입도 근본적으로 다르다. 

**코드52-3 인수를 포워드하여 두 메서드가 동일한 일을 하도록 보장한다**
```java
public boolean contentEquals(StringBuffer sb) {
    return contentEquals((CharSequence) sb);
}
```