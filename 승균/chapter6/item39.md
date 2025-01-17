## 명명 패턴보다 애너테이션을 사용하라

## **핵심정리**

전통적으로 도구나 프레임워크가 특별히 다뤄야 할 프로그램 요소에는 딱 구분되는 명명 패턴을 적용해왔다.

예컨대 테스트 프레임워크인 JUnit 버전 3까지 테스트 메서드 이름을 test로 시작하게끔 했다.

실수로 이름은 testSafetyOverride로 지으면 Junit3 메서드를 무시하고 지나치기에 개발자는 이 테스트가 실패했으나, 통과했다고 오해 할 수 있었다.

명명 패턴의 단점 두 번쨰는 올바른 프로그램 요소에만 사용되리라 보증할 방법이 없다는 것이다. 

예컨대 클래스 이름을 TestSafety Mechanisms 지워 Junit 던져줬다고 생각해보자. 
이번에도 테스트 메서드를 수행해주길 기대하지만 전혀 수행되지 않음

세번 째, 단점은 프로그램 요소를 매개변수로 전달할 마땅한 방법이 없다.
특정 예외를 던져야만 성공하는 테스트가 있다고 해보자.
기대하는 예외 타입을 테스트에 매개변수로 전달해야만 하는 상황이다. 
예외의 이름을 테스트 메서드 이름에 덧붙이는 방법도 있지만 보기도 나쁘고 꺠지기 쉬움

컴파일러는 메서드 이름에 덧붙인 문자열이 예외를 가르키는지도 알 수 없음.

애너테이션은 이 모든 문제를 해결해주는 개념이다.

**코드39-1 마커(marker) 애너테이션 타입 선언**
```java
import java.lang.annotation.*;

/**
 * 테스트 메서드임을 선언하는 애너테이션이다.
 * 매개변수 없는 정적 메서드 전용이다.
 */
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface Test {
    
}
```

애너테이션 선언에 다는 애너테이션을 메타애너테이션이라 한다.

@Retention(RetensionPolicy.RUNTIME) 메타애너테이션은 @Test가 런타임에도 유지되어야 한다는 표시이다.
@Target(ElementType.METHOD) 메타애너테이션은 @Test가 반드시 메서드 선언에서만 사용돼야 한다고 알려준다.

앞 코드의 메서드 주석에는 "매개변수 없는 정적 메서드 전용이다"라고 쓰여있다. 

이 제약을 컴파일러가 강제할 수 있으면 좋겠지만, 그렇게 하려면 적절한 애너테이션 처리기를 직접 구현해야 한다.

적절한 애너테이션 처리기 없이 인스턴스 메서드나 매개변수가 있는 메서드에 달면 어떻게 된다? 컴파일은 잘되지만, 테스트 도구를 실행할 때 문제가 된다.

@Test애너테이션을 실제 적용한 모습이다. "아무 매개변수 없이 단순히 대상에 마킹(marking)한다" 마커애너테이션이라 한다.

**코드 39-2 마커애너테이션을 사용한 프로그램 예**
```java
public class Sample {
    @Test public static void m1() { } // 성공
    @Test public static void m2() { } 
    @Test public static void m3() { // 실패
        throw new RuntimeException("실패");
    }
    public static void m4() { }
    @Test public  void m5() { } // 잘못 사용한 예: 정적 메서드가 아니다.
    public static void m6() { }
    @Test public static void m7 { // 실패해야 한다.
        throw new RuntimeException("실패");
    }
    public static void m8() { }
}
```

Sample 클래스에는 정적 메서드 7개  그 중 4개에 @Test

m3와 m7메서드는 예외를 던지고 m1과 m5는 그렇지 않다.

그리고 m5는 인스턴스 메서드이므로 @Test를 잘못 사용한 경우다.

@Test 애너테이션이 Sample 클래스의 의미에 직접적인 영향을 주지 않는다. 

그저 이 애너테이션에 관심 있는 프로그램에게 추가 정보를 제공할 뿐이다.

**코드39-3 마커 애너테이션을 처리하는 프로그램**

```java
import java.lang.reflect.InvocationTargetException;

public class RunTests {
    public static void main(String[] args) throws Exception {
        int tests = 0;
        int passed = 0;
        Class<?> testClass = Class.forName(args[0]);
        for (Method m : testClass.getDeclaredMethods()) {
            if (m.isAnnotationPresent(Test.class)) {
                tests++;
                try {
                    m.invoke(null);
                    passed++;
                } catch (InvocationTargetException wrappedExc) {
                    Throwable exc = wrappedExc.getCause();
                    System.out.println(m + " 실패: " + exc);
                } catch (Exception exc) {
                    System.out.println("잘못 사용한 @Test: " +m);
                }
            }
        }
        System.out.println("성공: %d, 실패: %d%n", passed, tests - passed);
    }
}
```

클래스에서 @Test 애너테이션이 달린 메서드를 차례로 호출 

isAnnotationPresent가 실행할 메서드를 찾아주는 메서드다.


InvocationTargetException외의 예외 발생하면 @Test 애너테이션을 잘못 사용

인스턴스 메서드, 매개변수가 있는 메서드, 호출할 수 없는 메서드등에 달음.

두 번째 블록은 잘못 사용해서 발생한 예외를 붙잡아 적절한 오류 메시지를 출력한다.

**코드 39-4 매개변수 하나를 받는 애너테이션 타입**

```java
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface  ExceptionTest {
    Class<? extends Throwable> value();
}
```

매개변수 타입은 Class<? extends Throwable>이다.

"Throwable을 확장한 클래스의 Class 객체" 따라서 모든 예외 수용

**코드39-5 매개변수 하나짜리 애너테이션을 사용한 프로그램**
```java
public class Sample2 {
    @ExcepionTest(ArithmeticException.class)
    public static void m1() { // 성공
        int i = 0;
        i = i / i;
    }
    
    @ExceptionTest(ArithmeticException.class)
    public static void m2() { // 다른 예외 발생 실패 
        int[] a = new int[0];
        int i = a[1];
    }
    
    @ExceptionTest(ArithmeticException.class) 
    public static void m3(){ }; // 실패 예외 발생 x
}
```


애너테이션 다룰 수 있도록 수정

```java
import java.lang.reflect.InvocationTargetException;
if(m.isAnnotationPresent(ExceptionTest .class)){
    tests++;
    try{
        m.invoke(null);
        System.out.printf("테스트 %s 실패: 예외를 던지지 않음 %n",m);
    }catch(InvocationTargetException wrappedEx) {
            Throwable exc = wrappedEx.getCause();
            Class<? extends Throwable> excType = m.getAnnotation(ExceptionTest.class).value();
            if(excType.isInstance(exc)) {
                passed++;
            }else {
                System.out.printf("테스트 %s 실패: 기대한 예외 %s, 발생한 예외 %s%n", m, excType.getName(), exc);           
            }
    } catch(Excepion exc) {
        System.out.println("잘못 사용한 @ExceptionTest: " + m);
    }
} 
```