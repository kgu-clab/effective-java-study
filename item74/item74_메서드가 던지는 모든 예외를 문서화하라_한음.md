## item74. 메서드가 던지는 모든 예외를 문서화하라

### 사전 지식
- 검사 예외는 checked exception, 비검사 예외는 unchecked exception이라고도 부른다.
- checked exception 은 개발자가 명시한 예외로 반드시 try-catch 블록으로 처리하거나 throws 절에 선언해야 한다.
- unchecked exception 은 컴파일러가 강제하지 않으며 런타임에 발생하는 모든 예외를 지칭한다.

### 메서드 예외 문서화
- checked exception 은 항상 따로따로 선언하고, 각 예외가 발생하는 상황을 자바독의 @throws 태그로 문서화해야 한다.
- 단, Exception 이나 Throwable 같은 포괄적인 예외를 던지는 메서드는 피해야 한다.
- 아래는 LocalDateTime.java 의 문서화 예시다

```java
    /**
     * Obtains an instance of {@code LocalDateTime} from year, month,
     * day, hour, minute and second, setting the nanosecond to zero.
     * <p>
     * This returns a {@code LocalDateTime} with the specified year, month,
     * day-of-month, hour, minute and second.
     * The day must be valid for the year and month, otherwise an exception will be thrown.
     * The nanosecond field will be set to zero.
     *
     * @param year  the year to represent, from MIN_YEAR to MAX_YEAR
     * @param month  the month-of-year to represent, not null
     * @param dayOfMonth  the day-of-month to represent, from 1 to 31
     * @param hour  the hour-of-day to represent, from 0 to 23
     * @param minute  the minute-of-hour to represent, from 0 to 59
     * @param second  the second-of-minute to represent, from 0 to 59
     * @return the local date-time, not null
     * @throws DateTimeException if the value of any field is out of range,
     *  or if the day-of-month is invalid for the month-year
     */
```

- 자바 언어가 요구하는 것은 아니지만 unchecked exception 도 문서화하는 것이 좋다.
- unchecked exception 은 일반적으로 프로그래밍 오류이기 때문에, 명시해두면 프로그래머는 자연스럽게 해당 오류가 나지 않게 코딩하기 때문이다.
- unchecked exception 을 문서화할 때는 @throws 태그를 사용하되, 메서드 선언의 throws 절에는 포함하지 않는다.
- 한 클래스에 정의된 많은 메서드가 같은 이유로 같은 예외를 던진다면 그 예외를 클래스 설명에 추가하는 방법도 있다

### 핵심 정리
- 발생 가능한 예외를 모두 자바독의 @throws 태그로 문서화하자.
- checked exception 은 메서드 선언의 throws 절에 모두 명시하고, unchecked exception 은 문서화할 때만 @throws 태그로 명시한다.