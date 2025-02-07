## **item54: null이 아닌, 빈 컬렉션이나 배열을 반환하라**

## **핵심정리**

**코드54-1 컬렉션이 비었으면 null 반환한다. -따라 하지 말 것!**
```java
private final List<Cheese> cheesesInStock = ...;

/**
 * @return 매장 안의 모든 치즈 목록을 반환한다.
 *      단, 재고가 하나도 없다면 null을 반환한다.
 */
public List<Cheese> getCheeses() {
    return cheesesInStock.isEmpty() ? null : new ArrayList<>(cheesesInStock);
}
```

재고가 없다고 해서 특별히 취급할 이유는 없다.

위에 처럼 작성하면, null처리 상황 코드도 추가해야함.

```java
List<Cheese> cheeses = shop.getCheeses();
if(cheeses != null && cheeses.contains(Cheese.STILTON))
    System.out.println("좋았어, 바로 그거야.");
```

컬렉션이나 배열 같은 컨테이너가 비었을 때 null 반환하는 메서드를 사용할 때면 항시 이와 같은 방어 코드 넣어줘야 함.

클라이언트에서 방어 코드를 빼먹으면 오류가 발생할 수 있다. 실제로 객체가 0개일 가능성이 거의 없는 상황에서는 수년 뒤에야 오류가 발생하기도 함. 

null 반환하려면 반환하는 쪽에서도 이 상황을 특별히 취급해줘야 해서 코드가 더 복잡해진다.

떄로는 빈 컨테이너를 할당하는 데도 비용이 드니 null을 반환하는 쪽이 낫다는 주장도 있다. 이는 틀린 주장임

첫 번째,  성능 분석결과 이 할당이 성능 저하의 주범이라고 확인되지 않는 한 이 정도의 성능 차이는 신경 쓸 수준이 못 된다. 두 번째, 빈 컬렉션과 배열은 굳이 새로 할당하지 않고도 반환할 수 있다.

빈 컬렉션을 반환하는 전형적인 코드로 대부분의 상황에서는 아래처럼 하면 됨

## **빈 컬렉션을 반환하는 올바른 예**
```java
public List<Cheeses> getCheeses() {
    return new ArrayList<>(cheesesInStock);
}
```

사용 패턴에 따라 빈 컬렉션 할당이 성능을 눈에 띄게 떨어뜨릴 수 있는 확률 있음.

해법은 불변 컬렉션을 반환

## **코드 54-3 최적화- 빈 컬렉션을 매번 새로 할당하지 않도록 했다.**
```java
public List<Cheese> getCheeses() {
    return cheesesInStock.isEmpty() ? Collections.emptyList() : new ArrayList<>(cheesesInStock);
}
```

절대 null 반환하지 말고 길이가 0인 배열을 반환하라 

보통은 단순히 정확한 길이의 배열을 반환하기만 하면 된다. 

그 길이가 0일 수도 있을 뿐이다. 다음 코드에서 toArray 메서드에 건넨 길이 0짜리 배열은 우리가 원하는 반환 타입(이 경우엔 Cheese[])을 알려주는 역할을 한다.

## 코드 54-4 길이가 0일 수도 있는 배열을 반환하는 올바른 방법
```java
public Cheese[] getCheeses() {
    return cheesesInStock.toArray(new Cheese[0]);
}
```

이 방식은 성능을 떨어뜨릴 것 같다면 길이 0짜리 배열을 미리 선언해두고 매번 그 밸열을 반환하면 된다. 길이 0인 배열은 모두 불변이기 때문이다. 

## **코드 54-5 최적화 - 빈 배열을 매번 새로 할당하지 않도록 했다.**
```java
private static final Cheese[] EMPTY_CHEESE_ARRAY = new Cheese[0];

public Cheese[] getCheeses() {
    return cheesesInStock.toArray(EMPTY_CHEESE_ARRAY);
}
```

## **코드 54-6 나쁜 예 - 배열을 미리 할당하면 성능이 나빠진다.**
```java
return cheesesInStock.toArray(new Cheese[cheesesInStock.size()]);
```