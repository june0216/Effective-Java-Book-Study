### 6장 : 열거 타입과 애너테이션
# 🍀 [아이템 39]  명명 패턴보다 애너테이션을 사용하라

## 📒 명명 패턴의 단점
ex) junit3 : 테스트 메서드는 무조건 test로 시작!

- 오타가 나면 안 된다
    - 오타 나면 junit이 무시하고 지나감 -> 개발자는 테스트 통과했다고 오해할 수 잇음
- 올바른 프로그램 요소에서만 사용되리라 보증할 방법이 없다
- 프로그램 요소를 매개변수로 전달할 마땅한 방법이 없다
    - 특정 예외를 던져야 성공하는 테스트 : 방법이 없다!

&nbsp;

## 📒 마커(marker) 애너테이션

### 📃 메타 애너테이션(meta-annotation)
> annotation 선언에 다는 annotation

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface Test {
}
```
- `@Retention(RetentionPolicy.RUNTIME)` : @Test가 런타임에도 유지되어야 한다(보존 정책)
- `@Target(ElementType.METHOD)` : @Test가 메서드 선언에서만 사용되어야 한다(적용 대상)

### 📃 마커 애너테이션(marker annotation)
> 아무 매개변수 없이 단순히 대상에 marking한다

- 대상 코드의 의미는 그대로 둔 채, annotation에 관심 있는 도구에서 특별한 처리를 할 기회를 준다.
- 실제 클래스에 영향을 주지 않지만, annotation에 관심 있는 프로그램에 추가 정보를 제공한다

![Alt text](image.png)

```java
// 코드 39-3 마커 애너테이션을 처리하는 프로그램
import java.lang.reflect.*;

public class RunTests { 
  public static void main(String[] args) throws Exception {
    int tests = 0;
    int passed = 0;
    Class<?> testClass = Class.forName(args[0]);
    for (Method m : testClass.getDeclaredMethods()) {
      if (m.isAnnotationPresent(Test.class)) { // @Test 애너테이션이 있는지 확인
        tests++;
        try {
          m.invoke(null); // 테스트 메서드 실행
          passed++;
        } catch (InvocationTargetException wrappedExc) {
          Throwable exc = wrappedExc.getCause();
          System.out.println(m + " 실패: " + exc);
        } catch (Exception exc) {
          System.out.println("잘못 사용한 @Test: " + m);
        }
      }
    }
    System.out.printf("성공: %d, 실패: %d%n",
                      passed, tests - passed);
  }
}
```
리플렉션을 사용하여 @Test annotation이 달린 메서드 찾기, 원래 예외에 담긴 실패 정보 추출

### 📃 reflection이란?
> 프로그램의 런타임 동작을 조사하고 수정하는데 사용되는 도구

- 런타임에 동적으로 클래스 정보를 조사: 프로그램이 실행 중일 때, 사용자로부터 입력된 클래스 이름에 해당하는 클래스를 동적으로 로드해야 했습니다. 이때 Reflection을 사용하여 클래스 정보를 동적으로 가져올 수 있습니다.

```java
Class<?> testClass = Class.forName(args[0]);
```

- 애너테이션을 사용한 메서드를 찾아 실행: 코드는 주어진 클래스에서 @Test 애너테이션이 부착된 메서드를 찾아서 실행해야 했습니다. Reflection을 사용하여 클래스의 메서드를 동적으로 검사하고 애너테이션을 확인할 수 있습니다.

```java
for (Method m : testClass.getDeclaredMethods()) {
    if (m.isAnnotationPresent(Test.class)) {
        // @Test 애너테이션이 부착된 메서드를 실행
        // ...
    }
}
```
&nbsp;

## 📒 매개변수를 가진 annotation
> 예외를 던져야 하는 test annotation 만들기

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface ExceptionTest {
  Class<? extends Throwable> value();
}
```
- 필드 : annotation에서 사용하는 옵션값을 의미한다
- `Class<? extends Throwable>` : `Throwable`을 확장한 클래스의 Class 객체 - 모든 예외와 오류 타입을 수용함

```java
public class Sample {
    @ExceptionTest(ArithmeticException.class)
    public static void m1() { // 성공
        int i = 0; 
        i = i / 1;
    }

    @ExceptionTest(ArithmeticException.class)
    public static void m2() { // 실패해야 함(예외 발생)
        int[] a = new int[0];
        int i = a[1];
    }


   @ExceptionTest(ArithmeticException.class)
    public static void m3() { // 실패해야 함(예외 발생하지 않음)
    }


}
```
위의 클래스를 테스트하는 도구는 아래와 같다

```java
if (m.isAnnotationPresent(ExceptionTest.class)) {
  tests++;
  try {
    m.invoke(null);
    System.out.printf("테스트 %s 실패: 예외를 던지지 않음%n", m);
  } catch (InvocationTargetException wrappedEx) {
    Throwable exc = wrappedEx.getCause();
    Class<? extends Throwable> excType =
      m.getAnnotation(ExceptionTest.class).value();
    if (excType.isInstance(exc)) {
      passed++;
    } else {
      System.out.printf(
        "테스트 %s 실패: 기대한 예외 %s, 발생한 예외 %s%n",
        m, excType.getName(), exc);
    }
  } catch (Exception exc) {
    System.out.println("잘못 사용한 @ExceptionTest: " + m);
  }
}
```
이 코드는 annotation 매개변수의 값을 추출하여 테스트 메서드가 올바른 예외를 던지는지 확인한다
- 테스트 프로그램이 문제 없이 컴파일 되면 애터네이션 매개변수가 가리키는 예외가 올바른 타입일 것이다.
- 컴파일타임에는 존재했으나 런타임에는 존재하지 않는 경우 : TypeNotPresentException


&nbsp;
## 📒 배열 매개변수를 받는 annotation 타입
```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface ExceptionTest {
    Class<? extends Exception>[] value();
}
```

단일 원소 배열에 최적화되어 있지만, 다른 @ExceptionTest들도 무리없이 수용

```java
@ExceptionTest({IndexOutOfBoundsException.class, NullPointerException.class})
```

```java
if (m.isAnnotationPresent(ExceptionTest.class)) {
  tests++;
  try {
    m.invoke(null);
    System.out.printf("테스트 %s 실패: 예외를 던지지 않음%n", m);
  } catch (Throwable wrappedExc) {
    Throwable exc = wrappedExc.getCause();
    int oldPassed = passed;
    Class<? extends Throwable>[] excTypes =
      m.getAnnotation(ExceptionTest.class).value();

    // for문을 추가해서 throwable 배열을 검증
    for (Class<? extends Throwable> excType : excTypes) {
      if (excType.isInstance(exc)) {
        passed++;
        break;
      }
    }
    if (passed == oldPassed)
      System.out.printf("테스트 %s 실패: %s %n", m, exc);
  }
```

&nbsp;
## 📒 반복 가능한 annotation
> `@Repeatable` : 하나의 프로그램 요소에 여러개의 애너테이션을 달 수 있다.
- java 8부터 가능하다
- 배열 매개변수 사용하는 대신, `@Repeatable` annotation을 추가한다

#### @Repeatable 선언 시 주의점
- `@Repeatable`을 단 애너테이션을 반환하는 '컨테이션 애너테이션' 을 하나 더 정의한다
- `@Repeatable`에 이 컨테이너 애너테이션의 class 객체를 매개변수로 전달해야 한다
- 컨테이너 애너테이션은 내부 애너테이션 타입의 배열을 반환하는 value 메서드를 정의해야한다
- 컨테이너 애너테이션 타입에 적절한 `@Retention`과 `@Target`을 명시해야 한다.

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
@Repeatable(ExceptionTestContainer.class)
public @interface ExceptionTest {
  Class<? extends Throwable> value();
}

@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface ExceptionTestContainer {
  ExceptionTest[] value();
}
```

### 📃 적용 방법
```java
@ExceptionTest(IndexOutOfBoundsException.class)
@ExceptionTest(NullPointerException.class)
public static void doublyBad() {
```

#### @Repeatable 처리 시 주의점
- 여러 개 달면 하나만 달았을 때와 구분하기 위해 해당 '컨테이너' 애너테이션 타입이 적용된다.
- isAnnotationPresent : 반복 가능 애너테이션이 달렸는지 검사 (false : 컨테이너 애너테이션)
- getAnnotationsByType : 반복 가능 애너테이션과 컨테이너 애너테이션을 모두 가져옴
- 따로따로 확인하여 코딩한다

```java
// 해당 부분을 넣어서 검사
if (m.isAnnotationPresent(ExceptionTest.class)
    || m.isAnnotationPresent(ExceptionTestContainer.class)) {
  tests++;
  try {
    m.invoke(null);
    System.out.printf("테스트 %s 실패: 예외를 던지지 않음%n", m);
  } catch (Throwable wrappedExc) {
    Throwable exc = wrappedExc.getCause();
    int oldPassed = passed;
    ExceptionTest[] excTests =
      m.getAnnotationsByType(ExceptionTest.class);
    for (ExceptionTest excTest : excTests) {
      if (excTest.value().isInstance(exc)) {
        passed++;
        break;
      }
    }
    if (passed == oldPassed)
      System.out.printf("테스트 %s 실패: %s %n", m, exc);
  }
```

- 애너테이션을 여러번 달아 코드 가독성을 높였다.
- 애너테이션을 선언하고 처리하는 부분에서 코드양이 늘어난다.
- 처리 코드가 복잡해져 오류가 날 가능성이 커짐을 명시하자

&nbsp;

## 📒 결론
애너테이션이 명명패턴보다 낫다

다른 프로그래머가 소스코드에 추가 정보를 제공할 수 있는 도구를 만드는 일을 한다면 적당한 애너테이션 타입을 제공하자

애너테이션으로 할 수 있는 일을 명명패턴으로 처리할 이유는 없다.

자바 프로그래머라면 예외 없이 자바가 제공하는 애너테이션 타입들은 사용하자