# 옵셔널 반환은 신중히 하라

### 자바 8 이전
메서드가 값을 반환할 수 없을 때
1. 예외를 던진다
- 문제점 : 비용이 비싸다.
2. null을 반환한다.
- NPE 발생 가능성이 커진다.

#  `Optional<T>`
- 자바 8부터 등장
- 값이 있을 수도, 없을 수도 있는 객체를 감싸는 래퍼 클래스

```java
public static <E extends  Comparable<E>>
    Optional<E> max(Collection<E> c) {
    if (c.isempty())
        return Optional.empty();
    //빈 옵셔널은 Optional.empty()로 : 빈 옵셔널을 반환
    E result = null;
    for (E e : c)
        if (result == null || e.compareTo(result) > 0)
            result = objects.requireNonNull(e);
            //e는 null이 아님을 보장
    
    return Optional.of(result);
    //값이 들어있는 옵셔널은 Optional.of(value)로 : e를 반환
    //null값도 허용하려면 .offNullable(value)지만.. 굳이?
}
```

# 사용
## 1. 메서드 실행시 '값이 없는 것'이 충분히 예상 가능한 결과중 하나일 때

```java
public static <E extends Comparable<E>> Optional<E> max(Collection<E> c) {
    if (c.isEmpty()) {
        return Optional.empty(); // '결과 없음'을 명시적으로 표현
    }
    // ... 최댓값을 찾아 Optional.of()로 반환
}
```
## 2. null 반환시 오류가 발생하기 쉬운 상황일 때
NPE 방지에 탁월하다.

# 사용 시 장점
### `Optional<T>`를 사용함으로써 반환 값이 없을수도 있다고 API사용자에게 명확히 알려준다.

### NPE 발생 가능성을 크게 줄여준다.

### 다양한 메서드를 제공하여 유연하게 활용이 가능하다.
- `filter`, `map`, `flatMap`, `ifPresent`

# 활용

### 1. 기본값을 정해둘 수 있다.
```java
.orElse("단어 없음..")
//값을 받지 못하였을 때 "단어 없음.." 문자열을 반환한다.
```
### 2. 원하는 예외를 던질 수 있다.

```java
.orElseThrow(TemperTantrumException::new);
//예외 팩터리(제조법)을 전달
```

### 3. 항상 값이 채워져 있다고 가정한다.
```java
max(Elements.NOBLE_GASES).get();
//.get()으로 바로 값을 꺼낸다.
```

## Stream
스트림을 사용한다면 Stream<Optional<T>>로 받아서, Stream<T>에 담아 처리하는 경우도 많다.
```
streamOfOptionals
    .filter(Optional::isPresent)//값 존재하는것만 필터링
    .map(Optinal::get)          //값을 꺼내 매핑
```

# `Optional` 반환 시 주의사항
### 성능 비용
객체를 생성하고 값을 꺼내는 데에 약간의 오버헤드가 발생하기 때문에 성능이 중요할 때에는 null을 반환하거나, 예외를 던지는 것이 더 좋을수도 있다.
### 반환값으로만 사용
컬렉션의 Key, Value, 배열의 원소로 사용하는것은 적절하지 않다.

(키가 없는 경우가 생기거나, 속이 빈 옵셔널인 경우 복잡성을 높여 혼란과 오류 가능성을 키운다.)
### 박싱된 기본 타입 금지
`Optional<Integer/Long>` 대신 `OptionallInt`, `OptionalLong` 같은 전용클래스 사용 (더 무겁다.)
### `isPresent`대신 함수형 메서드 사용
```java
if(opt.isPresent()) {
..
}
```
위의 코드보다는 `map()`, `orElse()`, `ifPresent()`같은 Optional의 메서드를 활용하는 것이 더 간결하고 관용적이다. 


# 핵심정리
`Optional<T>` : 값을 반환하지 못할 수도 있고, 호출할 때마다 그럴 가능성을 염두에 둬야 하는 메서드일 때 사용하는 일종의 '래퍼 클래스'이다.
- NPE 방지에도 탁월하다.
### 성능 저하가 뒤따르기도 한다.
### 반환값으로만 사용해야 한다.
### 기본타입은 전용메서드 사용
### `ifPresent` 대신 `map()`, `orElse()`등의 메서드 활용