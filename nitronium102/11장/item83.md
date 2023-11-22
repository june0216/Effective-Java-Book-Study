### 11장 : 동시성
# 🍀 [아이템 83] 지연 초기화는 신중히 사용하라
 
## 📒 지연 초기화 lazy initialization
- 필드의 초기화 시점을 그 값이 처음 필요할 때까지 늦추는 기법
- 값이 쓰이지 않으면 초기화도 절대 일어나지 않는다
- 클래스와 인스턴스 초기화 때 발생하는 위험한 순환 문제를 해결할 수 있다

### 최적화에서의 지연 초기화
- 클래스 혹은 인스턴스 생성 시의 초기화 비용은 감소한다
- 하지만 지연 초기화되는 필드에 접근하는 비용은 커진다
=> 지연 초기화 필드 중 결국 초기화가 이뤄지는 비율, 실제 초기화에 드는 비용, 초기화된 각 필드의 호출 빈도에 따라 성능 판단

### 지연 초기화가 필요한 시점
- 해당 클래스의 인스턴스 중 그 필드를 사용하는 인스턴스의 비율이 낮고, 그 필드를 초기화하는 비용이 클 때
=> 지연 초기화 적용 전후의 성능을 측정

### 멀티 스레드 환경에서의 지연 초기화
지연 초기화 필드를 둘 이상의 스레드가 공유한다면 반드시 동기화!

## 📒 초기화 방법
```java
public class FieldType {
}

public class Initialization {
  private static FieldType computeFieldValue() {
    return new FieldType();
  }
}
```

### 📃 인스턴스 필드의 일반적인 초기화
> 대부분의 상황에서 일반적인 초기화가 지연 초기화보다 낫다
```java
private final FieldType field1 = computeFieldValue();
```

### 📃 인스턴스 필드의 지연 초기화
> 지연 초기화가 초기화 순환성을 깨뜨릴 것 같으면 synchronized를 단 접근자 사용

**초기화 순환성[ initialization circularity ]**
Class A in its constructor creates instance of class B, class B creates instance of class C, and class C creates instance of class A.

```java
private FieldType field2;
private synchronized FieldType getField2() {
  if (field2 == null)
    field2 = computeFieldValue();
  return field2;
}
```

### 📃 정적 필드용 [지연 초기화 홀더 클래스] 관용구
> 성능 때문에 정적 필드를 지연 초기화하는 경우
- 클래스가 처음 쓰일 때 비로소 초기화
- getField 메서드가 필드에 접근하면서 동기화를 전혀 하지 않기 때문에 성능이 느려지지 않음

```java
private static class FieldHolder {
  static final FieldType field = computeFieldValue();
}

private static FieldType getField() { return FieldHolder.field; }
```

- 일반적인 VM은 클래스를 초기화할 때만 필드 접근을 동기화
- 클래스 초기화가 끝난 후에는 VM이 동기화 코드를 제거하여, 그 다음부터는 아무런 검사나 동기화없이 필드에 접근

### 📃 인스턴스 필드 지연 초기화용 [이중검사] 관용구
> 성능 때문에 인스턴스 필드를 지연 초기화하는 경우

- 초기화된 필드에 접근할 때의 동기화 비용을 없애준다
- 첫 번째 검사 : 동기화 없이 검사
- 두 번째 검사 : 동기화하여 검사 (필드가 아직 초기화되지 않은 경우에만 수행)
-> 두 번째 검사에서도 필드가 초기화되지 않았다면 그제서야 초기화

- 필드가 초기화된 이후로는 동기화하지 않으므로 반드시 volatile으로 선언

```java
private volatile FieldType field4;

private FieldType getField4() {
  FieldType result = field4;
  if (result != null)    // 첫 번째 검사 (락 사용 안 함)
    return result;

  synchronized(this) {
    if (field4 == null) // 두 번째 검사 (락 사용)
      field4 = computeFieldValue();
    return field4;
  }
}
```

### 📃 [단일검사] 관용구
반복해서 초기화해도 상관없는 인스턴스 필드를 지연 초기화하는 경우에 사용

```java
private volatile FieldType field5;

private FieldType getField5() {
  FieldType result = field5;
  if (result == null)
    field5 = result = computeFieldValue();
  return result;
}

private static FieldType computeFieldValue() {
  return new FieldType();
}
```

### 📃 [짜릿한 단일검사] 관용구
- 모든 스레드가 필드의 값을 다시 계산해도 상관없고, 필드의 타입이 long과 double을 제외한 다른 기본 타입인 경우
- 단일 검사의 필드 선언에서 volatile을 없앴다
```java
private FieldType field5;

private FieldType getField5() {
  FieldType result = field5;
  if (result == null)
    field5 = result = computeFieldValue();
  return result;
}

private static FieldType computeFieldValue() {
  return new FieldType();
}
```