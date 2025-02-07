## **item58: 전통적인 for문 보다는 for-each문을 사용하라**

## **핵심정리**

아이템 45에서 이야기했듯, 스트림이 제격인 작업이 있고 반복이 제격인 작업이 있다. 

## **코드 58-1 컬렉션 순회하기 - 더나은 방법이 있음**
```java
for(Iterator<Element> i = c.iterator(); i.hasNext(); ) {
    Element e = i.next();
    ... // e로 무언가를 한다.
}
```

다음은 전통적인 for문으로 배열을 순회하는 코드이다.

## **코드 58-2 배열 순회하기 - 더 나은 방법이 있다.**
```java
for(int i=0; i< a.length; i++) {
        ... // a[i]로 무언가를 한다.
}
```

## 코드 58-3 컬렉션과 배열을 순회하는 올바른 관용구 
```java
for(Element e : elements) {
        ... // e로 무언가를 한다.
}
```

여기서 콜론(:) 안의(in) 라고 읽으면 된다. 따라서 이 반복문은 "elements 안의 각 원소 e에 대해"라고 읽는다. 


## 코드 58-4 버그를 찾아보자.
```java
enum Suit { CLUB, DIAMOND, HEARD, SPADE }
enum Rank { ACE, DEUCE, THREE, FOUR, FIVE, SIX, SEVEN, EIGHT, NINE, TEN, JACK, QUEEN, KING }

static Collection<Suit> suits = Arrays.asList(Suit.values());
static Collection<Rank> ranks = Arrays.asList(Rank.values());

List<Card> deck = new ArrayList<>();

    for(Iterator<Suit> i = suits.iterator(); i.hasNext(); )
        for(Iterator<Rank> i = suits.iterator(); i.hasNext(); )
            deck.add(new Card(i.next(), j.next()));
```

## 코드 58-5 같은 버그, 다른 증상!
```java
enum Face { ONE, TWO, THREE, FOUR, FIVE, SIX }

Collections<Face> faces = EnumSet.allOf(Face.class);

for(Iterator<Face> i = faces.iterator(); i.hasNext();) {
    for(Iterator<Face> j = faces.iterator(); j.hasNext(); )    System.out.println(i.next() + " " + j.next());
}
```

## 코드 58-6 문제는 고쳤지만 보기 좋진 않다. 더 나은 방법이 있다.
```java
for(Iterator<Suit> i = suits.iterator(); i.hasNext(); ) {
    Suit suit = i.next();
    for(Iterator<Rank> j = ranks.iterator(); j.hasNext(); )
        deck.add(new Card(suit, j.next()));
}
```

for-each 사용할 수 없는 세 가지 존재
- **파괴적인 필터링(destructive filtering)** - 컬렉션을 순회하면서 선택된 원소를 제거해야 한다면 반복자의 remove 메서드 호출

- **번형(transforming)** - 리스트나 배열을 순회하면서 그 원소의 값 일부 혹은 전체를 교체해야 한다면 리스트의 반복자나 배열의 인덱스 사용

- **병렬 반복(parallel iteration)** - 여러 컬렉션을 병렬로 순회해야 한다면 각각의 반복자와 인덱스 변수를 사용해 엄격하고 명시적으로 제어

