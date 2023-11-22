## 🍏  readObject란?

***readObject***는 역직렬화 과정에서 별도의 처리가 필요할 때 구현하는 ***메서드***

### 예시

**`Person`** 클래스에 **`readObject`** 메서드를 구현한 예시입니다. 이 클래스는 **`Serializable`** 인터페이스를 구현하며, **`name`** 필드가 **`null`**이 아니도록 역직렬화 과정에서 검증하는 로직을 포함

```java
import java.io.ObjectInputStream;
import java.io.IOException;
import java.io.Serializable;

public class Person implements Serializable {
    private String name;
    private transient int age; // transient 필드는 기본 직렬화에 포함되지 않음

    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }

    // readObject 메서드 구현
    private void readObject(ObjectInputStream ois) throws IOException, ClassNotFoundException {
        ois.defaultReadObject(); // 기본 역직렬화 수행

        // 역직렬화 후 추가적인 검증이나 초기화 수행
        if (name == null) {
            throw new InvalidObjectException("name cannot be null");
        }

        // 필요한 경우 transient 필드 초기화
        if (age == 0) {
            // 예시: age 필드를 기본값으로 설정
            age = 20;
        }
    }

    // getters and setters
}
```

### **readObject 메서드의 역할**

- **`defaultReadObject`** 호출: 이 호출은 기본 직렬화 메커니즘에 의해 저장된 모든 비-transient 필드를 읽어 들입니다.
- 추가적인 검증: **`readObject`** 메서드는 역직렬화된 객체의 무결성을 확인할 수 있도록 추가적인 검증 로직을 포함할 수 있습니다. 위의 예시에서는 **`name`** 필드가 **`null`**인지 확인합니다.
- 필드 초기화: transient 필드는 기본 직렬화에 포함되지 않으므로, **`readObject`**에서 이러한 필드를 적절한 값으로 초기화할 수 있습니다.

## 🍏 readObject 메서드

- 아이템 50에서 소개했듯 불변인 날짜 범위 클래스를 만드는 데 가변인 Date 필드를 이용
- 그래서 불변식을 지키고 불변을 유지하기 위해 생성자와 접근자에서 Date 객체를 방어적으로 복사하느라 코드가 상당히 길어짐

```java
// 방어적 복사를 사용하는 불변 클래스
public final class Period {
    private final Date start;
    private final Date end;

    /**
     * @param start 시작 시각
     * @param end 종료 시각; 시작 시각보다 뒤여야 한다.
     * @throws IllegalArgumentException 시작 시각이 종료 시각보다 늦을 때 발생한다.
     * @throws NullPointerException start나 end가 null이면 발행한다.
     */
    public Period(Date start, Date end) {
        this.start = new Date(start.getTime());
        this.end = new Date(end.getTime());
        if(this.start.compareTo(this.end) > 0) {
            throw new IllegalArgumentException(start + "가 " + end + "보다 늦다.");
        }
    }

    public Date start() { return new Date(start.getTime()); }
    public Date end() { return new Date(end.getTime()); }
    public String toString() { return start + "-" + end; }
}

```

- 이 클래스를 직렬화 한다면?
    - 이 클래스 선언에 implements Serializable을 추가하면 될 것 같다.
    - 하지만 이렇게 해서는 이 클래스의 중요한 **불변식**을 더는 보장하지 못하게 된다.
    - ⇒ readObject 메서드가 실질적으로 또 다른 public 생성자이기 때문이다. 따라서 다른 생성자와 똑같은 수준으로 주의를 기울여야 한다.
        - **인수가 유효한지 검사 + 매개변수를 방어적으로 복사**
- 하지만 불변식을 깨뜨릴 의도👿로 임의 생성한 바이트 스트림을 건네면 정상적인 생성자로는 만들어낼 수 없는 객체를 생성해 낼 수 있다.

## 🍏 문제를 해결하려면?

- readObject 메서드가 defaultReadObject를 호출한 다음 역직렬화된 객체가 유효한지 검사해야 한다.
- 이 유효성 검사에 실패하면 InvalidObjectException을 던지게 하여 잘못된 역직렬화가 일어나는 것을 막을 수 있다.

### But

- 정상 인스턴스에서 시작된 바이트 스트림 끝에 private Date의 필드로의 참조를 추가하면 가변 인스턴스를 만들어 낼 수 있다.
- 공격자는 ObjectInputStream에서 인스턴스를 읽은 후 스트림 끝에 추가된 '악의적인 객체 참조'를 읽어 객체의 내부 정보를 얻을 수 있다.
    - 이 참조로 얻은 Date 인스턴스들은 수정할 수 있으므로 Period 인스턴스는 더는 불변이 아니게 된다.

```java
// 가변 공격의 예
public class MutablePeriod {
    //Period 인스턴스
    public final Period period;

    //시작 시각 필드 - 외부에서 접근할 수 없어야 한다.
    public final Date start;
    //종료 시각 필드 - 외부에서 접근할 수 없어야 한다.
    public final Date end;

    public MutablePeriod() {
        try {
            ByteArrayOutputStream bos = new ByteArrayOutputStream();
            ObjectArrayOutputStream out = new ObjectArrayOutputStream(bos);

            //유효한 Period 인스턴스를 직렬화한다.
            out.writeObject(new Period(new Date(), new Date()));

            /**
             * 악의적인 '이전 객체 참조', 즉 내부 Date 필드로의 참조를 추가한다.
             * 상세 내용은 자바 객체 직렬화 명세의 6.4절을 참고하자.
             */
/*바이트 배열 ref를 생성하여 특정 형식의 바이트를 추가합니다. 이 바이트는 직렬화된 Period 인스턴스 내부의 Date 객체들을 가리키는 참조로 사용됩니다.
ref 배열에 담긴 바이트 값 0x71, 0, 0x7e, 0, 5는 직렬화된 스트림에서 특정 객체에 대한 참조를 나타냅니다. 여기서는 Period의 start와 end 필드를 참조합니다.
*/
            byte[] ref = {0x71, 0, 0x7e, 0, 5}; // 참조 #5
            bos.write(ref); // 시작 start 필드 참조 추가
            ref[4] = 4; //참조 #4
            bos.write(ref); // 종료(end) 필드 참조 추가

            // Period 역직렬화 후 Date 참조를 훔친다.
            ObjectInputStream in = new ObjectInputStream(new ByteArrayInputStream(bos.toByteArray()));
/*ObjectInputStream을 사용하여 바이트 스트림을 다시 Period 객체로 역직렬화합니다.
이후, 추가된 바이트를 통해 역직렬화된 start와 end Date 객체들에 접근*/
            period = (Period) in.readObject();
            start = (Date) in.readObject();
            end = (Date) in.readObject();
        } catch (IOException | ClassNotFoundException e) {
            throw new AssertionError(e);
        }
    }
}
```

```java
// 공격이 실제로 이뤄지는 모습
public static void main(String[] args) {
    MutablePeriod mp = new MutablePeriod();
    Period p = mp.period;
    Date pEnd = mp.end;

    //시간을 되돌리자!
    pEnd.setYear(78);
    System.out.println(p);

    //60년대로 회귀!
    pEnd.setYear(60);
    System.out.println(p);
}
```

- 위 공격 예시로 Period 인스턴스는 불변식을 유지한 채 생성됐지만, 의도적으로 내부의 값을 수정할 수 있다
- 이처럼 변경할 수 있는 Period 인스턴스를 획득한 공격자는 이 인스턴스가 불변이라고 가정하는 클래스에 넘겨 엄청난 보안 문제를 일으킬 수 있다.
- **객체를 역직렬화 할 때는 클라이언트가 소유해서는 안되는 객체 참조를 갖는 필드를 모두 반드시 방어적으로 복사해야 한다.**
- 따라서 readObject에서는 불변 클래스 안의 모든 Private 가변 요소를 방어적으로 복사해야 한다.

## 방어적 복사와 유효성 검사를 수행하는 readObject 메서드

```java
private void readObject(ObjectInputStream s) throws IOException, ClassNotFoundException {
    s.defaultReadObject();

    // 가변 요소들을 방어적으로 복사한다.
    start = new Date(start.getTime());
    end = new Date(end.getTime());

    // 불변식을 만족하는지 검사한다.
    if(start.compareTo(end) > 0) {
       throw new InvalidObjectException(start + "가 " + end + "보다 늦다.");
    }
}
```

### 주목할 점

- 방어적 복사를 유효성 검사보다 앞서 수행
- Date의 clone 메서드는 사용하지 않음

### 주의

- final 필드는 방어적 복사가 불가능
    - 그래서 이 readObject 메서드를 사용하려면 start와 end필드에서 final 한정자를 제거
- 아쉬운 일이지만 공격 위험에 노출되는 것보다 낫다.

## 🟣 기본 readObject를 써도 좋을지 판단하는 방법

- transient 필드를 제외한 모든 필드의 값을 매개변수로 받아 유효성 검사 없이 필드에 대입하는 public 생성자를 추가해도 괜찮은가?
    - 아니오라면 커스텀 readObject를 만들어 (생성자에서 수행했어야 할) 모든 유효성 검사와 방어적 복사를 수행, 혹은 직렬화 프록시 패턴(아이템 90)을 사용

        ```java
        public class Period implements Serializable {
            private final Date start;
            private final Date end;
        
            public Period(Date start, Date end) {
                this.start = new Date(start.getTime());
                this.end = new Date(end.getTime());
        
                if(this.start.compareTo(this.end) > 0) {
                    throw new IllegalArgumentException(start + "가 " + end + "보다 늦다.");
                }
            }
        
            private Object writeReplace() {
                return new SerializationProxy(this);
            }
        
            private static class SerializationProxy implements Serializable {
                private final Date start;
                private final Date end;
        
                public SerializationProxy(Period p) {
                    this.start = p.start;
                    this.end = p.end;
                }
        
                private Object readResolve() {
                    return new Period(start, end);
                }
            }
        }
        ```


---

## 🍒 안전한 readObject 메서드를 작성하는 지침

- private이어야 하는 객체 참조 필드는 각 필드가 가리키는 객체를 방어적으로 복사하라. 불변 클래스 내의 가변 요소가 여기 속한다.
- 모든 불변식을 검사하여 어긋나는 게 발견되면 InvalidObjectException을 던진다. 방어적 복사 다음에는 반드시 불변식 검사가 뒤따라야 한다.
- 역직렬화 후 객체 그래프 전체의 유효성을 검사해야 한다면 ObjectInputValidation 인터페이스를 사용하라(이 책에서는 다루지 않는다.)
- 직접적이든 간접적이든, 재정의할 수 있는 메서드는 호출하지 말자.