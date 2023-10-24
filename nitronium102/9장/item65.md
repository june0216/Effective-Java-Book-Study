### 9장 : 일반적인 프로그래밍 법칙
# 🍀 [아이템 65] 리플렉션보다는 인터페이스를 사용하라

## 📒 리플렉션이란?
- 실행 중인 자바 애플리케이션이 JRE (Java Runtime Environment) 를 검사, 수정, 상호작용할 수 있도록 하는 기능이다.
- 애플리케이션의 런타임 동작을 검사하거나 수정할 수 있다.

## 📒 리플렉션의 기능
- 특정 클래스의 생성자, 메서드, 필드 정보를 가져올 수 있다.
추가적으로 멤버 이름, 필드 타입, 메서드 시그니쳐 등을 가져올 수 있다.
- 실제 생성자, 메서드, 필드를 조작할 수도 있다.
Method.invoke() 는 메서드를 호출할 수 있게 해준다.

=> 리플렉션을 이용하여 컴파일 당시 존재하지 않던 클래스도 이용 가능하다.

## 📒 리플렉션의 단점
- 컴파일 타입 검사의 이점을 누릴 수 없다.(런타임에야 오류를 알게될 것이다.)
- 코드가 지저분해진다.
- 성능이 떨어진다.
- 리플렉션을 통한 메서드 호출은 상당히 느리다. (책 예시에서는 무려 11배 차이)

=> 일반적인 코드에서는 리플렉션이 필요 없다. <u>코드 분석 도구나 의존관계 주입 프레임워크</u>와 같은 복잡한 애플리케이션에서나 리플렉션이 가끔 필요하다.


리플렉션은 아주 제한된 형태로만 사용해야 단점을 피하고 이점만 취할 수 있다.

### 📃 리플렉션 예제
> 클래스 이름으로 클래스를 생성하는 코드
```java
public class Item65Test {
    @Test
    public void reflectionTest() {
        // 클래스 가져오기
        Class<? extends Set<String>> cl = null;

        try {
            // 🔥 리플렉션 사용[Class.forName]
            cl = (Class<? extends Set<String>>) Class.forName("java.util.LinkedHashSet");
        } catch(ClassNotFoundException e) {
            // 해당 경로에서 클래스를 찾을 수 없을 때
            e.printStackTrace();
        }

        // 생성자 얻기
        Constructor<? extends Set<String>> cons = null;
        try {
            // 🔥 리플렉션 getDeclaredConstructor()
            cons = cl.getDeclaredConstructor();
        } catch (NoSuchMethodException e) {
            // 매개변수 없는 생성자가 없을 때
            e.printStackTrace();
        }

        Set<String> s = null;

        try {
            s = cons.newInstance();
        } catch (InvocationTargetException e) {
            // 생성자가 예외를 던졌을 때
            e.printStackTrace();
        } catch (InstantiationException e) {
            // 클래스를 인스턴스화할 수 없을 때
            e.printStackTrace();
        } catch (IllegalAccessException e) {
            // 생성자에 접근할 수 없을 때
            e.printStackTrace();
        } catch (ClassCastException e) {
            // Set을 구현하지 않은 클래스일 때
            e.printStackTrace();
        }

        /*
        java.util.LinkedHashSet 의 결과
        s = [안녕, 하세, 요, .]
        s1 = 안녕하세요.
        */

        /*
        java.util.HashSet 의 결과,
        s = [요, 안녕, 하세, .]
        s1 = 요안녕하세.
        */
        s.addAll(List.of("안녕", "안녕", "하세", "요", "."));
        System.out.println("s = " + s);

        StringJoiner sj = new StringJoiner("");
        for (String s1 : s) {
            sj.add(s1);
        }
        String s1 = sj.toString();
        System.out.println("s1 = " + s1);
    }
}
```

### 📃 단점
- 런타임에 6가지 예외가 발생 가능하다.
    - 제대로 예외처리를 한다면 고려할 점이 너무 많을 수 있다.
    - 다만, 자바7부터는 ReflectiveOperationException 하나로 리플렉션 예외를 잡을 수 있다
- 클래스 이름만으로 클래스를 생성하기 위해서는 25줄의 코드를 작성해야 한다.
=> 일반 자바코드를 쓰는 것보다 압도적으로 코드가 많다.

## 📒 정리
- 리플렉션은 런타임에 존재하지 않을 수도 있는 다른 클래스, 메서드, 필드와의 의존성을 관리할 때 적합하다.
- 컴파일 타임에는 알 수 없는 클래스를 사용하는 프로그램을 작성하면 리플렉션을 사용해야 한다.
- 되도록, 객체 생성에만 사용하고 생성한 객체를 이용할 때는 적절한 인터페이스나 컴파일타임에 알 수 있는 상위 클래스로 형변환해 사용하자.