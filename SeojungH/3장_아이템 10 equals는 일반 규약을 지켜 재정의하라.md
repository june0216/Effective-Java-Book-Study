# 아이템 10. equals는 일반 규약을 지켜 재정의하라

#### equals 메서드는 다음 상황 중 하나에 해당한다면 재정의하지 않는 것이 최선이다

- #### 각 인스턴스가 본질적으로 고유하다
  - 값을 표현하는 게 아니라 동작하는 개체 표현하는 클래스 해당
  - 예: thread
  
- #### 인스턴스의 '논리적 동치성(logical equality)'을 검사할 일이 없다
  - 설계자가 판단할 때 클라이언트가 논리적 동치성을 검사하는 방식을 애초에 필요하지 않다고 생각하면 Object의 기본 equals만으로 해결 가능
    
- #### 상위 클래스에서 재정의한 equals가 하위 클래스에도 딱 들어맞는다
  - 대부분의 Set 구현체는 AbstractSet이 구현한 equals를 상속받아 쓰고, List 구현체들은 AbstractList로 부터, Map 구현체들은 AbstractMap으로부터 상속받아 그대로 사용

- #### 클래스가 private이거나 package-private이고 equals 메서드를 호출할 일이 없다
```java
 @Override public boolean equals(Object o) {
  throw new AssertionError(); // 호출 금지!
}
```
#### equals를 재정의해야 할 때

- #### 상위 클래스의 equals가 논리적 동치성을 비교하도록 재정의되지 않았을 때
  - 주로 값 클래스가 여기 해당
    - 값 클래스 : Integer 와 String처럼 값을 표현하는 클래스
    - 값 클래스여도 값이 같은 인스턴스가 둘 이상 만들어지지 않음을 보장하는 인스턴스 통제 클래스라면 equals를 재정의하지 않아도 된다
      - enum이 여기 해당
        
#### equals 메서드를 재정의할 때는 반드시 일반 규약을 따라야 한다
- equals 메서드는 동치관계를 구현하며 다음을 만족한다
  - 반사성(reflexivity) : null이 아닌 모든 참조 값 x에 대해, x.equals(x)는 true다
  - 대칭성(symmetry) : null이 아닌 모든 참조 값 x,y에 대해, x.equals(y)가 true면 y.equals(x)도 true다
  - 추이성(transitivity) : null이 아닌 모든 참조 값 x,y,z에 대해, x.equals(y)가 true이고 y.equal(z)도 true면 x.equals(z)도 true다
  - 일관성(consistency) : null이 아닌 모든 참조 값 x,y에 대해, x.equals(y)를 반복해서 호출하면 항상 true를 반환하거나 항상 false를 반환한다
  - null-아님 : null이 아닌 모든 참조 값 x에 대해, x.equals(null)은 false다
 
- 이 규약을 어기면 프로그램이 이상 동작을 하거나 종료되고, 원인이 되는 코드를 찾기 어려울 것이다
- 동치관계 : 집합을 서로 같은 원소들로 이뤄진 부분집합으로 나누는 연산
- 반사성 : 객체는 자기 자신과 같아야 한다는 뜻
- 대칭성 : 두 객체는 서로에 대한 동치 여부에 똑같이 답해야 한다는 뜻

  ```java
  // 잘못된 코드 - 대칭성 위배
  public final class CaseInsensitiveString {
    private final String s;

    public CaseInsensitiveString(String s) {
      this.s = Objects.requireNonNull(s);
    }

    // 대칭성 위배
    @Override public boolean equals(Object o) {
      if (o instanceof CaseInsensitiveString)
        return s.equalsIgnoreCase(
          ((CaseInsensitiveString) o).s);
      if (o instanceof String) // 한 방향으로만 작동한다
        return s.equalsIgnoreCase((String) o);
      return false;
    }
  ...
  }
  ```

  - CaseInsensitiveString의 equals는 일반 문자열과도 비교를 시도한다
  ```java
  CaseInsensitiveString cis = new CaseInsensitiveString("Polish");
  String s = "polish";
  ```
    - CaseInsensitiveString의 equals는 일반 String을 알고 있지만 String 의 equals는 CaseInsensitiveString의 존재를 모르므로, s.equals(cis)는 false를 반환하여 대칭성을 명백히 위반한다

  ```java
  List<CaseInsensitiveString> list = new ArrayList<>();
  list.add(cis);
  ```

  #### equals 규약을 어기면 그 객체를 사용하는 다른 객체들이 어떻게 반응할지 알 수 없다
  - CaseInsensitiveString의 equals를 String과도 연동하려고 하면 안된다
  ```java
  @Override public boolean equals(Object o) {
    return o instanceof CaseInsensitiveString &&
      ((CaseInsensitiveString) o).s.equalsIgnoreCase(s);
  }
  ```

  - 추이성 : 첫 번째 객체와 두 번째 객체가 같고, 두 번째 객체와 세 번째 객체가 같다면, 첫 번째 객체와 세 번째 객체도 같아야 한다는 뜻
    ```java
    public class Point {
      private final int x;
      private final int y;

      public Point(int x, int y){
        this.x = x;
        this.y = y;
      }

      @Override public boolean equals(Object o){
        if (!(o instanceof Point))
          return false;
        Point p = (Point)o;
        return p.x = x && p.y == y;
      }
    
      ...
    }
    
    ```
    ```java
    public class ColorPoint extends Point{
      private final Color color;

      public ColorPoint(int x, int y, Color color) {
        super(x, y);
        this.color = color;
      }

      ...
    }
    ```

    ```java
    // 잘못된 코드 - 대칭성 위배

    @Override public boolean equals(Object o) {
      if(!(o instanceof ColorPoint))
        return false;
      return super.equals(o) && ((ColorPoint) o).color == color;
    }
    ```
    이 메서드는 일반 Point를 ColorPoint에 비교한 결과와 그 둘을 바꿔 비교한 결과가 다를 수 있다

    ```java
    Point p = new Point(1,2);
    ColorPoint cp = new ColorPoint(1,2, Color.RED);
    ```

    ```java
    // 잘못된 코드 - 추이성 위배
    @Override public boolean equals(Object o) {
      if(!(o instanceof Point))
        return false;

      // o가 일반 Point면 색상을 무시하고 비교한다
      if (!(o instanceof ColorPoint))
        return o.equals(this);

      // o가 ColorPoint면 색상까지 비교한다
      return super.equals(o) && ((ColorPoint) o).color == color;
    }
    ```
    이 방식은 대칭성은 지켜주지만 추이성을 깨버린다

    ```java
    ColorPoint p1 = new ColorPoint(1,2, Color.RED);
    Point p2 = new Point(1,2);
    ColorPoint p3 = new ColorPoint(1,2,Color.BLUE);
    ```
    p1.equals(p2)와 p2.equals(p3)는 true를 반환하는데, p1.equals(p3)가 false를 반환한다
    이 방식은 무한 재귀에 빠질 위험도 있다

    #### 구체 클래스를 확장해 새로운 값을 추가하면서 equals 규약을 만족시킬 방법은 존재하지 않는다

    ```java
    // 잘못된 코드 - 리스코프 치환 원칙 위배
    @Override public boolean equals(Object o) {
      if (o == null || o.getClass() != getClass())
        return false;
      Point p = (Point) o;
      return p.x = x && p.y == y;
    }
    ```
    여기서 equals는 같은 구현 클래스의 객체와 비교할 때만 true를 반환한다

    리스코프 치환 원칙(Liskov substitution principle)에 따르면, 어떤 타입에 있어 중요한 속성이라면 그 하위 타입에서도 마찬가지로 중요하다.
    따라서 그 타입의 모든 메서드가 하위 타입에서도 똑같이 잘 작동해야 한다

  ```java
  public class ColorPoint {
    private final Point point;
    private final Color color;

    public ColorPoint(int x, int y, Color color) {
      point = new Point(x,y);
      this.color = Objects.requireNonNull(color);
    }

    /**
     * 이 ColorPoint의 Point 뷰를 반환한다
     */
    public Point asPoint() {
      return point;
    }

    @Override public boolean equals(Object o) {
        if (!(o instanceof ColorPoint))
          return false;
        ColorPoint cp = (ColorPoint) o;
        return cp.point.equals(point) && cp.color.equals(color);
    }
    ...
  }
  ```



  - 일관성 : 두 객체가 같다면 앞으로도 영원히 같아야 한다
    - 가변 객체는 비교 시점에 따라 서로 다를 수도 같을 수도 있지만 불변 객체는 한번 다르면 끝까지 달라야 한다

#### 클래스가 불변이든 가변이든 equals의 판단에 신뢰할 수 없는 자원이 끼어들게 해서는 안 된다
- equals는 항시 메모리에 존재하는 객체만을 사용한 결정적(deterministic) 계산만 수행해야 한다

  - null-아님 : 이름처럼 모든 객체가 null과 같지 않아야 한다는 뜻
    ```java
    // 명시적 null 검사 - 필요없다
    @Override public boolean equals(Object o) {
        if (o == null)
          return false;
        ...
      }
    ```
    이러한 검사는 필요치 않다. 동치성을 검사하려면 equals는 건네받은 객체를 적절히 형변환한 후 필수 필드 값을 알아내야 한다. 그러려면 형변환에 앞서 instanceof 연산자로 입력 매개변수가 올바른 타입인지 검사해야한다.

    ```java
    @Override public boolean equals(Object o) {
        if (!(o instanceof MyType))
          return false;
        MyType mt = (MyType) o;
        ...
    }
    ```
    equals가 타입을 확인하지 않으면 잘못된 타입이 인수로 주어졌을 때 Class CaseException을 던져서 일반 규약을 위해하게 된다.
    그런데 instanceof는 첫번째 피연산자가 null이면 false를 반환하므로 입력이 null이면 타입 확인 단계에서 false를 반환하기 때문에 null 검사를 명시적으로 하지 않아도 된다

#### 양질의 equals 메서드 구현 방법 단계별 정리
1. == 연산자를 사용해 입력이 자기 자신의 참조인지 확인한다
2. instanceof 연산자로 입력이 올바른 타입인지 확인한다
3. 입력을 올바른 타입으로 형변환한다
4. 입력 객체와 자기 자신의 대응되는 '핵심' 필드들이 모두 일치하는지 하나씩 검사한다

- float와 double을 제외한 기본 타입 필드는 == 연산자로 비교
- 참조 타입 필드는 각각의 equals 메서드로 비교
- float과 double 필드는 각각 정적 메서드인 Float.compare(float, float)와 Double.compare(double, double)로 비교


어떤 필드를 먼저 비교하느냐가 equals의 성능을 좌우하기도 한다
: 최상의 성능을 바란다면 다를 가능성이 더 크거나 비교하는 비용이 싼 필드를 먼저 비교하자

#### equals 구현 이후 -> 대칭적인가? 추이성이 있는가? 일관적인가? 자문하기
- 그리고 단위 테스트를 작성해 돌려보기
- equals 메서드를 AutoValue를 이용해 작성했다면 테스트를 생략해도 괜찮다

```java
// 전형적인 equals 메서드의 예
public final class PhoneNumber {
    private final short areaCode, prefix, lineNum;

    public PhoneNumber(int areaCode, int prefix, int lineNum) {
        this.areaCode = rangeCheck(areaCode, 999, "지역코드");
        this.prefix   = rangeCheck(prefix,   999, "프리픽스");
        this.lineNum  = rangeCheck(lineNum, 9999, "가입자 번호");
    }

    private static short rangeCheck(int val, int max, String arg) {
        if (val < 0 || val > max)
            throw new IllegalArgumentException(arg + ": " + val);
        return (short) val;
    }

    @Override public boolean equals(Object o) {
        if (o == this)
            return true;
        if (!(o instanceof PhoneNumber))
            return false;
        PhoneNumber pn = (PhoneNumber)o;
        return pn.lineNum == lineNum && pn.prefix == prefix
                && pn.areaCode == areaCode;
    }
    ...
}
```

#### equals를 재정의할 땐 hashCode도 반드시 재정의하자

#### 너무 복잡하게 해결하여 들지 말자

#### Object 외의 타입을 매개변수로 받는 equals 메서드는 선언하지 말자

```java
// 잘못된 예 - 입력 타입은 반드시 Object 여야 한다!
public boolean equals(MyClass o) {
 ...
}
```
이 메서드는 Object.equals를 재정의 한 것이 아니라 입력 타입이 Object가 아니므로 재정의가 아니라 다중정의한 것이다
@Override 애너테이션을 일관되게 사용하면 '타입을 구체적으로 명시한' equals 메서드가 하위 클래스에서의 @Override 애너테이션이 긍정오류를 내게 하고 보안 측면에 잘못된 정보를 주는 실수를 예방할 수 있다

```java
// 여전히 잘못된 예 - 컴파일 되지 않음
@Override public boolean equals(MyClass o) {
  ...
}
```

AutoValue 프레임워크를 사용하면 equals를 작성하고 테스트를 대신 해준다
