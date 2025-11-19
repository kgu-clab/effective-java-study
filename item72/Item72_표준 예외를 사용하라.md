# Item 72 : 표준 예외를 사용하라

## 재사용
코드 재사용과 마찬가지로 예외도 재사용 하는 것이 좋다.

## 표준 예외 재사용시 장점
- 당신의 API가 다른사람이 익히고 사용하기 쉬워진다.
- 그 API를 사용한 다른 프로그램 또한 익히고 사용하기 쉽다.
- 예외가 적을수록 메모리 사용량도 줄고 시간도 적게 걸린다.

### IllegalArgumentException
- 인수로 부적절한 값을 넘길 때
- ex) 반복 횟수를 지정하는 매개변수에 음수를 건넬 때

### IllegalStateException
- 객체의 상태가 호출된 메서드를 수행하기 부적절할 때
- ex) 초기화`( init() )`되지 않은 객체를 사용하려 할 때

### NullPointerException
- null값을 허용하지 않는 메서드에 null을 건넸을 때
- ex) `str = null`인 상태에서 `str.length()`나 `str.equals("test")`를 호출할 때

### IndexOutOfBoundsException
- 시퀀스의 허용 범위를 넘는 값을 건넬 때
- ex) 크기가 3인 리스트에서 list.get(3)을 호출하여 존재하지 않는 인덱스에 접근할 때

### ConcurrentModificationException
- 단일 스레드 사용을 위해 설계한 객체를 여러 스레드가 동시에 수정하려 할 때
- ex) for-each 문으로 리스트를 순회하는 도중, list.remove()를 사용하여 요소를 삭제하여 컬렉션의 구조를 변경할 때

### UnsupportedOperationException
- 클라이언트가 요청한 동작을 대상 객체가 지원하지 않을 때
- ex) `Arrays.asList()`로 생성된 고정 크기 리스트에 `add()`나 `remove()` 메서드를 호출할 때


## 주의
### Exception, RuntimeException, Throwable, Error는 직접 재사용하지 말자.
- 예외들의 상위 클래스로, 여러 성격의 예외들을 포괄하기 때문에 안정적으로 테스트 할 수 없다.

### 재사용 예외 선택이 어려울 수도 있다.
예를 들어, 카드 덱을 표현하는 객체가 있고, 인수로 건넨 수만큼 카드를 뽑아 나눠주는 메서드를 제공한다고 해보자.

덱에 남은 카드보다 큰 값을 건넨다면

인수의 값이 너무 크다면 : IllgealArgumentException (부적절한 인수 값)

덱에 남은 카드 수가 적다면 : IllegalStateException (객체가 수행할 수 없는 상태임)

### 일반적인 규칙
- 어차피 실패했을 거라면 IllegalStateException을,
- 그렇지 않으면 IllegalArgumentException을 던지자.

