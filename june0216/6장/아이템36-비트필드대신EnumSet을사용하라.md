집합으로 다루고 싶을 때 → 비트 필드 대신 EnumSet을 사용하라 

열거한 값들이 단독이 아닌 집합으로 사용될 경우, 예전에는 각 상수에 서로 다른 2의 거듭제곱 값을 할당한 정수 열거 패턴을 사용해왔다.

## 비트 필드 열거 상수 - 구닥다리 기법!

```java
public class Text {
    public static final int STYLE_BOLD = 1 << 0; // 1
    public static final int STYLE_ITALIC = 1 << 1; // 2
    public static final int STYLE_UNDERLINE = 1 << 2; // 4
    public static final int STYLE_STRIKETHROUGH = 1 << 3; // 8

    // 매개변수 styles는 0개 이상의 STYLE_ 상수를 비트별 OR한 값이다.
    public void applyStyles(int styles) {
        // ...
    }
}
```

- 아래와 같은 식으로 비트별 OR을 사용해 여러 상수를 **하나의 집합**으로 모을 수 있다.
    - → 이렇게 만들어진 집합을 `비트 필드(bit field)`라 한다.

```java
public class Item36 {
    public static void main(String[] args) {
        text.applyStyles(STYLE_BOLD | STYLE_ITALIC);
    }
}
```

- 비트 필드를 사용하면 비트별 연산을 사용해 합집합과 교집합 같은 **집합 연산을 효율적으로 수행**할 수 있다.
    - 하지만! 비트 필드는 정수 열거 상수의 단점을 그대로 지닌다.(대표적으로 타입 안전성 없음)
    - 추가로, 비트 필드 값이 그대로 출력되면 단순한 정수 열거 상수를 출력할 때보다 해석하기가 훨씬 어렵다.
    - 비트 필드 하나에 녹아 있는 모든 원소를 순회하기도 까다롭다.
    - 마지막으로, 최대 몇 비트가 필요한지를 API 작성 시 미리 예측하여 적절한 타입(보통은 int나 long)을 선택해야 한다. API를 수정하지 않고는 비트 수(32비트 or 64비트)를 더 늘릴 수 없기때문이다.

## EnumSet - 비트 필드를 대체하는 현대적 기법

```java
public class Text {
    public enum Style {BOLD, ITALIC, UNDERLINE, STRIKETHROUGH}

    // 어떤 Set을 넘겨도 되나, EnumSet이 가장 좋다.
    public void applyStyle(Set<Style> styles) {
        // ...
    }
}
```

- 비트 필드보다 더 나은 대안으로 **`EnumSet 클래스가`** 있다.
    - 열거형 타입과 함께 사용하기 위한 특별한 Set 구현체이다.
    - java.util 패키지의 EnumSet 클래스는 열거 타입 상수의 값으로 구성된 집합을 효과적으로 표현해준다.
- Set 인터페이스를 완벽히 구현하며, 타입 안전하고, 다른 어떤 Set 구현체와도 함께 사용할 수 있다.
    - 그럼Set이 아니고 EnumSet만의 이점은?
        - **`EnumSet`**은 일반 **`Set`**과는 다르게 열거 타입(**`enum`**) 전용으로 설계되어 있습니다.
        - 성능 =  **EnumSet의 내부는 비트 벡터로 구현**되었다. 이는 매우 작은 메모리 공간을 사용하면서도 효율적인 연산을 수행할 수 있게 합니다.원소가 총 64개 이하라면, 즉 대부분의 경우에 EnumSet 전체를 long 변수 하나로 표현하여 비트 필드에 비견되는 성능을 보여준다.
        - **편의성**: **`EnumSet`**은 열거 타입과 함께 사용될 때 특정한 연산을 편리하게 수행할 수 있는 여러 메소드를 제공합니다. 예를 들어, **`allOf`**, **`noneOf`**, **`of`** 등의 메서드를 통해 쉽게 집합을 초기화할 수 있습니다.
            - removeAll과 retainAll 같은 대량 작업은 (비트 필드를 사용할 때 쓰는 것과 같은) 비트를 효율적으로 처리할 수 있는 산술 연산을 써서 구현했다.
                - 난해한 작업들은 EnumSet이 다 처리해주기 때문에 비트를 직접 다룰 때 겪는 흔한 오류들로부터 해방된다.
        - **명확성**: **`EnumSet`**을 사용하면 해당 집합이 열거 타입에만 제한된다는 것을 코드 수준에서 명확하게 알 수 있습니다. 이는 코드의 가독성을 높이고 잘못된 데이터 타입의 추가를 컴파일 시점에서 방지합니다.
        

```java
public class Item36 {
    public static void main(String[] args) {
        text.applyStyles(EnumSet.of(Style.BOLD, Style.ITALIC));
    }
}
```

- applyStyles 메서드가 `EnumSet<Style>`이 아닌 `Set<Style>`을 받은 것은 다형성 때문이다.
    - 이렇게 함으로써 메서드는 **`EnumSet`** 뿐만 아니라 **`Set`** 인터페이스를 구현한 어떠한 집합 구현체와도 호환됩니다. 즉, 다양한 종류의 **`Set`**을 입력으로 받을 수 있게 되므로 메서드의 유연성이 향상됩니다.
    - 모든 클라이언트가 EnumSet을 건네리라 짐작되는 상황이라도 이왕이면 인터페이스로 받는 게 일반적으로 좋은 습관이다.

## 핵심 정리

- **열거할 수 있는 타입을 한데 모아 집합 형태로 사용한다고 해도 비트 필드를 사용할 이유는 없다.**
- EnumSet 클래스가 비트 필드 수준의 명료함과 성능, 그리고 열거 타입의 장점까지 제공해준다.
- EnumSet의 유일한 단점은 자바 9까지는 아직 불변 EnumSet을 만들 수 없다는 것이다(자바 11까지도 수정되지 않았다).
- 명확성과 성능이 조금 희생되긴 하지만, EnumSet 대신 Collections.unmodifiableSet으로 EnumSet을 감싸 사용할 수 있다.