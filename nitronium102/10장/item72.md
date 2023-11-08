### 10장 : 예외
# 🍀 [아이템 72] 표준 예외를 사용하라

## 📒 표준 예외를 재사용하라
- 우리의 API가 다른 사람이 익히고 사용하기 쉬워진다
- 낯선 예외를 사용하지 않기 때문에 읽기 쉽다
- 예외 클래스 수가 적을수록 메모리 사용량도 줄고 클래스를 적재하는 시간도 적게 걸린다

## 📒 많이 재사용되는 예외 종류
- `IllegalArgumentException` : 허용하지 않는 값이 인수로 건네졌을 때
    - `NullPointerException` : null은 따로 처리
- `IllegalStateException` : 객체가 메서드를 수행하기에 적절하지 않은 상태일 때
    - ex) 제대로 초기화되지 않은 객체를 사용하려 할 때
- `IndexOutOfBoundsException` : 인덱스가 범위를 넘어섰을 때
- `ConcurrentModificationException` : 허용하지 않는 동시 수정이 발견됐을 때
    - ex) 단일 스레드에서 사용하려고 설계한 객체를 여러 스레드가 동시에 수정하려고 할 때
- `UnsupportedException` : 호출한 메서드를 지원하지 않을 때
    - ex) 원소를 넣을 수만 있는 List 구현체에 누군가 remove 메서드를 호출할 때 

## 📒 일반적인 규칙
- 인수 값이 무엇이었든 어차피 실패 : `IllegalStateException`
- 그렇지 않으면 : `IllegalArgumentException`