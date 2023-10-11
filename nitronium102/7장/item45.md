### 7장 : 람다와 스트림
# 🍀 [아이템 45] 스트림은 주의해서 사용하라

## 📒 스트림의 특징
> 다량의 데이터 처리 작업을 돕기 위한 API
- stream : 데이터 원소의 유한 혹은 무한 시퀀스 제공
- stream pipeline : 원소들로부터 수행하는 연산 단계

- `소스 스트림` -> `중간 연산` -> `종단 연산`으로 구성
    - 각 중간 연산은 스트림을 어떠한 방식으로 `변환 transform` 해준다

- 스트림 파이프라인은 `지연평가` 되며, 평가는 종단 연산이 호출될 때 이루어진다
    - 종단 연산이 쓰이지 않는 데이터 원소는 계산에 쓰이지 않는다
    - 종단 연산이 없는 스트림 파이프라인은 아무 일도 하지 않는 명령어와 같다!

- 기본적으로 메서드 연쇄를 지원하며, 순차적으로 수행된다

### 평가?
> 병렬 처리가 가능한 Stream 형태에서 Stream 이 아닌 다른 형태의 자바 객체로 바꾸는 행위

toArray(), collect(), reduce(), forEach() 같이 Stream 타입이 아닌 자바 객체로 변환하고 이를 반환하는 것을 말한다.

&nbsp;

## 📒 예시 : anagram
> 스펠링의 위치만 바꿔서 다른 단어를 만들 수 있는 단어

ex) stop -> spot

### 📃 1. 스트림을 사용하지 않은 버전
```java
public class Item45Test {
    @Test
    public void anagramTest() {
        List<String> words = new ArrayList<>();
        words.add("stop");
        words.add("spot");
        words.add("trim");
        words.add("meet");
        words.add("ball");
        words.add("free");

        Map<String, Set<String>> groups = new HashMap<>();

        // 그룹핑하기
        for (String word : words) {
            groups.computeIfAbsent(alphabetize(word), (unused) -> new TreeSet<>()).add(word);
        }

        // 필터링하기 : 그룹 내 원소의 숫자가 2개 이상이라면 출력
        for (Set<String> group : groups.values()) {
            if(group.size() >= 2) {
                System.out.println(group.size() + ": " + group);
            }
        }
    }

    // ex) stop -> o, p, s, t => opst
    // ex) spot -> o, p, s, t => opst
    // => {opst, {stop, spot}}
    private String alphabetize(String s) {
        char[] a = s.toCharArray();
        Arrays.sort(a);
        return new String(a);
    }
```

computeIfAbsent 메서드를 사용
- 맵 안에 키가 있는 경우 : 해당 키에 매핑된 값을 반환
- 맵 안에 키가 없는 경우 : 함수값을 계산 -> 매핑 -> 매핑된 값 반환

### 📃 2. 스트림을 과도하게 사용한 버전
```java
@Test
public void anagramTest2() {
    List<String> words = new ArrayList<>();
    words.add("stop");
    words.add("spot");
    words.add("trim");
    words.add("meet");
    words.add("ball");
    words.add("free");

    // alphabetize() 함수를 사용하지 않고 안에서 어떻게든 처리
    // 코드 가독성 BAD
    words.stream().collect(
            Collectors.groupingBy(
                    word -> word.chars().sorted()
                            .collect(StringBuilder::new, (sb, c) -> sb.append((char) c), StringBuilder::append).toString()))
            .values().stream()
            .filter(group -> group.size() >= 2)
            .map(group -> group.size() + ": " + group)
            .forEach(System.out::println);
}

private String alphabetize(String s) {
    char[] a = s.toCharArray();
    Arrays.sort(a);
    return new String(a);
}
```

### 📃 3. 스트림을 적절하게 사용한 버전 👍
```java
@Test
public void anagramTest3() {
    List<String> words = new ArrayList<>();
    words.add("stop");
    words.add("spot");
    words.add("trim");
    words.add("meet");
    words.add("ball");
    words.add("free");

    // alphabetize()를 메서드화시켜 적절한 캡슐화 진행
    words.stream().collect(
            Collectors.groupingBy(this::alphabetize))
            .values()
            // method chaining 형식이 보기 깔끔함
            // for 블럭 여러 개보다 결과가 연계된다는 것이 잘 느껴짐
            .stream()
            .filter(group -> group.size() >= 2)
            .map(group -> group.size() + ": " + group)
            .forEach(System.out::println);
}

```

&nbsp;

## 📒 char를 다룰 때는?
> 스트림을 삼가는 것이 좋다!
```java
"Hello World".chars().forEach(System.out::print);
```
실제로는 721011081081113211911111410810033을 출력한다

```java
"Hello World".chars() // int값 반환
```

```java
"Hello World".chars().forEach(x -> System.out.print((char) x));
```
-> (char)를 통해 캐스팅해야만 문자열이 나오기 때문에 까다롭다<br>
-> char를 이용하면 코드가 더러워질 확률 up

&nbsp;

## 📒 람다/스트림을 사용하면 안 되는 경우
### 람다
- 범위 안의 지역 변수를 읽고 수정해야 하는 경우
    - 람다에서는 final이거나 사실상 final인 변수만 읽을 수 있음
    - 지역 변수를 수정하는 것은 불가능

- return, break, continue 처럼 중간에 작업을 끊어야 하는 경우
- 처리 과정 중 검사 예외를 던져야 하는 경우

### 스트림
- 처리 과정 중 이전 단계의 값에 접근해야 하는 경우
    - 스트림 파이프라인은 작업이 끝나면 원래 값은 잃는 구조
    - 우회해서 처리할 수 있지만 만족스럽지는 않다

&nbsp;

## 📒 람다/스트림을 사용하면 좋은 경우
- 원소들의 시퀀스를 일관되게 변경하는 경우 (`map()`)
- 원소들의 시퀀스를 필터링하는 경우 (`filter()`)
- 원소들의 시퀀스를 하나의 연산을 사용해 결합하는 경우 (`collect()`)
- 원소들의 시퀀스를 컬렉션에 모으는 경우 (`collect()`)
- 원소들의 시퀀스에서 특정 조건을 만족하는 원소를 찾는 경우

&nbsp;