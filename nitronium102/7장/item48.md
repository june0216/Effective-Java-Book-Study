### 7장 : 람다와 스트림
# 🍀 [아이템 48] 스트림 병렬화는 주의해서 적용하라

## 📒 자바 언어와 동시성
> 자바는 동시성 프로그램의 선두주자
- 1996년부터 스레드, 동기화, wait/notify를 지원
- 자바 5부터 java.util.concurrent, Executor 등을 선도적으로 지원했다.
- 자바 7부터 fork/join 패키지를 추가
- 자바 8부터 병렬 스트림을 지원
- 스트림에서는 parallel()을 통해 손쉽게 동시성을 제공했다.

### 주의점
> 안전성 safety과 응답 가능 liveness 상태를 유지해라!

&nbsp;

## 📒 예제 : 메르센 소수 구하기
> 메르센 소수 : 2^n - 1로 표현되는 소수

```java
@Test
public void mersenne() {
    primes()
            .map(p -> TWO.pow(p.intValueExact()).subtract(ONE))
            .filter(mersenne -> mersenne.isProbablePrime(50))
            .limit(20)
            .forEach(mp -> System.out.println(mp.bitLength() + ": " + mp));
}
```

```java
@Test
    public void mersenne() {
        primes()
                .parallel()
                .map(p -> TWO.pow(p.intValueExact()).subtract(ONE))
                .filter(mersenne -> mersenne.isProbablePrime(50))
                .limit(20)
                .forEach(mp -> System.out.println(mp.bitLength() + ": " + mp));
    }
```

자바는 pipeline을 병렬화할 방법을 찾아내지 못했기 때문에 연산이 끝나지 않는다

환경이 아무리 좋더라도 `데이터 소스가 Stream.iterate 거나 중간 연산으로 limit 을 쓰면` 병렬화로 성능 개선을 기대할 수 없다.

파이프라인 병렬화는 limit 이 있을 때, CPU 코어가 남는다면 원소를 몇개 더 처리한 후 제한된 개수 이후의 결과를 버려도 아무런 해가 없다고 가정한다.

=> 계속 버려지기 때문에 계속 이전까지의 값을 다시 구해야 한다.

&nbsp;

## 📒 병렬화 `parallel`을 적용하는 기준
> 스트림의 소스가 ArrayList, HashMap, HashSet, ConcurrentHashMap의 인스턴스거나, 배열, int 범위, long 범위 등 쪼개기 쉬울 때 병렬화의 효과가 가장 좋다.

- 데이터를 원하는 크기로 정확하고 손쉽게 나눌 수 있어서 스레드에 분배하기 좋다.
- 위 자료구조는 참조 지역성이 뛰어나다. 이웃한 원소의 참조들이 메모리에 연속해서 저장되어 있다. 이 경우 성능이 좋아진다.
- 기본타입 배열이 참조 지역성이 제일 좋다.
- 참조 지역성이 낮으면, 스레드는 데이터가 주 메모리에서 캐시 메모리로 전송되어 오기를 기다리며 시간을 보내게 된다.

&nbsp;

## 📒 종단 연산과 병렬 수행 효율
> 병렬 연산에 가장 적합한 것은 축소(reduction)이다.

이도 역시 같은 원리를 따라서 쪼개기 쉬울수록 병렬화의 효과가 좋다.


- 특정 집계 연산, 조건에 맞으면 반환하는 등의 메서드 (min, max, count, sum) 는 병렬화하기 좋다.
- 조건에 맞으면 바로 반환하는 메서드들 (anyMatch, allMAtch, noneMatch) 도 적합하다.

조건에 맞으면 parallel() 호출 하나로 거의 프로세서 코어 수에 비례하는 성능 향상을 맛볼 수 있다.

**스트림을 잘못 병렬화하면 성능이 나빠질 뿐만 아니라 결과 자체가 잘못되거나 예상 못한 동작이 발생할 수 있다.**

결과가 잘못되거나 오동작하는 것을 안전 실패 (safety failure) 라고 한다.

안전 실패가 일어나지 않게 만들기 위해서는 Stream 의 명세를 잘 지켜야 한다. 이를테면 accumulator 와 combiner 함수는 반드시 결합 법칙을 지켜야 하며, 간섭받지 않아야 하고, 상태를 갖지 않아야 한다.