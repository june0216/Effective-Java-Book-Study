### 2장 : 객체 생성과 파괴 
# 🍀 [아이템 1] 생성자 대신 정적 팩토리 메서드를 고려하라

클래스의 인서튼스를 생성하는 방법은 (1) public 생성자 (2) 정적 팩토리 메서드가 있다
클래스는 public 생성자 대신 (혹은 생성자와 함께) 정적 팩토리 메서드를 제공할 수 있다.

1. public 생성자
```java
public class Person() {
    public Person() {

    }
}
```

2. 정적 팩토리 메서드 static factory method
```java
public class Person() {
    private static Person PERSON = new Person();

    private Person() { // 외부 생성 금지, private
    }

    public static final Person getInstance() {
        return PERSON;
    }
}
```
&nbsp;

java 개발 시 **Arrays, Collections** 클래스에서 정적 팩토리 메서드를 사용한다.
```java
int[] nums = {1,2,3,4,5};
List<String> list = Arrays.asList(nums);
Collections.sort(list);
```
&nbsp;

### 정적 팩토리 메서드란?
static으로 선언된 메소드이며, `new Object()`와 같이 객체 생성을 하지 않고 사용할 수 있는 메소드

```java
public static Boolean valueOf(boolean b) {
    return b ? Boolean.TRUE : Boolean.FALSE;
}
```

&nbsp;

# 📒정적 팩토리 메서드의 장점
## 📃 1.이름을 가질 수 있다
> 한 클래스에 생성자가 여러 개 필요하다면, 생성자를 정적 팩토리 메서드로 바꾸고 이름을 지어주자

한 객체의 생성자 여러 개로 오버로딩하면, 모든 생성자의 이름은 같지만 매개변수만 달라진다.이 경우, 단순히 매개변수만 보고 반환될 객체의 특징을 제대로 알기 어렵다.

```java
// 동일한 시그니처를 가진 생성자를 여러 개 만들 수 없다
public class Foo {
	public Foo(String name) { ... }
	public Foo(String address) { ... } // 컴파일 에러
}
```

반면 잘 만든 이름을 가진 정적 팩토리의 객체의 특성을 묘사하기 좋다.

```java
public class Foo {
	public Foo() { ... }
	public static withName(String name) { 
		Foo foo = new Foo();
		foo.name = name;
		return foo;
	}
	public static withAddress(String address) { 
		Foo foo = new Foo();
		foo.address = address;
		return foo;
	} 
}
```
&nbsp;
## 📃 2. 호출될 때마다 인스턴스를 새로 생성하지 않아도 된다
불변 클래스는 인스턴스를 미리 만들어 놓거나 새로 생성한 인스턴스를 캐싱하여 재활용하는 식으로 불필요한 객체 생성을 피할 수 있다. 

따라서 같은 객체가 자주 요청되는 상황에서 성능을 끌어올려 준다.
```java
public class Foo {
	private static final Foo GOOD_NIGHT = new Foo();
	public Foo() { ... }

	public static getInstance() { 
		return GOOD_NIGHT;
	} 

	public static void main(String[] args) {
		Foo foo = Foo.getInstance();
	}
}
```
GOOD_NIGHT이라는 상수를 만들어 getInstance()가 호출될 때마다 매번 새로운 Foo 객체가 아닌,
동일한 (이미 만들어놓은) Foo 객체가 return 되도록 함.

```java
// 객체가 하나만 생성되었음을 확인할 수 있다.
public class Main {
    public static void main(String[] args) {
        Foo a = Foo.getInstance();
        Foo b = Foo.getInstance();
        System.out.println(a == b); // true
    }
}
```
&nbsp;

**불변 클래스 (Immutable)**
> 변경이 불가능한 클래스
- 레퍼런스 타입의 객체 (heap 영역에서 생성)
- Thread-safe (재할당은 가능. 값 복사 X)
Ex. String, Boolean, Integer, Float, Long ↔ (StringBuilder는 가변 클래스)

&nbsp;

**플라이웨이트 패턴 (Flyweight pattern)**
> '공유'를 통하여 다양한 객체들을 효과적으로 지원하는 방법
- 객체 내부에서 참조하는 객체를 직접 만들지 않고, 있다면 객체를 공유하고 없다면 만들어 줌 (Pool로 관리)
- Ex. String Pool (Heap 영역의 permanant 영역에 할당): 같은 내용의 String 객체가 선언된다면 기존의 String 객체를 참조
- 데이터를 공유하여 메모리를 절약하는 패턴

&nbsp;

**인스턴스 통제 instance-controlled 클래스**
반복되는 요청에 같은 객체를 반환하는 정적 팩토리 방식의 클래스는 인스턴스를 통제할 수 있다.
- 클래스를 singleton으로 만들 수 있다(인스턴스를 오직 하나만 생성 가능)
- 클래스를 인스턴스화 불가 noninstantiable로 만들 수 있다.
- 불변 값 클래스에서 동치인 인스턴스가 단 하나뿐임을 보장할 수 있다.
`a==b`일 때만 `a.equals(b)`` 성립

&nbsp;

## 📃 3. 반환 타입의 하위 타입 객체를 반환할 수 있는 능력이 있다.

정적 메서드로부터 인터페이스 자체를 반환하여, 구현 클래스를 공개하지 않고도 그 객체를 반환할 수 있다. <br>
**구현 클래스의 상세를 숨길 수 있고, 사용자는 객체를 인터페이스만으로 다룰 수 있다.**

```java
public static void main(String[] args){
    //기존 생성자라면
    java.util.Collections collections = new Collections();
    collections.45개의API();
    
    //정적 팩토리 메소드라면
    Collections.45개의API();
}
```
사용자는 명시한 인터페이스대로 동작하는 객체를 얻을 것임을 알기에 굳이 별도 문서를 찾아가며 실제 구현 클래스가 무엇인지 알아보지 않아도 된다. 

## 📃 4. 입력 매개변수에 따라 매번 다른 클래스의 객체를 반환할 수 있다.
> 반환 타입의 하위 객체이기만 하면 어떤 클래스의 객체를 반환하던 상관없다.


```java
public interface MyInt {
    static MyInt of(int v) {
        MyInt instance;

        if (v > 100) {
            instance = new MyBigInt(v);
        } else {
            instance = new MySmallInt(v);
        }

        return instance;
    }

    String getValue();

    class MySmallInt implements MyInt {
        private int value;
        MySmallInt(int v) {
            this.value = v;
        }

        @Override
        public String getValue() {
            return "MySmallInt: " + this.value;
        }
    }

    class MyBigInt implements MyInt {
        private int value;

        MyBigInt(int v) {
            this.value = v;
        }

        @Override
        public String getValue() {
            return "MyBigInt: " + this.value;
        }
    }
}
```

```java
public interface MyInt {
    static MyInt of(int v) {
        MyInt instance;

        if (v > 100) {
            instance = new MyBigInt(v);
        } else {
            instance = new MySmallInt(v);
        }

        return instance;
    }

    String getValue();

    class MySmallInt implements MyInt {
        private int value;
        MySmallInt(int v) {
            this.value = v;
        }

        @Override
        public String getValue() {
            return "MySmallInt: " + this.value;
        }
    }

    class MyBigInt implements MyInt {
        private int value;

        MyBigInt(int v) {
            this.value = v;
        }

        @Override
        public String getValue() {
            return "MyBigInt: " + this.value;
        }
    }
}
```

클라이언트는 팩토리가 건네주는 객체가 어느 클래스의 인스턴스인지 알 수도 없고 알 필요도 없다. `MyInt`의 하위 클래스이기만 하면 된다.

정수가 100 이하라면 `MySmallInt` 인스턴스를, 그렇지 않다면 `MyBigInt` 인스턴스를 반환한다. <br>
따라서 향후에 JDK 변화에 따라 새로운 타입을 만들거나 기존 타입이 사라지더라고 문제가 되지 않는다.

&nbsp;
## 📃 5. 정적 팩토리 메소드를 작성하는 시점에는 반환할 객체의 프레임워크가 존재하지 않아도 된다.
> 장점 4번의 연장선

이런 유연한 점을 이용하여 **서비스 제공자 프레임워크Service Provider Framework**를 만드는 근간이 된다.

서비스 제공자 프레임워크는 3가지의 핵심 컴포넌트로 이뤄진다.

- 서비스 인터페이스(Service Interface): 구현체의 동작을 정의
- 제공자 등록 API(Provider Registration API): 제공자가 구현체를 등록할 때 사용
- 서비스 접근 API(Service Access API): 클라이언트가 서비스의 인스턴스를 얻을 때 사용
<br>
3개의 핵심 컴포넌트와 더불어 종종 네번째 컴포넌트가 사용되기도 한다.

- 서비스 제공자 인터페이스(Service Provider Interface): 서비스 인터페이스의 인스턴스를 생성하는 팩토리 객체를 설명
대표적인 서비스 제공자 프레임워크로는 JDBC(Java Database Connectivity)가 있다.
MySql, OracleDB, MariaDB등 다양한 Database를 JDBC라는 프레임워크로 관리할 수 있다.

ex) JDBC의 경우, DriverManager.registerDriver()가 프로바이더 등록 API, DriverManager.getConnection()이 서비스 액세스 API, 그리고 Driver가 서비스 프로바이더 인터페이스 역할을 한다. <br> getConnection을 썼을 때 실제 return 되어서 나오는 객체는 DB Driver마다 다르다.

이 덕분에 자바 버전에 상관없이 이 전의 코드들이 안정적으로 작동할 수 있다.

&nbsp;

# 📒정적 팩토리 메서드의 단점
## 📃 1. 상속을 하려면 public이나 protected 생성자가 필요하므로, 정적 팩토리 메서드만 제공하면 하위 클래스를 만들 수 있다.
상속을 하게되면 `super()`` 를 호출하며 부모 클래스의 함수들을 호출하게된다. <br>그러나 부모 클래스의 생성자가 private이라면 상속이 불가능하다.

보통 정적 팩토리 메서드만 제공하는 경우 생성자를 통한 인스턴스 생성을 막는 경우가 많다.
ex) Collections는 생성자가 private으로 구현되어 있기 때문에 상속할 수 없다.

그러나, 이 제약은 상속보다 컴포지션을 사용하도록 유도하고, 불변 타입으로 만들려면 이 제약을 지켜야 한다는 점에서 오히려 장점으로 받아들이기도 한다.
(컴포지션: 기존 클래스를 확장하는 대신에 새로운 클래스를 만들고 private 필드로 기존 클래스의 인스턴스를 참조하는 방식)

&nbsp;
## 📃 2. 정적 팩토리 메서드는 프로그래머가 찾기 어렵다
생성자는 javadoc 상단에 모아서 보여주지만 static 팩토리 메서드는 API 문서에서 특별히 다뤄주지 않는다. 따라서 클래스나 인터페이스 문서 상단에 팩토리 메서드에 대한 문서를 제공하는 것이 좋겠다.

&nbsp;

# 📒 정적 팩터리 메서드 명명 방식
- from: 매개변수를 하나 받아서 해당 타입의 인스턴스를 반환하는 형변환 메서드<br>
`Date d = Date.from(instant);`

- of: 여러 매개변수를 받아 적합한 타입의 인스턴스를 반환하는 집계 메서드<br>
`Set<Rank> faceCards = EnumSet.of(JACK, QUEEN, KING);`

- instance 혹은 getInstance: 매개변수로 명시한 인스턴스를 반환, 같은 인스턴스임을 보장하진 않는다<br>
`StackWalker luke = StackWalker.getInstance(options);`

- create 혹은 newInstance: instance 혹은 getInstance와 같지만, 매번 새로운 인스턴스 생성을 보장<br>
`Object newArray = Array.newInstance(classObject, arrayLen);`
- getType: getInstance와 같으나, 생성할 클래스가 아닌 다른 클래스에 팩터리 메서드를 정의할 때 사용<br>
`FileStore fs = Files.getFileStore(path); ("Type"은 팩터리 메서드가 반환할 객체의 타입)`

- newType: newInstance와 같으나, 생성할 클래스가 아닌 다른 클래스에 팩터리 메서드를 정의할 때 사용<br>
`BufferedReader br = Files.newBufferedReader(path);`

- type: getType과 newType의 간결한 버전<br>
`List<Complaint> litany = Collections.list(legacyLitany);`
