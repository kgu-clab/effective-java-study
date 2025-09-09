# 매개변수가 유효한지 검사하라

메서드와 생성자 대부분은 입력 매개변수 값이 아래와 같은 특정 조건을 만족하기를 바란다.

- 인덱스 값은 음수이면 안 된다.
- 객체 참조는 null이 아니어야 한다.

## 1. 공개 메서드는 빠르게 검사하고, 친절히 문서화하자.

public이나 protected 메서드처럼 외부로 공개된 API는 매개변수 검사가 특히 중요하다.

매개변수 검사는 메서드 몸체가 시작되기 전에(최대한 빨리!) 수행하는 것이 좋다.

만약 검사 없이 로직을 수행하다가 문제가 생긴다면, 다음과 같은 골치 아픈 상황이 발생할 수 있다.

- 메서드 수행 중 모호한 예외를 던지며 실패
- 예외는 없지만 잘못된 결과를 반환
- 객체의 상태를 비정상적으로 만들어, 미래의 알 수 없는 시점에 관련 없는 오류를 발생(최악!)

### => 따라서, 메서드 시작 지점에서 매개변수를 확인하고, 잘못된 값이면 즉시 적절한 예외를 던져야 한다.

### 매개변수 제약은 반드시 문서화하자.
매개변수에 어떤 제약이 있는지, 제약을 어긴다면 어떤 예외가 발생하는지 API 사용자에게 명확히 알려줘야 한다.

던질 수 있는 예외 : ```IllegalArgumentException```, ```NullPointerException```, ```IndexOutOfBoundException```등..

### @throws 자바독 태그 활용
- item74 : 메서드가 던지는 모든 예외를 문서화하라
- 메서드가 던지는 예외는 그 메서드를 올바로 사용하는 데 아주 중요한 정보이므로 시간이 걸리더라도 검사 예외는 항상 따로따로 선언하고, 각 예외가 발생하는 상황을 @throws 태그로 정확히 문서화하자.

```java
/**
 *  (현재 값 mod m) 값을 반환한다. 이 메서드는
 *  항상 음이 아닌 BigInteger를 반환한다는 점에서 remainder 메서드와 다르다.
 *  
 * @param m 계수 (양수여야 한다.)
 * @return 현재 값 mod m
 * @throws ArithmeticException m이 0보다 작거나 같으면 발생한다.
 */
public BigInteger mod(BigInteger m) {
    if (m.signum() <= 0)
        throw new ArithmeticException("계수 (m)는 양수여야 합니다. " + m);
        // ...
}

//해당 메서드는 BigInteger의 객체에 대해 m으로 나눈 나머지를 반환하는 메서드이다.
```
m.signum 메서드를 호출시 m이 null이면 ```NullPointerException```을 던진다.

하지만 @throws에는 이에 대한 설명이 적혀있지 않다.
- BigInteger클래스가 '클래스 수준의 주석'으로 '매개변수로 null을 받으면 ```NullPointerException```을 던진다'고 이미 명시했기 때문이다.

- 클래스의 모든 public 메서드에 적용되는 규약이므로 모든 메서드에 일일이 적어둘 필요가 없다.



### 클래스 수준의 주석이란

```java
/**
 * Immutable arbitrary-precision signed decimal numbers.
 * ...
 * All methods in this class that accept a BigInteger argument
 * will throw a NullPointerException if the argument is null.
 *
 */
public class BigDecimal extends Number implements Comparable<BigDecimal> {
    // ...
}
```

"이 클래스의 모든 메서드는 BigInteger 인수가 null이면 NullPointerException을 던진다"고 클래스 전체의 공통 규칙으로 명시한 것이다.

- 모든 메서드에 @throws NullPointerException을 반복해서 다는 번거로움을 피하고 문서가 훨씬 깔끔해진다.


## null 검사를 위한 표준 유틸리티
### java.util.requireNonNull (Java 7+)
```java
this.strategy = Objects.requireNonNull(strategy, "전략");
```
null이면 NullPointerException을 던지고, 아니면 입력값을 그대로 반환해 바로 할당할 수 있다.
('예외 메시지 지정, 입력 그대로 반환, 사용과 동시에 null검사 수행'과 같은 장점들을 지니고 있다.)

### Objects의 범위 검사 메서드 (Java 9+)
- checkFromIndexSize
- checkFromToIndex
- checkIndex

리스트와 배열 전용으로 설계되어 인덱스 기반의 검사를 유용하게 도와준다.

## 2. 공개되지 않은 메서드라면..
공개되지 않은 private 메서드의 경우는, 패키지 제작자인 나 자신이 메서드 호출 상황을 통제할 수 있다.

- 이는 '유효한 값만이 메서드에 넘겨질 것을 보증해야 한다.'는 의미이다.

### 단언문(assert)을 사용해 매개변수 유효성 검증
```java
private static void sort(long a[], int offset, int length){
    assert a != null;
    assert offset >= 0 && a.length;
    assert length >= 0 && length <= a.length - offset;
    // ...
}
```
### 단언문이란
- 가정 표현으로써, 개발자가 코드를 작성하며 논리적으로 '참이라고 확신하는 조건'을 명시적으로 나타낸다.
- 이 가정이 깨진다면, 일반적인 예외가 아닌 ```AssertionError```라는 예외를 던진다. 이는 심각한 버그가 발생함을 알려준다.
- 런타임에 -ea 옵션을 켤 때만 동작하고, 일반적인 실행에서는 아무런 코드도 추가되지 않아 성능 저하가 전혀 없다.

### 일반적인 유효성 검사와 차이점

1. 실패하면 AssertionError를 던진다.
2. 런타임에 아무런 효과도, 성능저하도 없다.


## 3. 주의 : 나중에 사용될 매개변수는 즉시 검사하라
```java
//코드 20-1. 나쁜 예시 : 검사를 실행 시까지 미룬다.
static List<Integer> intArrayAsList(int[] a) {
    // a가 null이여도 예외를 던지지 않고 List 객체가 생성된다.
    return new AbstractList<>() {
        @Override
        public Integer get(int i) {
            // 이 시점에 a가 null이면 NullPointerException 발생
            return a[i]; 
        }
        // ...
    };
}
```
a가 null인 상태로 List를 반환, 이후 클라이언트가 List를 사용할 때가 되어서야 NullPointerException발생

=> 문제 발생 시점과 원인 제공 시점이 다르기 때문에 디버깅이 어려워진다.
```java
//좋은 예시 : 생성 시점에 즉시 검사.
static List<Integer> intArrayAsList(int[] a) {
    Objects.requireNonNull(a); // 여기서 null 검사를 수행
    // ...
}
```

위 코드처럼 Objects.requireNonNull()을 사용해 null 검사를 미리 수행하면, 예외가 즉시 발생하여 문제를 쉽게 파악할 수 있다.

## 4. 매개변수 유효성 검사 예외상황
### 1. 유효성 검사 비용이 지나치게 높을 때
예시: 회원가입 폼에서 사용자가 아이디를 입력할 때, 키보드를 누를 때마다 실시간으로 DB에 중복 아이디가 있는지 확인하는 경우. 'a', 'ab', 'abc'를 입력하면 그 짧은 순간에 DB 조회가 3번 일어난다. (DB 부하)
### 2. 유효성 검사가 실용적이지 않을 때
```java
List<Object> list = new ArrayList<>();
    list.add("apple");
    list.add("banana");
    list.add(123); // String과 Integer를 섞음

Collections.sort(list);
// 객체 리스트 정렬 메서드.
```
리스트 안의 객체들은 모두 상호 비교될 수 있어야 하지만, 비교 불가하다면 ClassCastException 예외를 던진다.

정렬 과정에서 이미 에외가 발생하니, 유효성 검사 코드를 작성할 필요가 없다.
### 3. 계산 과정에서 암묵적으로 검사가 수행될 때
```java
Integer number = null;
int sum = 0;
sum += number; // 여기서 NullPointerException 발생
```


### 이번 아이템을 '매개변수에 제약을 두라'라고 기억해서는 안된다. 오히려 가능하다면 '최대한 범용적으로' 설계해야한다.

# 핵심정리 및 요약
### 1. 매개변수 검사는 최대한 빨리(메서드 몸체 시작 전에) 수행하자.
### 2. 매개변수에 어떤 제약이 있는지, 어긴다면 어떤 예외가 발생하는지
- 공개 메서드는 '문서화'하자.
- 공개되지 않은 private 메서드는 단언문을 사용하자.
### 3. 나중에 사용할 매개변수는 미루지 말고 '즉시' 검사하자.
### 4. 매개변수 유효성 검사를 하지 않아도 되는 '예외상황'이 존재한다.
### 5. '매개변수에 제약을 두라'는 것이 아니다. 가능하다면 매개변수 제약은 '적을수록' 좋다.
