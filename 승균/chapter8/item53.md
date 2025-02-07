## **item53 : 다중정의는 신중히 사용하라**

## **핵심정리**

가변인수(varargs)메서드는 명시한 타입의 인수를 0개 이상 받을 수 있다.

가변인수 메서드를 호출하면, 가장 먼저 인수의 개수와 길이가 같은 배열을 만들고 인수들을 이 배열에 저장하여 가변인수 메서드에 건네준다. 

**코드53-1 간단한 가변인수 활용 예**
```java
static int sum(int... args) {
    int sum = 0;
    for(int arg : args) 
        sum += arg;
    return sum;
}
```

인수가 1개 이상이어야 할 떄도 있다. 최솟값을 찾는 메서드인데 인수를 0개만 받을 수 있도록 설계하는 건 좋지 않음 

**코드53-2 인수가 1개 이상이어야 하는 가변인수 메서드 - 잘못 구현한 예!**
```java
static int min(int... args) {
    if(args.length == 0)
        throw new IllegalArgumentException("인수가 1개 이상 필요합니다.");
    int min = args[0];
    for(int i = 1; i < args.length; i++) 
        if(args[i] < min)
            min = args[i];
    return min;
}
```

가장 심각한 문제는 인수를 0개만 넣어 호출하면 (컴파일타임이 아닌) 런타임에 실패한다는 점이다. 코드도 지저분하다. 

**코드53-3 인수가 1개 이상이어야 할 때 가변인수를 제대로 사용하는 방법**
```java
static int min(int firstArg, int... remainingArgs) {
    int min = firstArg;
    for(int arg: remainingArgs)
        if(arg < min)
            min = arg;
    return min;
}
```

printf 가변인수와 한 묶음으로 자바에 도입되었고, 이때 핵심 리플렉션기능도 재정비 되었다. 

가변인수는 성능에 민감한 상황이라면 걸림돌이 된다.

```java
public void foo() { }
public void foo(int a1) { }
public void foo(int a1, int a2) { }
public void foo(int a1, int a2, int a3) { }
public void foo(int a1, int a2, int a3, int... rest) { }
```
