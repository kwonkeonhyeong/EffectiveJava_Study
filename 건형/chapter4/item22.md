## 📖 인터페이스는 타입을 정의하는 용도로만 사용하라

### 핵심 내용

## 💡 주요 내용 정리 & 🛠️ 실습 코드

- 인터페이스는 자신을 구현한 클래스의 인스턴스를 참조할 수 있는 타입 역할을 한다.

- 클래스가 어떤 인터페이스를 구현한다는 것은 자신의 인스턴스로 무엇을 할 수 있는지를 클라이언트에 얘기해주는 것이다.

- 인터페이스는 오직 이 용도로만 사용해야 한다.

### Interface 를 잘못 사용한 예

상수 인터페이스 안티패턴은 인터페이스를 잘못 사용한 예!!
```java
public interface PhysicalConstants {
    static final double AVOGADROS_NUMBER = 6.002_140_857e23;
    static final double BOLTZMANN_CONSTANT = 1.380_648_52e-23;
    static final double ELECTRON_MASS = 9.109_383_56e-31;
}
```

- 클래스 내부에서 사용하는 상수는 외부 인터페이스가 아니라 내부 구현에 해당한다.

- 따라서, 상수 인터페이스를 구현하는 것은 이 내부 구현을 클래스의 API로 노출하는 행위다.

- 이렇게 되면 사용자에게 혼란을 주기도 하먀, 더 심하게는 클라이언트 코드가 내부 구현에 해당하는 이 상수들에 종속되게 된다.

- 이렇게 종속되어 버리면 다음 릴리스에서 이 상수들을 더 이상 쓰지 않게 되어도 호환성을 위해 여전히 해당 인터페이스를 구현하고 있어야 하는 문제에 직면한다.


## 추가 강의 내용 정리

### Constant static final은 anti pattern

- 클래스 내부에서 사용하는 상부는 내부 구현에 해당된다.

- 차라리 클래스에 static final 로 추가하는 것이 더 낫다
```java
public class OrderService {
    public static final double SECOND_TO_MIN = 60.;
}
```
- 만약 여러 곳에서 사용해야 할 값이라면 util class! 로 구현하자
```java
public class timeConvertUtil {
    public static final double SECOND_TO_MIN = 60.;
    public static double secondToMin(double second) {
        return second/SECOND_TO_MIN;
    }
}
```

### Static Import

```java
import example.chapter4.TimeConvertUtil;

...
TimeConvertUtil.SECOND_TO_MIN

import example.chapter4.TimeConvertUtil.SECOND_TO_MIN;
SECOND_TO_MIN
```

- 두 가지 방법이 있는데, 전자의 방법을 추천한다.

- 비슷한 이름의 상수 값이 (enum도 똑같다.) import 될 경우 혼란을 일으킬 수 있고, 심지어 같은 값이 있는 경우도 아주 가끔이지만 존재한다.

- 코드를 줄이기 위해 후자를 사용하려는 의도가 많지만, 언급한 것처럼 혼란을 일으킬수 있으니 이를 조심하자!!


### 정리

인터페이스를 타입을 정리하는 용도로만 쓰고, 상수 공개용으로 사용하지 마라!!

필요하다면 Class에 담아라

Properties 에 값을 선언해두고 injection 하는 것도 방법일 수 있다.


## 🤔 생각 정리

- interface를 통해서 상수를 구현하는 행위는 앞 item에서 알아봤던 캡슐화 관련된 내용에도 위배되는 행동이라고 생각하게 되었습니다.

