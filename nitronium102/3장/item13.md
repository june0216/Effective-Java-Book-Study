### 3장 : 모든 객체의 공통 메서드
# 🍀 [아이템 13]  `clone` 재정의는 주의해서 진행하라

# 📒 clone()
> 객체의 모든 필드를 복사하여 새로운 객체에 넣어 반환하는 동작을 수행. 즉, 필드 값이 같은 객체를 새로 만드는 것

### 📃 얕은 복사
clone으로 객체를 복제하는 경우 원본 객체와 복제된 객체가 같은 객체를 공유하므로 둘 중 하나만 변경되어도 두 객체가 모두 바뀐다. 이는 완전한 복제라고 볼 수는 없으며 이런 복제를 얕은 복사(shallow copy) 라고 한다.

얕은 복사 는 복제된 인스턴스가 메모리에 새로 생성되지 않는다. 값 자체를 복사하는 것이 아니라 주소값을 복사하여 같은 메모리를 가리키도록 한다.

새로 인스턴스를 생성하지 않기 때문에 깊은 복사(Deep copy) 보다 상대적으로 빠르다. 참조 타입(reference type) 을 복사하는 경우 얕은 복사가 일어난다.

# 📒 Clonable
> 어떤 객체가 복제(clone)을 허용한다는 사실을 알리기 위해서 만들어진 믹스인 인터페이스이다. 
&nbsp;


### 📃 믹스인 인터페이스
> 다른 클래스에서 사용할 목적으로 만들어진 클래스

클래스가 본인의 기능 이외에 추가로 구현할 수 있는 자료형으로, 어떤 선택적 기능을 제공한다는 사실을 선언하기 위해 쓰인다.
https://jake-seo-dev.tistory.com/30

- `Object` 클래스에 `protected clone()` 이라는 메서드 있음
- `Clonable` 인터페이스는 `clone()`의 동작 방식을 결정
- `Clonable` 인터페이스를 구현하지 않은 인스턴스에서 `clone()`을 호출하면 `CloneNotSupportedException`을 던짐

&nbsp;

### 📃 `clone()` 사용 예시
#### 에러 코드 
```java
static class Entry {
    String key;
    String value;

    public Entry(String key, String value) {
        this.key = key;
        this.value = value;
    }

    @Override
    protected Object clone() throws CloneNotSupportedException {
        return super.clone();
    }
}
```

```java
java.lang.CloneNotSupportedException: item13.Item13Test$Entry

    at java.base/java.lang.Object.clone(Native Method)
    at item13.Item13Test$Entry.clone(Item13Test.java:17)
    at item13.Item13Test.entryCloneTest(Item13Test.java:24)
```
&nbsp;

#### 정상 코드
```java 
public class Item13Test {
    static class Entry implements Cloneable {
        String key;
        String value;

        public Entry(String key, String value) {
            this.key = key;
            this.value = value;
        }

        @Override
        protected Object clone() throws CloneNotSupportedException {
            return super.clone();
        }
    }

    @Test
    public void entryCloneTest() throws CloneNotSupportedException {
        Entry entry = new Entry("key", "value");
        System.out.println("entry.key = " + entry.key);
        System.out.println("entry.value = " + entry.value);

        Entry clonedEntry = (Entry) entry.clone();
        System.out.println("clonedEntry.key = " + clonedEntry.key);
        System.out.println("clonedEntry.value = " + clonedEntry.value);
    }
}
```

&nbsp;


# 📒 `Clonable`의 문제점
- 일반적인 인터페이스의 동작 방식과는 다르게 상위 `Object`클래스에 `protected` 접근자로 된 `clone()` 메서드가 존재하고, 그걸 오버라이드해야 한다
(믹스인으로 의도했지만, 믹스인이라고 말하기 애매함)
- `Clonable`만 사용한다고 해서 복제가 이루어지는 것은 아니다
- 자바의 기본 의도와는 다르게 생성자를 호출하지 않고 객체를 생성할 수 있게 되어버린다
- `clone()` 메서드의 일반 규약은 약간 허술하다

&nbsp;

# 📒 `clone()` 메서드 구현 방법
### 📃 `clone()` 메서드 일반 규약
- x.clone() != x 식은 참이어야 한다.

복사된 객체가 원본이랑 같은 주소를 가지면 안된다는 뜻이다.
- x.clone().getClass() == x.clone().getClass() 식도 참이어야 한다.

복사된 객체가 같은 클래스여야 한다는 뜻이다.
- x.clone().equals()는 참이어야 하지만, 필수는 아니다.

복사된 객체가 논리적 동치는 일치해야 한다는 뜻이다. (필수는 아니다.)

### 📃 `clone()` 메서드는 `super.clone()`을 사용하는 편이 좋다
- 이걸 사용하지 않으면, 상속한 하위 클래스에서 super.clone()을 호출했을 때 엉뚱한 결과가 나올 수 있다
- 단, final 클래스라면 이런 걱정할 필요 X

### 📃 구현 순서
1. `super.clone()` 호출
- 정의된 모든 필드는 원본 필드와 똑같은 값을 갖게 된다
- 모든 필드가 `primitive` 타입이나 `final` 이라면 여기에서 종료
```java
@Override
public PhoneNumber clone() {
  try {
    return (PhoneNumber) super.clone();
  } catch (CloneNotSupportedException e) {
    throw new AssertionError(); // 일어날 수 없는 일이다.
  }
}
```
2. 해당 클래스 선언에 `Cloneable`을 구현한다고 선언하기

&nbsp;
# 📒 가변 객체를 참조할 때의 `clone()` 메서드 구현
> 가변 객체 : `instance` 생성 이후에도 내부 상태 변경이 가능한 객체
```java
static class Stack {
    private Object[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    public Stack() {
        this.elements = new Object[DEFAULT_INITIAL_CAPACITY];
    }

    public void push(Object e) {
        ensureCapacity();
        elements[size++] = e;
    }

    public Object pop() {
        if(size == 0) {
            throw new EmptyStackException();
        }

        Object result = elements[--size];
        elements[size] = null;
        return result;
    }

    private void ensureCapacity() {
        if (elements.length == size) {
            elements = Arrays.copyOf(elements, 2 * size + 1);
        }
    }
}
```

해당 클래스를 그대로 복제하게 되면 `primitive` 타입의 값은 올바르게 복제되지만, 복제된 `Stack`의 `element`는 복제 전의 `Stack`과 같은 배열을 가리키게 됨<br>
-> 원본이나 복제본 중 하나를 수정함녀 다른 하나도 수정되어 불변성을 해침

### 📃 해결 방법
> 가변 객체인 `element` 배열에 `clone()` 메서드를 이용해 따로 복사
```java
@Override
protected Object clone() throws CloneNotSupportedException {
    Stack clonedStack = (Stack) super.clone();
    clonedStack.elements = this.elements.clone();

    return clonedStack;
}
```

배열의 clone은 런타입 타입과 컴파일 타입 모두가 원본 배열과 똑같은 배열을 반환<br>
-> 배열을 복제할 때는 배열의 clone 메서드를 사용하라고 권장

하지만 elements 필드가 `final`이면 해당 방식은 사용할 수 없다.
이는 "Cloneable 아키텍처는 "가변 객체를 참조하는 필드는 final로 선언하라" 는 일반 용법과 충돌한다. 따라서 복제할 수 있는 클래스를 만들기 위해 일부 필드에서 final을 제거해야할 수도 있다.

&nbsp;

# 📒 가변 객체 내부에 또 다른 가변 객체가 있을 때의 `clone()` 메서드
```java
static class HashTable implements Cloneable {
    private Entry[] buckets;

    private static class Entry {
        final Object key;
        Object value;
        Entry next;

        public Entry(Object key, Object value, Entry next) {
            this.key = key;
            this.value = value;
            this.next = next;
        }
    }

    @Override
    public HashTable clone() throws CloneNotSupportedException { // 복제본
        HashTable result = (HashTable) super.clone();
        result.buckets = buckets.clone();
        return result;
    }
}
```

복제본은 자신만의 버킷 배열을 갖지만, 이 배열은 원본과 같은 연결리스트를 참조하고 있음

### 📃 해결 방법
> 각 버킷을 구성하는 연결리스트 복사하기

```java
static class HashTable implements Cloneable {
    private Entry[] buckets;

    private static class Entry {
        final Object key;
        Object value;
        Entry next;

        public Entry(Object key, Object value, Entry next) {
            this.key = key;
            this.value = value;
            this.next = next;
        }

        Entry deepCopy() {
            Entry result = new Entry(key, value, next);
            for(Entry p = result; p.next != null; p = p.next) {
                p.next = new Entry(p.next.key, p.next.value, p.next.next);
            }
            return result;
        }
    }

    @Override
    public HashTable clone() throws CloneNotSupportedException {
        HashTable result = (HashTable) super.clone();
        result.buckets = new Entry[buckets.length];

        for (int i=0; i<buckets.length; i++) {
            if(buckets[i] != null) {
                result.buckets[i] = buckets[i].deepCopy();
            }
        }

        return result;
    }
}

```
HashTable.Entry 는 `깊은복사(deep copy)` 를 지원하도록 deepCopy에서 값만 복사해 만들어주고있다. HashTable의 clone은 적절한 크기의 새로운 버킷 배열을 할당한 다음 원래의 버킷 배열을 순회하며 비어있지 않은 각 버킷에 대해 깊은 복사를 수행한다.

하지만 연결리스트를 복제하는 방법으로 재귀적 호출을 선택하는것이 좋은 방법은 아니다. 재귀 호출 떄문에 리스트의 원소 수만큼 스텍 프레임을 소비하여 리스트가 길면 스택 오버플로우를 일으킬 수 있다.

이 문제를 해결하려면 deepCopy를 재귀 호출대신 반복자를 사용하여 순회하는 방향으로 수정해야한다.

```java
Entry deepCopy() {
            Entry result = new Entry(key, value, next);
            for (Entry p = result; p.next != null; p = p.next) {
                p.next = new Entry(p.next.key, p.next.value, p.next.next);
            }

            return result;
}

```

&nbsp;

# 📒 `clone()` 메서드 주의사항
- 상속용 클스에서는 Cloneable을 구현해서는 안된다.

clone 메서드를 재정의해 CloneNotSupportedException()을 던지게하자.
- Object의 clone메서드는 동기화를 신경쓰지않았다.

동시성 문제가 발생할 수 있다.
- 재정의한 clone메서드는 throws 절을 없애야 한다.

사용의 편의성 때문.

&nbsp;

# 이 모든 작업이 꼭 필요한가?
대부분의 상황은 이처럼 복잡하지 않다
- Cloneable을 이미 구현한 클래스를 확장하는 경우 : 어쩔 수 없이 clone 작동하게 하기
- 그렇지 않은 상황인 경우 : 복사 생성자와 복사 팩터리로 객체 복사하기

&nbsp;

# 📒 복사 생성자와 복사 팩토리로 `clone()` 구현하기
> 복사 생성자 : 단순히 자신과 같은 클래스의 인스턴스를 인수로 받는 생성자를 말한다.
```java
/**
 * Constructs a new {@code HashMap} with the same mappings as the
 * specified {@code Map}.  The {@code HashMap} is created with
 * default load factor (0.75) and an initial capacity sufficient to
 * hold the mappings in the specified {@code Map}.
 *
 * @param   m the map whose mappings are to be placed in this map
 * @throws  NullPointerException if the specified map is null
 */
public HashMap(Map<? extends K, ? extends V> m) {
    this.loadFactor = DEFAULT_LOAD_FACTOR;
    putMapEntries(m, false);
}
```
위는 HashMap에서 제공하는 복사 생성자로 볼 수 있다.
사실 이 방식이 clone()보다 나은 면이 많다.
- 생성자를 쓰지 않는 생성방식을 쓰지 않는다.
- 정상적 final 필드 용법과 충돌하지 않는다.
- 불필요한 검사 예외를 던지지 않는다.
- 형변환도 필요 없다.
- '인터페이스' 타입의 인스턴스도 인수로 받을 수 있다.

더 정확한 이름은 변환 생성자(conversion constructor)와 변환 팩터리(conversion factory)이다.

# 정리
- 인터페이스를 만들 때는 절대 Cloneable을 확장해선 안된다.
- Cloneable은 클래스의 믹스인(사용) 의도로 만들어진 것이다.
- final 클래스라면 Cloneable을 구현해도 위험은 크지 않지만, 성능 최적화 관점에서 검토 후에 드물게 허용해야 한다.
- 복제 기능은 생성자와 팩터리를 이용하는 것이 최고이다.
단 한가지 예외는 배열을 복사할 때이다.