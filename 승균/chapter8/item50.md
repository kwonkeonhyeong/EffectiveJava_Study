## 적시에 방어적 복사본을 만들라

## **핵심정리**

자바로 작성한 클래스는 시스템의 다른 부분에서 무슨 짓을 하든 그 불변식이 지켜진다. 

메모리 전체를 하나의 거대한 배열로 다루는 언어에서는 누릴 수 없는 강점이다.

자바라 해도 다른 클래스로부터의 침범을 아무런 노력 없이 다 막을 수 있는 건 아니다. 

**클라이언트가 여러분의 불변식을 깨드리려 혈안이 되어 있다고 가정하고 방어적으로 프로그래밍해야한다**

실제로도 악의적인 의도를 가진 사람들이 시스템의 보안을 뚫으려는 시도가 늘고 있다.

평범한 프로그래머도 순전히 실수로 여러분의 클래스를 오작동하게 만들 수 있다.

어떤 경우든 적절치 않은 클라이언트로부터 클래스를 보호하는 데 충분한 시간을 투자하는게 좋다. 

어떤 객체든 그 객체의 허락 없이는 외부에서 내부를 수정하는 일은 불가능하다.

하지만 주의를 기울이지 않으면 자기도 모르게 내부를 수정하도록 허락하는 경우가 생긴다. 

예컨대 기간(period)을 표현하는 다음 클래스는 한번 값이 정해지면 변하지 않도록 할 생각이다.

**코드 50-1 기간을 표현하는 클래스 - 불변식을 지키지 못했다.**
```java
public final class Period {
    private final Date start;
    private final Date end;

    /**
     * @param start 시작 시각
     * @param end 종료 시각; 시작 시각보다 뒤여야 한다.
     * @throws IllegalArgumentException 시작 시각이 종료 시각보다 늦을 때 발생한다.
     * @throws NullPointerException start나 end가 null이면 발생한다.
     */
    public Period(Date start, Date end) {
        if(start.compareTo(end) > 0) 
            throw new IllegalArgumentException(start + "가 " + end + "보다 늦다.");
        this.start = start;
        this.end = end;
    }
    
    ,,, // 나머지 코드 생략
}
```

**코드 50-2 Period 인스턴스의 내부를 공격해보자.**
```java
Date start = new Date();
Date end = new Date();
Period p = new Period(start, end);
end.setYear(78); // p의 내부를 수정했다
```

Date 대신 불변인 Instant 사용하면 된다(혹은 LocalDateTime이나 ZonedDateTime 사용해도 된다.) **Date는 낡은 API이니 새로운 코드를 작성할 때는 더 이상 사용하면 안된다.**

Date처럼 가변인 낡은 값 타입을 사용하던 시절이 워낙 길었던 탓에 여전히 많은 API와 내부 구현에 그 잔재가 남아 있다. 

외부 공격으로부터 Period 인스턴스의 내부를 보호하려면 **생성자에서 받은 가변 매개변수 각각을 방어적으로 복사(defensive copy)해야 한다.**


**코드 50-3 수섲ㅇ한 생성자 - 매개변수의 방어적 복사본을 만든다.**
```java
public Period(Date start, Date end) {
    this.start = new Date(start.getTime());
    this.end = new Date(end.getTime());
    
    if(this.start.compareTo(this.end) > 0)
        throw new IllegalArgumentException(this.start + "가 " + this.end + "보다 늦다.");
}
```

새로 작성한 생성자를 사용하면 앞서의 공격은 더 이상 Period 위협이 되지 않는다. **매개변수의 유효성 검사하기 전에 방어적 복사본을 만들고, 이 복사본으로 유효성을 검사한 점에 주목하자**

순서가 부자연스러워 보이겠지만 반드시 이렇게 작성해야 한다. 멀티스레딩 환경이라면 원본 객체의 유효성을 검사한 후 복사본을 만드는 그 찰나의 순간에 다른 스레드가 원본 객체를 수정할 위험이 있기 때문이다. 

방어적 복사를 매개변수 유효성 검사전에 수행하면 이런 위험에서 해방될 수 있다. 컴퓨터 보안 커뮤니티에서는 이를 검사시점/사용시점(time-of-check/time-of-use) 공격 혹은 영어 표기를 줄여서 TOCTOU 공격이라 한다.

**매개변수가 제3자에 의해 확장될 수 있는 타입이라면 방어적 복사본을 만들 때 clone을 사용해서는 안된다.**

**코드50-4 Period 인스턴스를 향한 두 번째 공격**
```java
Date start = new Date();
Date end = new Date();
Period p = new Period(start, end);
p.end().setYear(78);
```

두 번째 공격을 막아내려면 단순히 접근자가 **가변 필드의 방어적 복사본을 반환하면 된다.**

**코드 50-5 수정한 접근자 - 필드의 방어적 복사본을 반환한다.**
```java
public Date start() {
    return new Date(start.getTime());
}

public Date end() {
    return new Date(end.getTime());
}
```

새로운 접근자까지 갖추면 Period는 완벽한 불변으로 거듭난다. 아무리 악의적인 혹은 부주의한 프로그래머라도 시작 시각이 종료 시각보다 나중일 수 없다.는 불변식을 위배할 방법이 없다. 

Period 자신 말고는 가변 필드에 접근할 방법이 없으니 확실하다. 모든 필드가 객체 안에 완벽하게 캡슐화되었다.

생성자와 달리 접근자 메서드에서는 방어적 복사에 clone 사용해도 된다. Period 가지고 있는 Date 객체는 java.util.Date 확실하기 때문이다

매개변수를 방어적으로 복사하는 목적이 불변 객체를 만들기 위해서만은 아니다. 메서드든 생성자든 클라이언트가 제공한 객체의 참조를 내부의 자료구조에 보관해야 할 때면 항시 그 객체가 잠재적으로 변경될 수 있는지를 생각해야 한다.

변경될 수 있는 객체라면 그 객체가 클래스에 넘겨진 뒤 임의로 변경되어도 그 클래스가 문제없이 동작하맂를 따져보라. 확신할 수 없다면 복사본을 만들어 저장해야 한다. 

클라이언트가 건네준 객체를 내부의 Set 인스턴스에 저장하거나 Map 인스턴스의 키로 사용한다면, 추후 그 객체가 변경될 경우 객체를 담고 있는 Set 혹은 Map의 불변식이 깨질 것이다.

내부 객체를 클라이언트에 건네주기 전에 방어적 복사본을 만드는 이유도 마찬가지다. 여러분의 클래스가 불변이든 가변이든, 가변인 내부 객체를 클라이언트에 반환할 때는 반드시 심사숙고해야 한다.
