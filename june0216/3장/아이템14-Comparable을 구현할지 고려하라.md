# 아이템 14. Comparable을 구현할지 고려하라

- Comparable 인터페이스의 유일무이한 메서드 compareTo를 알아보자
- Object의 메서드가 아니다.
    - 다른점
        - compareTo는 단순 동차성 비교에 더해 “순서까지 비교 가능”
            - Comparable을 구현했다는 것은 그 클래스의 인스턴스들에는 자연적인 순서가 있다. → `Arrays.sort(a)` 가능
            - 검색, 극단값 계산, 자동 정렬되는 컬렉션 관리도 쉽게 할 수 있다.
        - Generic을 지원(compareTo 메서드의 인수 타입은컴파일 타임에 정해짐)
- 자바 플랫폼 라이브러리의 모든 값 클래스와 열거 타입이 Comparable을 구현한다.
    - 알파벳, 숫자, 연대 같이 순서가 명확한 값 클래스를 작성한다면 반드시 Comparable 인터페이스를 구현하자

## CompareTo 규약

- compareTo 메서드의 일반 규약은 equals의 규약과 비슷하다
    - equals와 달리 compareTo는 타입이 다른 객체를 신경쓰지 않아도 된다.
- 자기 자신(this)이 compareTo에 전달된 객체보다 작으면 음수, 같으면 0, 크다면 양수를 리턴한다.
    - 이 객체와 비교할 수 없는 타입의 객체가 주어지면 ClassCastException 발생
- 반사성, 대칭성, 추이성을 만족해야 한다.
    - 반사성 : 자기 자신과 비교
    - 대칭성 : a.compareTo(b), b.compareTo(a)
    - 추이성 : x.compareTo(y) > 0 && y.compareTo(z) > 0 이면 x.compareTo(z) > 0
- 반드시 따라야 하는 것은 아니지만 compareTo 메서드로 수행한 동치성 테스트의 결과가 equals와 같아야 한다.`x.compareTo(y) == 0이라면 x.equals(y)가 true여야 한다.`
    - 일관되지 않으면 동작은 하지만 해당 컬렉션이 구현한 인터페이스(Collection, Set 혹은 Map)에 정의된 동작과 엇박자를 낼 것이다.

```java
/* BigDecimal 클래스는 compareTo와 equals가 일관되지 않는다 */
BigDecimal n1 = new BigDecimal("1.0");
BigDecimal n2 = new BigDecimal("1.00");

n1.compareTo(n2) -> 두 인스턴스가 똑같기 때문에 0
n1.equals(n2) -> 서로 다르기 때문에 false

```

## CompareTo 구현 방법

- compareTo 메서드의 인수 타입은컴파일 타임에 정해짐
    - 입력 인수 타입을 확인하거나 형변환할 필요가 없다.
- 자연적인 순서를 제공할 클래스에 implements Comparable<T> 을 선언한다.
    - compareTo 메서드를 재정의한다.

- `compareTo` 메서드는 각 필드가 동치인지 비교하는게 아니라 그 순서를 비교한다.
    - 객체 참조 필드를 비교하려면 `compareTo`메서드를 재귀적으로 호출한다.
    - `Comparable`을 구현하지 않은 필드나 표준이 아닌 순서로 비교해야 한다면 비교자를 대신 사용한다.
    - 비교자(Comparator)는 직접 만들거나 자바가 제공하는 것 중에 골라 쓰면 된다.
    - 자바가 제공하는 비교자를 사용하고 있다.
        - CaseInsensitiveString의 참조는 CaseInsensitiveString 참조와만 비교할 수 있으므로 Comparable을 구현할 때 일반적으로 따르는 패턴임

        ```java
        public final class CaseInsensitiveString implements Comparable<CaseInsensitiveString> { 
        	public int compareTo(CaseInsensitiveString cis) { return String.CASE_INSENSITIVE_ORDER.compare(s, cis.s); } ....
        }
        ```


- compareTo 메서드 안에서 기본 타입은 박싱된 기본 타입의 `compare`을 사용해 비교한다.**compareTo 메서드에서 관계 연산자 `<` 와 `>` 를 사용하는 이전 방식은 거추장스럽고 오류를 유발하니 추천하지 않는다.**
- 핵심 필드가 여러 개라면 어느 것을 먼저 비교하느냐가 중요해진다. 가장 핵심적인 필드부터 비교해나가자.
    - 핵심되는 필드가 똑같다면 똑같지 않은 필드를 찾을 때까지 그 다음으로 중요한 필드를 비교해나간다.
    - 순서를 비교하는 데 있어 가장 중요한 필드를 비교하고 그 값이 0이라면 다음 필드를 비교한다.

    ```java
    public int compareTo(PhoneNumber pn) {
      int result = Short.compare(areaCode, pn.areaCode); // 가장 중요한 필드
      if (result == 0) {
        result = Short.compare(prefix, pn.prefix); // 두 번째로 중요한 필드
        if (result == 0) {
          result = Short.compare(lineNum, pn.lineNum); // 세 번째로 중요한 필드
      }
      return result;
    }
    
    ```

- 기존 클래스를 확장하고 필드를 추가하는 경우 compareTo 규약을 지킬 수 없다.
    - Composition을 활용할 것. (Java 8 이전)
- Java 8부터 함수형 인터페이스, 람다, 메서드 레퍼런스와 Comparator가 제공하는 기본 메서드와 static 메서드를 사용해서 Comparator를 구현할 수 있었다.
    - 약간의 성능 저하가 뒤따른다.

    ```java
    //비교자 생성 메서드를 활용한 비교자
    //비교 생성 메서드 2개 
    private static final Comparator<PhoneNumber> COMPARATOR =
              comparingInt((PhoneNumber pn) -> pn.areaCode) //비교 생성 메서드 1
                      .thenComparingInt(pn -> pn.prefix) //비교 생성 메서드 1
                      .thenComparingInt(pn -> pn.lineNum);
    
     public int compareTo(PhoneNumber pn) {
         return COMPARATOR.compare(this, pn);
     }
    
    ```

    - Comparator가 제공하는 메서드 사용하는 방법
        - Comparator의 static 메서드를 사용해서 Comparator 인스턴스 만들기 `(ex. comparingInt)`
        - 인스턴스를 만들었다면 default 메서드를 사용해서 메서드 호출 이어가기 (체이닝) `(ex. thenComparingInt)`
        - static 메서드와 default 메서드의 매개 변수로는 람다표현식 또는 메서드 레퍼런스`(ex. PhoneNumber::getAreaCode)`를 사용할 수 있다.
- 값의 차를 기준하는 비교자 → 추이성을 위배한다. (정수 오버 플로 같은 오류를 낼 수 있다.)
    - ex)

        ```java
        static Comparator<Object> hashCodeOrder = new Comparator<>() {
        		public int compare(Object o1, Object o2){
        				return o1.hashCode()- o2.hashCode();
        		}
        };
        ```

    - 대신 2가지 방식 중 하나를 선택하자
        - 1) 정적 compare 메서드를 활용한 비교자

          ```java
          static Comparator<Object> hashCodeOrder = new Comparator<>() {
                  public int compare(Object o1, Object o2){
                          return Integer.compare(o1.hashCode(), o2.hashCode());
                  }
          };
          ```

        - 2) 비교자 생성 메서드를 활용한 비교자

          ```java
          static Comparator<Object> hashCodeOrder = 
              Comparator.comparingInt(o -> o.hashCode());
          ```


---

> 핵심 정리
>
> - 순서를 고려해야 하는 값 클래스를 작성한다면 꼭 **Comparable 인터페이스**를 구현하여 그 인스턴스들을 쉽게 정렬하고, 검색하고, 비교 기능을 제공하는 컬렉션과 어우러지도록 해야한다.
> - compareTo 메서드에서 필드의 값을 비교할 때 < 와 > 연산자는 쓰지 말아야 한다.
> - 그 대신 박싱된 기본 타입 클래스가 제공하는 정적 compare 메서드나 Comparator 인터페이스가 제공하는 비교자 생성 메서드를 사용하자.