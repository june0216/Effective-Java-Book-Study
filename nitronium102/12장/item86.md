### 12장 : 직렬화
# 🍀 [아이템 86] Serializable을 구현할지는 신중히 결정하라

## 📒 직렬화 가능한 클래스
어떤 클래스의 인스턴스를 직렬화할 수 있게 하려면 클래스 선언에 implements `Serializable`만 덧붙이면 된다. 너무 쉽게 적용하지만 매우 비싼 대가를 치를 수 있다.

## 📒 Serializable의 단점
### 릴리즈 후에 수정이 어렵다
- Serializable을 구현하면 직렬화 형태도 하나의 공개 API가 되는 것이다. 
- 직렬화 형태는 적용 당시 클래스의 내부 구현 방식에 종속적이게 되는 것이다.
- 클래스의 private과 package-private 인스턴스 필드마저 API로 공개되기 때문에 캡슐화도 깨진다.

- 클래스의 내부 구현을 수정한다면 원래의 직렬화 형태와 달라지게 된다. 구버전의 인스턴스를 직렬화한 후에 신버전 클래스로 역직렬화를 시도하면 오류가 발생할 것이다.

### SerialVersionUID
> 수정을 어렵게 만드는 요소
모든 직렬화된 클래스는 고유 식별 번호를 부여받는데, 클래스 내부에 직접 명시하지 않는 경우 시스템이 런타임에 식별 번호를 자동으로 생성한다.

<br>만약 SerialVersionUID를 수정하게 된다면 직렬 버전 UID값도 변한다 (자동 생성되는 값에 의존하여 호환성이 쉽게 깨진다)

### 버그와 보안에 취약하다
자바에서는 객체를 생성자를 이용하여 만든다<br>
하지만 직렬화는 이러한 기본 방식을 우회하여 객체를 생성하고, 역직렬화는 일반 생성자의 문제가 그대로 적용되는 '숨은 생성자'이다.
<br> => 역직렬화를 사용하면 불변식 깨짐과 허가되지 않은 접근에 쉽게 노출된다 

### 새로운 버전을 릴리즈할 때 테스트 요소가 많아진다
직렬화 가능한 클래스가 수정되면, 새로운 버전의 인스턴스를 구버전으로 역직렬화할 수 있는지 테스트해봐야 한다.

### 구현 여부는 쉽게 결정할 것이 아니다.
Serializable을 꼭 구현해야 한다면 클래스를 설계할 때마다 따르는 이득과 비용을 잘 고려해야 한다. 예를 들어 BigInteger와 Instant 같은 ‘값’ 클래스와 컬렉션 클래스는 Serializable을 구현하였고 스레드 풀처럼 ‘동작’ 하는 객체를 표현한 클래스는 대부분 구현하지 않았다.

## 📒 Serializable을 구현하면 안 되는 경우
- 상속 목적으로 설계된 클래스와 대부분의 인터페이스 : 확장하거나 구현하는 대상에게 위험성을 전이
- 만약 Serializable을 구현한 클래스만 지원하는 프레임워크를 사용한다면 방법이 없다<br>
ex) Throwable : 내부에서 Serializable 구현
- 직렬화와 확장이 모두 가능한 클래스를 만든다면, 하위 클래스에서 finalize 메서드를 재정의하지 못하게 해야 한다.
    - 자신이 재정의하고 final 키워드를 붙이자

### 내부 클래스는 직렬화를 구현하면 안 된다
내부 클래스는 바깥 인스턴스의 참조와 유효 범위에 속한 지역변수를 저장하기 위한 필드가 필요하다. <br>이 필드들은 컴파일러가 자동으로 추가하는데, 익명 클래스와 지역 클래스의 네이밍 규칙이 자바 명세에 없는 것처럼 이 필드들도 어떻게 추가되는지 알 수가 없다. <br>
=> 내부 클래스의 기본 직렬화 형태는 명확하지 않다<br> 단, 정적 멤버 클래스는 예외다.