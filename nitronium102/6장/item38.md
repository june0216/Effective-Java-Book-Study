### 6장 : 열거 타입과 애너테이션
# 🍀 [아이템 38]  확장할 수 있는 열거 타입이 필요하면 인터페이스를 사용하라

## 📒 열거 타입 확장은 하지 말자
열거 타입은 거의 모든 상황에서 타입 안전 열거 패턴(typesafe enum pattern)보다 우수하다

단점 : 타입 안전 열거 패턴은 확장할 수 있으나 열거 타입은 그렇지 못하다

### 📃 열거 타입을 확장하는 건 좋지 않다
```java
enum Day {
    MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY, SATURDAY, SUNDAY
}
```
이때, "열거 타입을 확장한다"는 것은 이 Day 열거 타입을 상속받아 새로운 열거 타입을 정의하려는 시도를 의미합니다. 예를 들어, 다음과 같이 시도하는 것은 열거 타입을 확장하는 시도입니다.

```java
enum ExtendedDay extends Day {
    HOLIDAY
}
```
- 확장한 타입의 원소는 기반 타입의 원소로 취급하지만, 그 반대는 성립하지 않을 수 있음
- 기반 타입과 확장 타입의 원소를 모두 순회할 방법이 마땅치 않음
- 확장성을 높이려면 고려할 요소가 늘어가 설계와 구현이 복잡하다


&nbsp;

### 📃 타입 안전 열거 패턴과 열거 타입?
**1) 타입 안전 열거 패턴(Type-Safe Enum Pattern):**

> 일반적으로 클래스나 인터페이스를 사용하여 열거 값을 정의하는 방법

열거 값은 보통 정적 상수(public static final)로 정의되며, 이러한 값은 클래스 내부에서만 사용 가능하므로 값이 변경되지 않습니다.
열거 값 간에 비교할 때 == 연산자를 사용할 수 있으며, IDE(Integrated Development Environment)에서 코드 자동완성 및 타입 검사를 지원합니다.
예를 들어, Java에서는 enum 키워드를 사용하여 타입 안전 열거 패턴을 구현할 수 있습니다.
```java
public class MyEnum {
    public static final MyEnum VALUE1 = new MyEnum("TXT");
    public static final MyEnum VALUE2 = new MyEnum("10/13컴백");
}
```
&nbsp;

**2) 열거 타입(Enum Type):**

> 명시적으로 언어에 내장된 타입으로, 열거 상수를 정의하고 사용할 수 있는 특수한 형태의 데이터 타입

열거 상수는 정적 상수로 선언되며, 열거 타입 내에서만 사용 가능합니다.
열거 타입은 열거 상수 간의 비교를 위해 == 연산자를 사용할 수 있으며, 각 열거 상수는 해당 열거 타입의 인스턴스입니다.
예를 들어, Java에서는 enum 키워드를 사용하여 열거 타입을 정의할 수 있습니다.
```java
enum MyEnum {
    VALUE1, VALUE2
}
```
&nbsp;

## 📒 연산 코드(opcode)에는 잘 어울린다!
### 📃 1. 열거 타입이 임의의 인터페이스를 구현하도록 함

```java
public interface Operation {
    double apply(double x, double y);
}
```
연산 코드용 인터페이스를 정의하고, 열거 타입이 이 인터페이스를 구현하게 함 (열거 타입이 해당 인터페이스의 표준 구현체 역할)

```java
public enum BasicOperation implements Operation {
    PLUS("+") {
        public double apply(double x, double y) { return x + y; }
    },
    MINUS("-") {
        public double apply(double x, double y) { return x - y; }
    },
    TIMES("*") {
        public double apply(double x, double y) { return x * y; }
    },
    DIVIDE("/") {
        public double apply(double x, double y) { return x / y; }
    };

    private final String symbol;

    BasicOperation(String symbol) {
        this.symbol = symbol;
    }

    @Override public String toString() {
        return symbol;
    }
}
```

새로운 열거 타입을 정의해 기본 타입인 `BasicOperation`을 대체할 수도 있다

```java
public enum ExtendedOperation implements Operation {
    EXP("^") {
        public double apply(double x, double y) {
            return Math.pow(x, y);
        }
    },
    REMAINDER("%") {
        public double apply(double x, double y) {
            return x % y;
        }
    };
    private final String symbol;
    ExtendedOperation(String symbol) {
        this.symbol = symbol;
    }
    @Override public String toString() {
        return symbol;
    }
}
```

`apply`가 인터페이스인 `Operation`에 선언되어 있으니 해당 인터페이스를 구현하기만 하면 어디에서든 사용 가능하다

&nbsp;
### 📃 2. Class 객체 대신 한정적 와일드카드 타입 `Collection<? extends Operation>`을 넘기기
```java
public static void main(String[] args) {
    double x = Double.parseDouble(args[0]);
    double y = Double.parseDouble(args[1]);
    test(Arrays.asList(ExtendedOperation.values()), x, y);
}
```

```java
private static void test(Collection<? extends Operation> opSet,
                         double x, double y) {
    for (Operation op : opSet)
        System.out.printf("%f %s %f = %f%n",
                x, op, y, op.apply(x, y));
}
```

여러 구현 타입의 연산을 조합해 호출할 수 있게 되었다.
하지만 특정 연산에서는 EnumSet과 EnumMap을 사용하지 못한다

#### 예시
> EnumSet과 EnumMap은 열거 타입(enum)을 기반으로 동작하는 자료 구조

```java
// simpleOperation은 enum 타입이 아님!!
public enum SimpleOperation implements Operation {
    ADD, SUBTRACT, MULTIPLY, DIVIDE
}

public static void main(String[] args) {
    double x = Double.parseDouble(args[0]);
    double y = Double.parseDouble(args[1]);
    
    // 가능: SimpleOperation은 Operation을 구현하므로 사용 가능
    test(Arrays.asList(SimpleOperation.values()), x, y);
    
    // 불가능: EnumSet을 사용할 수 없음
    EnumSet<SimpleOperation> set = EnumSet.of(SimpleOperation.ADD);
}
```
위의 코드에서 test 메서드는 Collection<? extends Operation>을 받아서 호출할 수 있지만, EnumSet은 SimpleOperation 타입을 필요로 하기 때문에 사용할 수 없습니다. 이런 경우에는 EnumSet과 함께 작동할 수 있도록 EnumSet<Operation>이나 EnumMap<Operation, ...>과 같이 완전한 타입을 사용해야 합니다.

따라서, EnumSet과 EnumMap을 사용해야 하는 경우에는 한정적 와일드카드 타입 대신 열거 타입(enum)을 사용해야 하며, 다른 상황에서 한정적 와일드카드 타입을 사용하여 구현의 유연성을 높일 수 있습니다.

#### 단점
같은 interface를 구현하는 구현체 열거 타입끼리는 상속이 불가능하다
 
&nbsp;
# 📒 정리
- 열거 타입 자체는 확장할 수 없지만, 인터페이스와 그 인터페이스를 구현하는 기본 열거 타입을 함께 사용해 같은 효과를 낼 수 있다
- API가 (기본 열거 타입을 직접 명시하지 않고) 인터페이스 기반으로 작성되었다면, 기본 열거 타입의 인스턴스가 쓰이는 모든 곳을 새로 확장한 열거 타입의 인스턴스로 대체해 사용할 수 있다.