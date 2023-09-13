### 4장 : 클래스와 인터페이스
# 🍀 [아이템 18]  상속보다는 컴포지션을 사용하라

## 📒 상속의 위험성
 다른 패키지의 구체 클래스를 상속하는 일은 위험성을 가지고 있다.

- 클래스가 다른 클래스를 확장하는 구현 상속은 위험 ⭕
- 클래스가 인터페이스를 구현하거나 인터페이스가 다른 인터페이스를 확장하는 인터페이스 상속 ❌

&nbsp;

## 📒 메서드 호출과 달리 상속은 캡슐화를 깨뜨린다
> 상위 클래스가 어떻게 구현되느냐에 따라 하위 클래스의 동작에 이상이 생길 수 있다.

- 캡슐화 : 타인의 외부에서의 조작에 대비해 외부에서 특정 속성이나 메서드를 사용하지 못하도록 숨겨놓은 것.
- 상속하면 상위 클래스의 구현이 하위 클래스에게 노출되게 됨
- 캡슐화가 깨짐으로써 하위 클래스는 상위 클래스에 강하게 결합, 의존하게 되고 강한 결합, 의존은 변화에 유연하게 대처하기가 힘들다.

```java
package effectivejava.chapter4.item18;
import java.util.*;

// 코드 18-2 래퍼 클래스 - 상속 대신 컴포지션을 사용했다. (117-118쪽)
public class InstrumentedSet<E> extends ForwardingSet<E> {
    private int addCount = 0;

    public InstrumentedSet(Set<E> s) {
        super(s);
    }

    @Override public boolean add(E e) {
        addCount++;
        return super.add(e);
    }

    @Override public boolean addAll(Collection<? extends E> c) {
        addCount += c.size();
        return super.addAll(c);
    }
    public int getAddCount() {
        return addCount;
    }

    public static void main(String[] args) {
        InstrumentedSet<String> s = new InstrumentedSet<>(new HashSet<>());
        s.addAll(List.of("틱", "탁탁", "펑"));
        System.out.println(s.getAddCount());
    }
}
```

이걸 구현하면 잘 동작할 것이라 생각하지만 사실은 아님!

상위 클래스인 HashSet 내부에서 addAll을 사용하는데, 이 때 불리는 add는 하위 클래스에서 재정의됨

-> 따라서 addCount에 값이 중복으로 더해져 최종값이 6으로 늘어남

&nbsp;

### 📃 자기사용(self-use)
> 자신의 다른 부분을 사용하는 행위

- 해당 클래스의 내부 구현 방식을 까보지 않으면 알 수 없음
- 자바 플랫폼의 전반적인 정책인지, 따라서 다음 릴리즈에서도 유지되는지도 알 수 없음

&nbsp;


### 📃 다른 방식으로 재정의를 해도..?
😊 응 다음 릴리즈에서 상위 클래스에 새로운 메서드 추가되면 그만이야~

1. 릴리즈 이후에 상위 클래스에 또 다른 원소 추가 메서드가 생김
2. 하위 클래스에서는 해당 메서드를 재정의하지 않음
3. 관련된 보안 구성 및 동작 방식에서 오류 발생 가능

&nbsp;

### 📃 재정의 안 하고 새로운 메서드 추가하면 되잖아요
😊 응 다음 릴리즈에서 똑같은 이름의 메서드 추가될 수 있어~

1. 다음 릴리즈에서 상위 클래스에 새 메서드 추가
2. 하위 클래스에 추가한 메서드와 시그니처가 같고 반환 타입 다르다면?
3. 해당 클래스는 컴파일조차 되지 않는다
4. 반환 타입까지 같으면 상위 클래스의 새 메서드를 재정의한 것이므로 이전 문제 발생
5. 해당 하위 메서드를 작성할 때는 상위 클래스의 메서드가 존재하지 않았으므로, 상위 클래스의 규약을 만족하지 않을 가능성도 있다. 

&nbsp;

## 📒 상속 대신 컴포지션 composition을 사용하자
### 📃 Composition & 전달 forwarding
- composition : 기존 클래스가 새로운 클래스의 구성요소로 사용된다. 
- forwarding : 새로운 클래스를 만들고 private 필드로 기존 클래스의 인스턴스를 참조하게 한다.

### 📃 장점
- 새 클래스는 기존 클래스의 내부 구현 방식의 영향에서 벗어남
- 기존 클래스에 새로운 메서드가 추가되더라도 전혀 영향받지 않음

&nbsp;

```java
package effectivejava.chapter4.item18;
import java.util.*;

// 코드 18-3 재사용할 수 있는 전달 클래스 (118쪽)
public class ForwardingSet<E> implements Set<E> {
    private final Set<E> s;

    // set을 구현한 객체를 받아 사용할 뿐
    public ForwardingSet(Set<E> s) { this.s = s; }

    public void clear()               { s.clear();            }
    public boolean contains(Object o) { return s.contains(o); }
    public boolean isEmpty()          { return s.isEmpty();   }
    public int size()                 { return s.size();      }
    public Iterator<E> iterator()     { return s.iterator();  }
    public boolean add(E e)           { return s.add(e);      }
    public boolean remove(Object o)   { return s.remove(o);   }
    public boolean containsAll(Collection<?> c)
                                   { return s.containsAll(c); }
    public boolean addAll(Collection<? extends E> c)
                                   { return s.addAll(c);      }
    public boolean removeAll(Collection<?> c)
                                   { return s.removeAll(c);   }
    public boolean retainAll(Collection<?> c)
                                   { return s.retainAll(c);   }
    public Object[] toArray()          { return s.toArray();  }
    public <T> T[] toArray(T[] a)      { return s.toArray(a); }
    @Override public boolean equals(Object o)
                                       { return s.equals(o);  }
    @Override public int hashCode()    { return s.hashCode(); }
    @Override public String toString() { return s.toString(); }
}
```

```java
package effectivejava.chapter4.item18;
import java.util.*;

// 코드 18-2 래퍼 클래스 - 상속 대신 컴포지션을 사용했다. (117-118쪽)
public class InstrumentedSet<E> extends ForwardingSet<E> {
    private int addCount = 0;

    public InstrumentedSet(Set<E> s) {
        super(s);
    }

    @Override public boolean add(E e) {
        addCount++;
        return super.add(e);
    }
    @Override public boolean addAll(Collection<? extends E> c) {
        addCount += c.size();
        return super.addAll(c);
    }
    public int getAddCount() {
        return addCount;
    }

    public static void main(String[] args) {
        InstrumentedSet<String> s = new InstrumentedSet<>(new HashSet<>());
        s.addAll(List.of("틱", "탁탁", "펑"));
        System.out.println(s.getAddCount());
    }
}
```


- InstrumentedSet은 ForwardingSet을 상속하여 생성 <br>
    ForwardingSet은 Set을 구현한 객체를 받아 사용할 뿐이기 때문에 HashSet을 상속했을 때처럼 기존 기능을 재정의한다기보다 앞뒤에 독립적인 새로운 부가기능을 넣는 방식이다.
- HashSet을 상속했을 때는 HashSet에만 카운트 기능을 넣을 수 있었지만, Set 인터페이스를 구현함으로써, TreeSet 등 Set 인터페이스만 구현하면 적용 가능해졌다.
```java
@Test
public void instrumentSetTest() {
    InstrumentedSet<String> s = new InstrumentedSet<>(new TreeSet<>());
    s.addAll(List.of("A", "B", "C"));
    System.out.println("s.getAddCount() = " + s.getAddCount());
}
```

### 📃 데코레이터 패턴
- InstrumentedSet : 래퍼 클래스
- 데코레이터 패턴 : 다른 set에 계속 기능을 덧씌운다

#### 래퍼 클래스의 주의점 (SELF 문제)
> callback 프레임워크와는 어울리지 않는다

- 내부 객체는 자신을 감싸고 있는 래퍼의 존재를 모르니 this(자신) 참조를 넘김
- 콜백 때는 wrapper가 아닌 내부 객체를 호출하게 된다

&nbsp;

## 📒 상속은 is-a 관계일 때만 사용되어야 한다
A가 B의 필수 구성 요소인지, 구현 방법 중 하나인지 생각해보자

### 📃 is-a 관계
> 일반적인 개념과 구체적인 개념의 관계

예시
사람은 동물이다.
소는 동물이다
새는 동물이다.

즉, 일반 클래스를 구체화 하는 상황에서 상속을 사용한다.

## 📒 컴포지션을 써야 하는 상황에서 상속을 쓴다면...
- 내부 구현을 불필요하게 노출함
- 클라이언트에서 상위 클래스를 직접 수정하여 하위 클래스의 불변식을 깨뜨릴 수 있음

### 📃 상속 쓰기 전에 생각했나요?
1. 확장하려는 클래스의 API에 아무런 결함이 없는가?
2. 결함이 있다면, 이 결함이 하위 클래스의 API까지 전파되어도 괜찮은가?

컴포지션으로는 숨길 수 있지만 상속은 상위 클래스의 API와 결함까지도 승계한다