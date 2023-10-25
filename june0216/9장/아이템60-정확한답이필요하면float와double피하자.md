## 📖**`float`과 `double`이 사용하는 이진 부동소수점의 취약점**

```java
@Test
public void floatDoubleTest1() {
    System.out.println(1.03 - 0.42);
// 결과: 0.6100000000000001
}
```

- `float` 과 `double` 은 과학과 공학 계산 용으로 설계되었다.
    - ⇒ 이진 부동소수점 연산에 쓰여 근사치로 계산하도록 설계된 타입이다.
- 위 코드의 경우 `0.61` 이 그대로 나오지 않고, `0.6100000000000001` 이 나오게 된다.
- 이진수로 소수점을 표현하는 데 한계가 있고, 그 한계가 이진 부동소수점을 사용함으로써 여실히 드러난다.
    - 예를들어, `0.1` 을 2진수로 표현하면 `0.0001100,1100,...` 와 같이 순환소수로 표현해야 한다.
        - 이러한 순환소수를 비트 수의 한계까지 표현하다보면 위와 같은 결과가 나온다.
- 금융 관련 계산과 맞지 않다.

    ```java
    
    public static void main(String[] args) {
        double funds = 1.00;
        int itemBought = 0;
        for(double price = 0.10; funds >= price; price += 0.10) {
            funds -= price;
            itemsBought++;
        }
        
        System.out.println(itemBought + "개 구입");
        System.out.println("잔돈(달러): " + funds);
    }
    ```

    - 잔돈으로 0.3999999999999가 출력된다.
        - 당연히 우리가 원하는 결과는 아니다. 이처럼 정확한 계산이 필요한 금융 계산에 부동소수 타입을 사용하면 정확한 답이 아닌 **근사치**로 계산된다.

### 📖 해결방법

```java
@Test
public void bigDecimalTest() {
    BigDecimal bigDecimal1 = new BigDecimal("1.03");
    BigDecimal bigDecimal2 = new BigDecimal("0.42");
    System.out.println(bigDecimal1.subtract(bigDecimal2));
// 결과: 0.61
}
```

- `BigDecimal`, `int`, `long` 등을 사용하여 정수로 연산해야 한다.
- `BigDecimal` 의 생성자 중 반드시 문자열로 숫자를 받는 생성자를 이용해야 한다.
    - 그래야만 값이 정확하게 나온다.


### 📖 단점

- 1) 기본 타입보다 쓰기가 불편
- 2) 느리다.

### 📖  정리

- 대안 = int 혹은 long 타입을 사용 (성능이 중요할 때만)
    - 하지만 이 또한, 값의 크기가 제한되고 소수점을 직접 관리해야한다.
    - 숫자를 아홉 자리 십진수로 표현 가능 → int
    - 숫자를 열여덟 자리 십진수를 표현할 수 있다면 → long
    - 18자리 이상 → BigDecimal 사용하자!