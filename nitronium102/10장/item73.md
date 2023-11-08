### 10장 : 예외
# 🍀 [아이템 73] 추상화 수준에 맞는 예외를 던져라

## 📒 예외 번역
메서드가 저수준 예외를 처리하지 않고 바깥으로 전파하면 수행하려는 일과 관련 없는 예외가 튀어나오게 된다

=> 상위 계층에서는 저수준 예외를 잡아 자신의 추상화 수준에 맞는 예외로 바꿔 던져야 한다

```java
try {

} catch (LowerLevelException e) {
    throw new HigherLevelException();
}
```

```java
public E get(int index) {
    ListIterator<E> i = listIterator(index);
    try {
        return i.next();
    } catch (NoSuchElementException e) {
        throw new IndexOutOfBoundsException("인덱스" + index);
    }
}
```

## 📒 예외 연쇄
> 문제의 근본 원인cause인 저수준 예외를 고수준 예외에 실어 보내는 방식

- 저수준 예외가 디버깅에 도움이 된다면 예외 연쇄를 사용
- 별도의 접근자 메서드(`Throwable`의 `getCause` 메서드)를 통해 필요하다면 언제든 저수준 예외를 꺼내볼 수 있다.

```java
Class HigherLevelException extends Exception {
    HigherLevelException(Throwable cause) {
        super(cause);
    }
}
```

## 📒 결론
- 무턱대로 예외를 전파하는 것보다는 예외 번역이 우수하지만, 그렇다고 해서 남용하는 것은 좋지 않다
- 가능하다면 저수준 메서드가 반드시 성공하게 하려 아래 계층에서는 예외가 발생하지 않도록 해야 한다
- 상위 계층에서는 `java.util.Logging`을 사용하여 조용히 예외를 처리할 수 있다.