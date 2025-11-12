## item56. 공개된 API 요소에는 항상 문서화 주석을 작성하라

- API를 쓸모 있게 하려면 잘 작성된 문서도 곁들여야 한다
- 자바독(Javadoc)이라는 유틸리티가 API 문서를 생성해준다

### 기본 자바독 작성 규칙
- public 클래스, 인터페이스, 메서드, 밀드 선언에 문서화 주석을 달아야 한다
- 메서드용 문서화 주석에는 해당 메서드와 클라이언트 사이의 규약을 명료하게 기술해야 한다
  - how가 아닌 what을 기술한다
  - @param 태그를 이용해 매개변수에 대해 기술하고, 반환 타입이 void가 아니라면 @return 태그를 이용해 반환 값에 대해 기술한다
  - 발생할 가능성이 있는 모든 예외에 대해 @throws 태그를 이용해 기술한다
```java
/**
 * @param  off the start offset of the binary representation.
 * @param  len the number of bytes to use.
 * @throws NumberFormatException {@code val} is zero bytes long.
 * @throws IndexOutOfBoundsException if the provided array offset and
 *         length would cause an index into the byte array to be
 *         negative or greater than or equal to the array length.
 */
```
- `<p>`나 `<i>` 같은 HTML 태그를 사용할 수 있는데 자바독은 문서화 주석을 HTML로 변환하기 때문이다
- 코드용 폰트는 `@code` 와 `<pre>` 태그를 사용하여 코드용 폰트로 렌더링 할 수 있게 하자
```java
/** An {@code IndexOutOfBoundsException} is thrown if the length of the array
 * {@code val} is non-zero and either {@code off} is negative, {@code len}
 * is negative, or {@code off+len} is greater than the length of
 * {@code val}.
 */
```
- 코드 가독성과 문서화 가독성 중 하나를 포기해야 한다면 문서화 가독성을 택하자
- 마침표 사용에 주의 하고 의도치 않은 마침표는 @literal로 감싸주자
```java
/**
 * A suspect, such as Colonel Mustard or {@literal Mrs. Peacock}.
 */
```
- API 요소의 색인은 자동으로 만들어지고 검색을 사용할 수 있는데, @index 태그를 사용하면 색인에 추가할 단어를 지정할 수 있다
```java
/* This method complies with the {@inex IEEE 754r} standard. */
```
- 패키지를 설명하는 문서화 주석은 package-info.java 파일에 담아야 한다
- 모듈 시스템을 사용한다면 module-info.java 파일에 모듈 설명을 담아야 한다
- 클래스 혹은 정적 메서드가 스레드 안전하든 그렇지 안든, 스레드 안전성에 대해 기술해야 한다

### 특정 클래스에 대한 자바독 작성 규칙
- 클래스를 상속용으로 설계할 때는 자기사용 패턴에 대해서도 문서로 남겨야 한다
  - 자기 사용 패턴은 @implSpec 태그를 사용해 기술한다
  - 메서드와 하위 클래스 사이의 계약을 설명하여 super 키워드를 이용해 호출할 때 어떻게 동작하는지 기술한다
- 헷갈리지 않기 위해 한 클래스 안에 요약 설명이 똑같은 멤버가 둘 이상이면 안된다
- 제네릭 타입이나 제네릭 메서드를 문서화할 때는 모든 타입 매개변수에 주석을 달아야 한다
```java
/**
 * An Object that maps keys to values.  A map cannot contain 
 * duplicate keys; each key can map to at most one value.
 * 
 * @param <K> the type of keys maintained by this map
 * @param <V> the type of mapped values
 */
public interface Map<K, V> { }
```
- 열거 타입을 문서화할 때는 상수들에도 주석을 달아야 한다
```java
public enum Operation {
    /** Addition. */
    PLUS,
    /** Subtraction. */
    MINUS,
    /** Multiplication. */
    TIMES,
    /** Division. */
    DIVIDE;
}
```
- 에너테이션 타입을 문서화할 떄는 멤버들에도 주석을 달아야 한다
```java
/**
 * Indicates that the annotated method is a test method that
 * must throw the designated exception to succeed.
 */
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface ExceptionTest {
    /** The class of the exception that should be thrown. */
    Class<? extends Throwable> value();
}
```
- 상위 타입의 문서를 상속 받아 표현하려면 {@inheritDoc} 태그를 사용한다
