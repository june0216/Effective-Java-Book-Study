### 11장 : 동시성
# 🍀 [아이템 82] 스레드 안전성 수준을 문서화하라  

## 📒 API 문서에 synchronized가 보이면 thread-safe하다?
> 아니오
- 스레드 안전성에도 여러 종류가 있기 때문에 스레드 환경에서도 안전하게 사용하기 위해서는 스레드 안전성 수준을 명시해야 한다

## 📒 스레드 안전성
> 높은 순서대로 설명
1) 불변(immutable)
- 해당 클래스의 인스턴스는 마치 상수와도 같아서 외부 동기화가 필요 없음.
- ex) String, Long, BigInteger
2) 무조건적인 스레드 안전(unconditionally thread-safe)
- 해당 클래스의 인스턴스는 수정될 수 있음
- 내부에서도 충실히 동기화하여 별도의 외부 동기화없이 동시에 사용해도 안전
- ex) AtomicLong, ConcurrentHashMap
3) 조건부 스레드 안전(conditionally thread-safe)
- 무조건적인 스레드 안전성과 같지만 일부 메서드는 동시에 사용하려면 외부 동기화가 필요
- ex) Collections.synchronized 래퍼 메서드가 반환한 컬렉션
4) 스레드 안전하지 않음(not thread-safe)
- 해당 클래스의 인스턴스는 수정될 수 있음
- 동시에 사용하려면 각각의 메서드 호출을 클라이언트가 선택한 외부 동기화 로직으로 감싸야 함.
- ex) ArrayList, HashMap
5) 스레드 적대적(thread-hostile)
- 외부 동기화로 감싸더라도 멀티스레드 환경에서 안전하지 않음.
- 이러한 클래스는 동시성을 고려하지 않고 만들다보면 우연히 만들어짐.

### 📃 스레드 안전성 에너테이션
- `@Immutable` : 불변
- `@ThreadSafe` : 무조건적인 스레드 안전, 조건부 스레드 안전
- `@NotThreadSafe` : 스레드 안전하지 않음, 스레드 적대적
- 일반적으로 스레드 적대적인 클래스는 문제를 고쳐 재배포하거나 deprecated API로 지정

## 📒 스레드 동기화에 대한 문서화
> 어떤 순서로 호출할 때 외부 동기화가 필요한지, 그리고 그 순서로 호출하려면 어떤 락 혹은 (드물게) 락들을 얻어야 하는지 알려줘야 함

### 📃 Collections.synchronizedMap
반환 타입만으로 명확히 알 수 없는 정적 팩토리 메서드라면 자신이 반환하는 객체에 대한 스레드 안전성을 문서화해야 함
```java
/**
 * It is imperative that the user manually synchronize on the returned
 * map when iterating over any of its collection views
 * 반환된 맵의 콜렉션 뷰를 순회할 때 반드시 그 맵으로 수동 동기화하라
 */
   Map m = Collections.synchronizedMap(new HashMap());
       ...
   Set s = m.keySet();  // Needn't be in synchronized block
       ...
   synchronized (m) {  // Synchronizing on m, not s!
       Iterator i = s.iterator(); // Must be in synchronized block
       while (i.hasNext())
           foo(i.next());
   }
```

## 📒 Lock 제공
> 외부에서 사용할 수 있는 락을 제공하면 클라이언트에서 메서드 호출을 원자적으로 수행할 수 있다. (유연)

### 📃 공개 락의 단점
- 내부에서 처리하는 고성능 동시제어 메커니즘과 혼용할 수 없다 (ConcurrentHashMap 같은 동시성 컬렉션과 사용 불가)
- 클라이언트가 공개된 락을 오래 쥐고 놓지 않는 서비스 거부 공격 수행 가능
- synchronized 메서드도 공개 락에 해당한다

### 📃 비공개 락 객체
```java
// 비공개 락 객체, final 선언! : 락 객체 교체 상황 방지를 위함
private final Object lock = new Object();

public void someMethod() {
  synchronized(lock) {
    // do something
  }
}
```

- 비공개 락 객체 관용구는 무조건적 스레드 안전 클래스에서만 사용 가능하다
- 조건부 스레드 안전 클래스에서는 특정 호출 순서에 필요한 락을 알려주어야 하므로(공개 필요) 비공개 락 객체 관용구를 사용할 수 없다.

## 📒 Lock 필드는 항상 final로 선언하라
- 필드를 final로 선언하면 우연히라도 락 객체가 고체되는 일을 방지해준다
- 일반적인 락이든, java.util.concurrent.locks 패키지에서 가져온 락이든 마찬가지이다.