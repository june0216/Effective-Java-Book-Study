# 문제점

- 아래의 코드들은 while문 보다 낫지만 가장 좋은 방법은 아니다.

**컬렉션 순회하기 - 더 나은 방법 존재**

```java
for (Iterator<Element> i = c.iterator(); i.hasNext(); ) {
    Element e = i.next();
    ...
}
```

**배열 순회하기 - 더 나은 방법 존재**

```java
for (int i = 0; i < a.length; i++) {
    ...
}
```

- 반복자와 인덱스 변수를 코드를 지저분하게하고 오류가 생길 가능성이 높아진다.
- 쓰이는 요소 종류가 늘어나면 오류 생길 가능성이 높아진다.
    - 1회 반복에서 반복자는 세 번 등장하고 인덱스는 네 번 등장하고 있다. 혹시라도 잘못된 변수를 사용했을 때 컴파일러가 잡아주지 못할 수도 있다. 또한 컬렉션이나 배열이냐에 따라 코드 형태도 달라지고 있다.

# for-Each

- for-each문의 정식 이름은 “향상된 for 문이다” (이욜~)
- 장점
    - 반복자와 인덱스 변수를 사용하지 않으니 코드가 깔끔해지고 오류가 날 일도 없다.
    - 하나의 관용구로 컬렉션과 배열 모두 처리할 수 있어서 어떤 컨테이너를 다루는지 신경쓰지 않아도 된다.

```java
for (Element e:  elements) { 
    ...
}
```

- 반복 대상이 컬렉션이든 배열이든, for-each 문을 사용해도 속도는 그대로다.
- 컬렉션을 중첩해 순회해야 한다면 for-each문의 이점은 더욱 커진다.

🐛**버그를 찾아보자-1**

```java
    // 버그를 찾아보자.
    enum Suit { CLUB, DIAMOND, HEART, SPADE }
    enum Rank { ACE, DEUCE, THREE, FOUR, FIVE, SIX, SEVEN, EIGHT,
        NINE, TEN, JACK, QUEEN, KING }

    static Collection<Suit> suits = Arrays.asList(Suit.values());
    static Collection<Rank> ranks = Arrays.asList(Rank.values());

    Card(Suit suit, Rank rank ) {
        this.suit = suit;
        this.rank = rank;
    }

    public static void main(String[] args) {
        List<Card> deck = new ArrayList<>();

        for (Iterator<Suit> i = suits.iterator(); i.hasNext(); )
            for (Iterator<Rank> j = ranks.iterator(); j.hasNext(); )
                deck.add(new Card(**i.next()**, j.next()));
```

- 문제 = 바깥 컬렉션의 반복자에서 next메서드가 너무 많이 불린다는 것이 문제!
- 마지막 줄의 i.next()가 숫자 하나당 한번씩만 불려야하는데, 안쪽 반복문에서 호출되는 바람에 카드 하나당 한 번씩 불리고 있다.
    - 그래서 숫자가 바닥나면 NoSuchElementException을 던진다.

이 문제를 해결하려면 바깥 반복문에 바깥 원소를 저장하는 변수를 추가해야한다.

🐛**버그를 찾아보자 -2**

```java
enum Face { ONE, TWO, THREE, FOUR, FIVE, SIX }

    public static void main(String[] args) {

        Collection<Face> faces = EnumSet.allOf(Face.class);

        for (Iterator<Face> i = faces.iterator(); i.hasNext(); )
            for (Iterator<Face> j = faces.iterator(); j.hasNext(); )
                System.out.println(i.next() + " " + j.next());
```

```
36개의 조합이 나와야 하는데 다음과 같이 6개의 조합만 나온다. 
ONE ONE
TWO TWO
THREE THREE
FOUR FOUR
FIVE FIVE
SIX SIX
```

- 두 개의 중첩된 **`for`** 루프에서 동일한 **`Iterator`** 인스턴스 **`i`**와 **`j`**가 사용되고 있다는 것입니다. Iterator는 한 번 움직이면 그 상태가 변경되기 때문에, 내부의 **`for`** 루프(**`j`** iterator)가 완전히 순회한 후에는 외부의 **`for`** 루프(**`i`** iterator)도 그 영향을 받아 버립니다. 그 결과로 예상한대로 36가지 조합이 출력되지 않습니다.

**해결 - Better**

```java
```java
enum Face { ONE, TWO, THREE, FOUR, FIVE, SIX }

    public static void main(String[] args) {

        Collection<Face> faces = EnumSet.allOf(Face.class);

        for (Iterator<Face> i = faces.iterator(); i.hasNext(); )
						Suit suit = i.next()
            for (Iterator<Face> j = faces.iterator(); j.hasNext(); )
                deck.add(new Card(suit, j.next()));
```
```

**짜잔!~ - Best!!!**

```java
for (Suit suit: suits)
    for (Rank rank: ranks)
        deck.add(new Card(suit, rank));
```

이렇게 for-each문을 사용하면 간단하게 해결된다.

# for-each 문 사용 불가 상황

“불가능한 상황이라면 for문을 사용하되, 언급한 문제들을 경계하라”

- 파괴적인 필터링

  컬렉션을 순회하면서 선택된 원소를 제거해야 한다면 remove 메서드를 호출해야 한다. 자바 8부터는 Collection의 removeIf 메서드를 사용해 컬렉션을 명시적으로 순회하는 일을 피할 수 있다.

- 변형

  리스트나 배열을 순회하면서 그 원소의 값 일부 혹은 전체를 교체해야 한다면 리스트의 반복자나 배열의 인덱스를 사용해야 한다.

- 병렬 반복

  여러 컬렉션을 병렬로 순회해야 한다면 각각의 반복자와 인덱스 변수를 사용해 명시적으로 제어해야 한다.


### Iterable을 구현해보자

- for- each문은 컬렉션과 배열은 물론 Iterable 인터페이스를 구현한 객체라면 무엇이든 순회할 수 있다.
- 만약 원소들의 묶음을 표현하는 타입을 작성해야한다면 Iterable을 구현하는 것을 고려해보자