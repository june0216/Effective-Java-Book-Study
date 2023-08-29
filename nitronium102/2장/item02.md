### 2장 : 객체 생성과 파괴 
# 🍀 [아이템 2] 생성자에 매개변수가 많다면 빌더를 고려하라
## 📒 생성자와 정적 팩토리 메서드의 제약
> 선택적 매개변수가 많을 때 적절히 대응하기 어렵다

### 점층적 생성자 패턴 telescoping constructor pattern
> 필수 매개변수를 받는 생성자 1개, 그리고 선택 매개변수를 하나씩 늘려가며 생성자를 만드는 패턴

<details>
<summary>코드</summary>
<div>

```java
public class NutritionFacts {
    private final int servingSize;
    private final int servings;
    private final int calories;
    private final int fat;
    private final int sodium;
    private final int carbohydrate;

    public NutritionFacts(int servingSize, int servings) {
        this(servingSize, servings, 0);
    }

    public NutritionFacts(int servingSize, int servings, int calories) {
        this(servingSize, servings, calories, 0);
    }

    public NutritionFacts(int servingSize, int servings, int calories, int fat) {
        this(servingSize, servings, calories,  fat, 0);
    }

    public NutritionFacts(int servingSize, int servings, int calories, int fat, int sodium) {
        this(servingSize, servings, calories,  fat,  sodium, 0);
    }

    public NutritionFacts(int servingSize, int servings, int calories, int fat, int sodium, int carbohydrate) {
        this.servingSize = servingSize;
        this.servings = servings;
        this.calories = calories;
        this.fat = fat;
        this.sodium = sodium;
        this.carbohydrate = carbohydrate;
    }
}
```
</div>
</details>

### 점층적 생성자 패턴의 단점
> 매개변수 개수가 많아지면 클라이언트 코드를 작성하거나 읽기 어렵다
- 초기화하고 싶은 필드만 포함한 생성자가 없다면, 원치 않는 매개변수까지 값을 지정해줘야 한다
- 복잡하고 읽기 어렵다
- 매개변수 개수가 많아질 경우 걷잡을 수 없다

&nbsp;

## 📒 자바빈즈 패턴 JavaBeans Pattern
> 매개변수가 없는 생성자로 객체를 만든 후, setter 메서드들을 호출해 원하는 매개변수의 값을 설정하는 방식


<details>
<summary>코드</summary>
<div>

```java
public class NutritionFacts {
    private int servingSize = -1;
    private int servings = -1;
    private int calories = 0;
    private int fat = 0;
    private int sodium = 0;
    private int carbohydrate = 0;

    public NutritionFacts() {}

    public void setServingSize(int servingSize) {
        this.servingSize = servingSize;
    }

    public void setServings(int servings) {
        this.servings = servings;
    }

    public void setCalories(int calories) {
        this.calories = calories;
    }

    public void setFat(int fat) {
        this.fat = fat;
    }

    public void setSodium(int sodium) {
        this.sodium = sodium;
    }

    public void setCarbohydrate(int carbohydrate) {
        this.carbohydrate = carbohydrate;
    }
}
```

```java
public class Main {
    public static void main(String[] args) {
        NutritionFacts cocaCola = new NutritionFacts();
        cocaCola.setServingSize(240);
        cocaCola.setServings(8);
        cocaCola.setCalories(100);
        cocaCola.setFat(0);
        cocaCola.setSodium(35);
        cocaCola.setCarbohydrate(27);
    }
}
```
</div>
</details>

### 단점
> 클래스를 불변으로 만들 수 없으며, 스레드 안전성을 얻기 위해 추가 작업(freeze 등)을 해줘야 한다
- 객체 하나를 만들기 위해서는 메서드 여러 개를 호출해야 한다
- 객체가 완전히 생성되기 전까지는 일관성 consistency이 무너진 상태에 놓이게 된다 (버그 코드와 문제 발생코드가 물리적으로 멀리 떨어져 있어 디버깅이 어렵다)

&nbsp;

## 📒 발더 패턴 Builder Pattern
> 필수 매개변수만들어 객체를 생성하고, 일종의 setter를 사용하여 선택 매개변수들을 설정한 후, 매개변수가 없는 build() 메서드를 호출해 완전한 객체를 생성하는 패턴

빌더는 생성할 클래스 안에 `static` 멤버 클래스로 만들어두는 것이 보통이다 (lombok 사용 가능)

<details>
<summary>코드</summary>
<div>

```java
public class NutritionFacts {
    private final int servingSize;
    private final int servings;
    private final int calories;
    private final int fat;
    private final int sodium;
    private final int carbohydrate;

    private NutritionFacts(Builder builder) {
        servingSize = builder.servingSize;
        servings = builder.servings;
        calories = builder.calories;
        fat = builder.fat;
        sodium = builder.sodium;
        carbohydrate = builder.carbohydrate;
    }

    public static class Builder {
        // 필수 매개변수
        private final int servingSize;
        private final int servings;

        // 선택 매개변수
        private int calories = 0;
        private int fat = 0;
        private int sodium = 0;
        private int carbohydrate = 0;

        // 필수 매개변수만을 담은 Builder 생성자
        public Builder(int servingSize, int servings) {
            this.servingSize = servingSize;
            this.servings = servings;
        }

        // 선택 매개변수의 setter, Builder 자신을 반환해 연쇄적으로 호출 가능
        public Builder calories(int val) {
            calories = val;
            return this;
        }

        public Builder fat(int val) {
            fat = val;
            return this;
        }
        
        public Builder sodium(int val) {
            sodium = val;
            return this;
        }
        
        public Builder carbohydrate(int val) {
            carbohydrate = val;
            return this;
        }
        
        // build() 호출로 최종 불변 객체를 얻는다.
        public NutritionFacts build() {
            return new NutritionFacts(this);
        }
    }
}
```
</div>
</details>

`NutritionFacts` 클래스는 불변이며, 모든 매개변수의 기본값들을 한 곳에 모아놨다. <br>
이 빌더의 setter 메소드는 빌더 자신을 반환하기 때문에 연쇄적으로 호출할 수 있다 (method chaining)
```java
NutritionFacts cocaCola = new NutriFacts.Builder(240, 8)
      .calories(100).sodium(35).carbohydrate(30).build();
```

### 빌더와 자바빈즈의 차이점
> 불변성
1) 자바빈즈
- 객체를 생성한 **후**, 값을 setter를 통해 넣는다
- 객체 사용 도중 setter를 통해 유효하지 않은 값이나 null값이 들어갈 수 있다.

2) 빌더 패턴
- 객체 생성 **전**, 값을 setter를 통해 넣는다
- 값을 모두 넣었다면 build()를 호출하여 객체를 생성한다
- 객체 사용 중에 값이 변경될 우려가 없으며, 불변성과 안정성이 올라간다
- 빌더 패턴 사용 시에넌 public setter를 선언해서는 안 된다

## 📒 게층적으로 설계된 클래스와 Builder Pattern
- 각 계층의 클래스에 관련 빌더를 멤버로 정의
- 추상 클래스는 추상 빌더를 갖게 한다
- 구체 클래스 concrete class는 구체 빌더는 갖게 한다

```java
public abstract class Pizza{
   public enum Topping { HAM, MUSHROOM, ONION, PEPPER, SAUSAGE }
   final Set<Topping> toppings;

   // 추상 클래스는 추상 Builder를 가진다. 서브 클래스에서 이를 구체 Builder로 구현한다.
   abstract static class Builder<T extends Builder<T>> {
      EnumSet<Topping> toppings = EnumSet.noneOf(Topping.class);
      public T addTopping(Topping topping) {
         toppings.add(Objects.requireNonNull(topping));
         return self();
      }

      abstract Pizza build();

      // 하위 클래스는 이 메서드를 overriding하여 this를 반환하도록 해야 한다. (형변환 없이 메서드 연쇄 지원)
      protected abstract T self();
   }

   Pizza(Builder<?> builder) {
      toppings = builder.toppings.clone();
   }
}
```

### 📃 빌더 클래스 
`Pizza.Builder`는 Pizza 객체를 생성하기 위한 빌더 클래스. <br> 빌더 클래스는 추상 클래스인데, 제네릭 타입으로 T extends Builder<T>를 사용하여 재귀적 타입 한정을 적용하고 있다. <br>이는 빌더 클래스의 하위 클래스에서도 같은 빌더 타입을 반환하도록 하여 메서드 체이닝을 지원하기 위함이다.
<br>
Pizza 클래스의 생성자는 빌더 객체를 받아와서 토핑 정보를 복사하여 초기화한다. 이렇게 함으로써 피자 객체의 불변성을 보장하고 다양한 피자를 생성할 수 있다.

### 📃 self (simulated self-type)
> 하위 클래스에서 형변환 없이 메서드 연쇄를 지원하기 위해 활용

일반적으로 자바에서는 메서드 체이닝을 하기 위해서는 메서드가 항상 같은 클래스의 인스턴스를 반환해야 한다. <br>그러나 빌더 패턴에서는 메서드 체이닝을 위해 다른 클래스의 하위 클래스를 반환하게 하려면 형변환이 필요한데, 이런 형변환은 가독성을 떨어뜨리고 실수를 유발할 수 있다.
<br>
그래서 재귀적 타입 한정을 이용한 self 개념을 도입한다. 이를 활용하면 하위 클래스에서 메서드가 항상 해당 하위 클래스 타입을 반환하도록 할 수 있다. 즉, 형변환이 필요 없이 메서드 체이닝을 할 수 있게 된다.

```java
protected abstract T self();
```
여기서 T는 Pizza.Builder의 재귀적 타입 한정을 나타낸다. 하위 클래스에서는 이 메서드를 오버라이딩하여 this를 반환하도록 해야 한다. 이렇게 하면 하위 클래스에서의 self 타입은 해당 하위 클래스로 고정되게 된다.

즉, 하위 클래스에서 self 메서드를 오버라이딩하지 않으면 컴파일 에러가 발생하게 되어 제대로된 타입 체크를 도와주며, 오버라이딩한 메서드에서 this를 반환하므로써 형변환이 없이도 메서드 체이닝을 할 수 있게 된다. 이로써 가독성을 높이고 안전한 코드를 작성할 수 있다.

### 📃 계층적 빌더를 사용하는 클라이언트 코드
<details>
<summary>계층적 부모 코드</summary>
<div>

```java
public class NyPizza extends Pizza {
   public enum Size { SMALL, MEDIUM, LARGE }
   private final Size size;

   public static class Builder extends Pizza.Builder<Builder> {
      private final Size size;

      public Builder(Size size) {
         this.size = Objects.requireNonNull(size);
      }

      @Override public NyPizza build() {
         return new NyPizza(this);
      }

      @Override protected Builder self() { return this; }
   }

   private NyPizza(Builder builder) {
      super(builder);
      size = builder.size;
   }
}
```

```java
public class Calzone extends Pizza {
   private final boolean sauceInside;

   public static class Builder extends Pizza.Builder<Builder> {
      private boolean sauceInside = false;

      public Builder sauceInside() {
         sauceInside = true;
         return this;
      }

      @Override public Calzone build() {
         return new Calzone(this);
      }

      @Override protected Builder self() { return this; }
   }

   private Calzone(Builder builder) {
      super(builder);
      sauceInside = builder.sauceInside;
   }
}
```
</div>
</details>
<br>

```java
public class Main {
    public static void main(String[] args) {
        NYPizza pizza = new NYPizza.Builder(SMALL)
                .addTopping(SAUSAGE)
                .addTopping(ONION)
                .build();

        Calzone calzone = new Calzone.Builder()
                .addTopping(HAM)
                .sauceInside()
                .build();
    }
}
```
빌더를 이용하면 가변 인수 매개변수를 여러 개 사용할 수 있다.

&nbsp;

## 📒 빌더 패턴의 단점
1. 성능<br>
객체를 만들기 위해서는 그에 앞서 빌더부터 만들어야 한다. <br>
빌더 생성 비용이 크지는 않지만, 성능에 민감한 상황에서는 문제가 될 수 있다.

2. 장황<br>
점층적 생성자 패턴보다는 코드가 장황해 매개변수가 4개 이상은 되어야 값어치를 한다.

하지만 API는 시간이 지날수록 매개변수가 많아지는 경향이 있으므로 애초에 빌더로 시작하는 펴이 나을 때가 많다.