### 5장 : 제네릭
# 🍀 [아이템 29]  이왕이면 제네릭 타입으로 만들라

## 📒 제네릭 타입으로 스택 구현하기
### 📃 Before Generic
```java
static class StackWithArray {
        private Object[] elements;
        private int size = 0;
        private static final int DEFAULT_INITIAL_CAPACITY = 16;

        public StackWithArray() {
            elements = new Object[DEFAULT_INITIAL_CAPACITY];
        }

        public void push(Object e) {
            ensureCapacity();
            elements[size++] = e;
        }

        public Object pop() {
            if (size == 0) {
                throw new EmptyStackException();
            }

            Object result = elements[size];
            elements[size] = null;

            return result;
        }

        private void ensureCapacity() {
            if(elements.length == size)
                elements = Arrays.copyOf(elements, 2 * size + 1);
        }
    }
```
- 제네릭 타입을 사용하지 않고 단순히 `Object`라는 최상위 클래스의 범용성을 이용해 구현하였다

### 📃 After Generic
```java
static class StackWithGeneric<E> {
    private E[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    public StackWithGeneric() {
        // 아래 코드는 에러가 난다.
        elements = new E[DEFAULT_INITIAL_CAPACITY];
    }

    public void push(E e) {
        ensureCapacity();
        elements[size++] = e;
    }

    public E pop() {
        if (size == 0) {
            throw new EmptyStackException();
        }

        E result = elements[size];
        elements[size] = null;

        return result;
    }

    private void ensureCapacity() {
        if(elements.length == size)
            elements = Arrays.copyOf(elements, 2 * size + 1);
    }
}
```
🚨 Generic Array Creation
제네릭 코드는 new 키워드로 새로운 클래스를 생성하지 못한다
(E와 같은 실체화 불가 타입으로는 배열을 만들 수 없다 - item28)

# 📒 제네릭 타입 배열 만들기
## 📃 1. `Object` 생성 후 `(E[])`로 형변환하기
```java
// Object배열 elements는 push로 들어온 E 타입만을 담는다.
// 타입 안정성을 보장하지만, 이 배열의 런타임 타입은 E[]가 아닌 Object[]다.
@SuppressWarnings("unchecked")
public StackWithGeneric() {
    elements = (E[]) new Object[DEFAULT_INITIAL_CAPACITY];
}
```
- Object 배열을 생성한 뒤에 (E[])와 같이 제네릭 배열로 캐스팅
- 컴파일러는 이 타입 변환이 안전한지 알 수 없기 때문에 경고 발생
- 💪하지만 우리는 알 수 있다!💪 (직접 읽으면 됨^.^)
    - push()에서만 유일하게 elements 배열에 무언가를 담는다는 것을 알고 있고, push()는 E 타입만 담으므로 다른 타입이 들어오지 않을 것을 알고 있다.
    - 그러므로 @SupressWarnings 애노테이션을 이용해 비검사 경고를 지워도 된다.
    - @SupressWarnings는 항상 가장 작은 범위로 설정하는 것이 좋다.


## 📃 2. 새 제네릭 타입 변수에 형변환하여 할당하기
```java
public E pop() {
  if(size == 0) {
    throw new EmptyStackException();
  }

  // 새 제네릭 타입 변수에 형변환하여 할당했다.
  @SuppressWarnings("unchecked") E result = (E) elements[--size];

  elements[size] = null; // 다 쓴 참조 해제
  return result;
}
```
⚠ [unchecked] unchecked cast
- E는 실체화 불가 타입이기 때문에 컴파일러는 런타임에 이뤄지는 형변환이 안전한지 증명할 방법이 없다
- 💪이번에도 개발자가 직접 증명하고 `@SuppressWarnings`로 경고를 숨겨야 한다

## 📃 두 가지 방법의 장단점
- `Object` 생성 후 `(E[])`로 형변환하기
    - 형변환을 배열 생성 시 단 한 번만 해주면 됨
- 새 제네릭 타입 변수에 형변환하여 할당하기
    - 배열에서 원소를 읽을 때마다 해주어야 함

=? 첫번째 방식이 더 자주 사용되지만, 배열의 런타임 타입이 컴파일타임 타입과 달라 힙 오염을 일으킨다.

### 힙 오염?
타입 캐스팅 시 컴파일러가 타입 캐스팅 객체를 진짜 캐스팅할 수 있는지 검사하지 않고, 캐스팅했을 때 대입되는 참조변수에 저장할 수 있느냐만 검사하기 때문에 일어나는 현상이다.

컴파일러는 컴파일 이후에 제네릭 타입 파라미터를 전혀 신경쓰지 않는다.
즉, ArrayList<Integer>, ArrayList<String> 모두 컴파일 이후에는 ArrayList<Object>가 된다.
힙 오염은 잠재적으로 ClassCastException을 일으킬 수 있다.

### 핵심 정리
클라이언트에서 직접 형변환하기보다 제네릭 타입을 쓰라.
제네릭 타입으로 배열을 생성할 때는 Object[] 배열 생성 후 (E[])와 같이 캐스팅하거나, 하나씩 E로 캐스팅하자.