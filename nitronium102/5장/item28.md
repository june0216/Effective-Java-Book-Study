### 5장 : 제네릭
# 🍀 [아이템 28]  배열보다는 리스트를 사용하라

## 📒 배열과 제네릭의 두 가지 차이
### 📃 1. 배열은 공변covariant이다.
- 공변 : 함꼐 변한다 (배열)
ex) Sub가 super의 하위 타입인 경우, 배열 Sub[]는 배열 Super[]의 하위 타입이 된다
- 불공변 : 함께 변하지 않는다 (리스트)
ex) TypeA, TypeB가 있을 경우 List<Type1>은 List<Type2>의 하위 타입도 아니고 상위 타입도 아니다.

#### 공변 타입
```java
@Test
public void covariantTest() {
    Object[] objects = new Long[1];
    objects[0] = "String"; // ArrayStoreException
}
```
- 런타임에 `ArrayStoreException`이 발생한다

#### 불공변 타입
```java
@Test
public void invariantTest() {
    List<Object> ol = new ArrayList<Long>(); // 불공변이라 타입 자체가 호환이 안된다. 이미 에러가 뜨고 있다.
    ol.add("스터디");
    // 제네릭의 타입 정보가 런타임에는 이미 사라져버리기 때문에 컴파일 타임에 검사한다.
    // 런타임에 제네릭 타입을 소거시키는 이유는 레거시 코드와 제네릭 타입을 함께 이용할 수 있게 해주기 위해서이다.
}
```
- 제네릭의 경우 모든 관계는 그저 다른 타입이기 때문에 컴파일 시 에러가 발생
- 이미 컴파일 시 에러가 잡히기 때문에 런타임에 심각한 에러로 발생할 가능성이 없다

=> 에러는 컴파일 타임에 잡는 것이 훨씬 이상적이다!

&nbsp;

## 📒 2. 배열은 실체화reify된다
- 배열은 런타임에도 자신이 담기로 한 원소의 타입을 인지하고 확인한다 (실체화된다)
- 제네릭은 타입 정보가 런타임에는 소거된다 (실체화되지 않는다)
    - 원소 타입을 컴파일 타임에만 검사하며 런타임에는 알 수조차 없다

### 📃 배열과 제네릭은 잘 어우러지지 못한다
- 배열은 제네릭 타입, 매개변수화 타입, 타입 매개변수로 사용할 수 없다.
- 코드를 new List<E>[], new List<String>[], new E[] 식으로 작성하면 컴파일할 때 제네릭 배열 생성 오류를 일으킨다.

### 📃 제네릭 배열을 만들지 못하도록 막은 이유는?
> type-safe하지 않기 때문!

#### 제네릭 배열을 생성한다고 가정
```java
@Test
public void genericArrayTest() {
    List<String>[] stringLists = new List<String>[1]; // (1)
    List<Integer> intList = List.of(42); // (2)
    Object[] objects = stringLists; // (3)
    objects[0] = intList; // (4)
    String s = stringLists[0].get(0); // * 문제가 되는 부분
}
```

- (4) : (2)에서 생성한 List<Integer>의 인스턴스를 Object 배열의 첫 원소로 지정한다
    - 제네릭은 소거 방식으로 구현되어서 성공한다
    - 런타임에는 List<Integer> 인스턴스의 타입은 단순히 List<>가 되고, List<Integer>[]의 인스턴스 타입은 List[]가 된다

- (5) : List<String> 인스턴스만 담겠다고 선언한 stringLists 배열에 현재 List<Integer> 인스턴스가 저장되어 있음
    - 배열의 첫 리스트에서 첫 원소를 꺼내야 함
    - 컴파일러는 꺼낸 원소를 자동으로 String으로 형변환하는데, 이 원소는 Integer이기 때문에 런타임 시 `ClassCastException`이 발생한다

=> 이런 일을 막기 위해 제네릭 배열이 생성되지 않도록 (1)에서 컴파일 오류를 내야 한다

&nbsp;

## 📒 실체화 불가 타입
> 실체화되지 않아서 런타임에는 컴파일 타임보다 타입 정보를 적게 가지는 타입
- `E`, `List<E`, `List<String>`같은 타입
- 소거 매커니즘 때문에, 매개변수화 타입 가운데 실체화될 수 있는 타입은 List<?>와 Map<?, ?> 같은 비한정적 와일드카드 타입 뿐이다.
- 배열을 비한정적 와일드카드 타입으로도 만들 수는 있지만 크게 쓸모는 없다.

=> 이러한 이유로 배열과 함께 쓰여서도 안되고, 가변인수 메서드와도 함께 쓰일 때도 배열의 원소가 실체화 불가 타입이라면 경고가 발생한다. <br>
후자의 경우 @SafeVarargs 애너테이션으로 대처할 수 있다.

&nbsp;

## 📒 배열 형변환 시 에러 해결하기
- 배열로 형변환하는 경우, 제네릭 배열 생성 오류나 비검사 형변환 경고가 뜨는 경우 대부분 배열인 E[] 대신 컬렉션인 List<E>를 사용하면 해결된다.
- 코드는 조금 복잡해지고 성능이 안 좋아질 수 있지만, 타입 안정성과 상호 운용성이 좋아진다.

### 📃 Chooser 클래스
```java
public class Chooser {
  private final Object[] choiceArray;

  public Chooser(Collection choices) {
    choiceArray = choices.toArray();
  }

  public Object choose() {
    Random rnd = ThreadLocalRandom.current();
    return choiceArray[rnd.nextInt(choiceArray.length)];
  }
}
```
- `choose()` 메서드 호출 시에 매번 반환된 `Object`를 원하는 타입으로 형변환해야 한다
- 혹시나 타입이 다른 원소가 들어 있었다면 런타임에 형변환 오류가 날 것

```java
public class Chooser<T> {
  private final T[] choiceArray;

  public Chooser(Collection<T> choices) {
    choiceArray = choices.toArray();
  }

  public Object choose() {
    Random rnd = ThreadLocalRandom.current();
    return choiceArray[rnd.nextInt(choiceArray.length)];
  }
}
```
🚨 incompatible types: Object[] cannot be converted to T[]
`collection.toArray()`는 object 타입을 반환하도록 되어 있기 때문에 `(T[])`로 변환해주어야 한다

```java
choiceArray = (T[]) choices.toArray();
```

- 수정 후에도 `unchecked cast` 경고가 뜰 것
- 런타임에는 제네릭 타입 정보가 전부 소거되었기 때문에 이게 안전한 캐스팅인지 런타임에 알 수 없기 때문이다.

#### 배열보다는 `List`를 적극 활용하여 완성시키자
```java
public class Chooser<T> {
  private final List<T> choiceList;

  public Chooser(Collection<T> choices) {
    choiceList = new ArrayList<T>(choices);
  }

  public Object choose() {
    Random rnd = ThreadLocalRandom.current();
    return choiceList.get(rnd.nextInt(choiceArray.length));
  }
}
```

&nbsp;

## 📒 정리
- 배열은 공변이고 실체화된다는 점에서 잘못 사용했을 때 불안정한 코드가 나온다.
- 제네릭은 불공변이고 타입 정보가 소거된다.
- 결과적으로 배열은 런타임에 에러 가능성이 존재하고, 제네릭은 그렇지 않다. 그래서 둘을 섞어 쓰긴 어렵다.
- 둘을 섞어 쓰다가 컴파일 오류를 만나면, 과연 이 작업에 진짜 배열이 필요한지 생각해보고 아니라면 바로 제네릭을 사용하는 리스트로 변경하자.
