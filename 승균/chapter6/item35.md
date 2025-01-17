## ordinal 메서드 대신 인스턴스 필드를 사용하라

## **핵심정리**
대부분의 열거 타입 상수는 자연스럽게 하나의 정수값에 대응된다. 

모든 열거 타입은 해당 상수가 열거 타입에서 몇 번째 위치인지를 반환하는 ordinal이라는 메서드를 제공한다. 

이런 이유로 열거 타입 상수와 연결된 정수값이 필요하면 ordinal 메서드를 이용하고 싶은 유혹에 빠진다. 

**35-1 ordinal을 잘못 사용한 예 - 따라 하지 말 것!**

```java
public enum Ensemble {
    SOLO, DUET, TRIO, QUARTET, QUINTET, 
    SEXTET, SEPTET, OCTET, NONET, DECTET;
    
    public int numberOfMusicians() { return ordinal() + 1; }
}
```

유지보수하기가 끔찍한 코드다. 

상수 선언 순서를 바꾸는 순간 numberOfMusicians가 오동작하며, 이미 사용 중인 정수와 값이 같은 상수는 추가할 방법이 없다.

예컨대 8중주(octet)상수가 이미 있으니 똑같이 8명이 연주하는 복4중주(double quartet)는 추가할 수 없다.

값을 중간에 비워둘 수도 없다.

열거 타입 상수에 연결된 값은 ordinal 메서드로 얻지 말고, 인스턴스 필드에 저장하자. 
```java
public enum Ensemble {
    SOLO(1), DUET(2), TRIO(3), QUARTET(4), QUINTET(5),
    SEXTET(6), SEPTET(7), OCTET(8), DOUBLE_QUARTET(8),
    NONET(9), DECTET(10), TRIPLE_QUARTET(12);
    
    private final int numberOfMusicians;
    Ensemble(int size) { this.numberOfMusicians = size; }
    public int numberOfMusicians() { return numberOfMusicians; }
}
```

