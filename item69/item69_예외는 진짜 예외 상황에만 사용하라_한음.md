## item69. 예외는 진짜 예외 상황에만 사용하라

```java
try {
	int i = 0;
	while(true)
		range[i++].climb();
} catch (ArrayIndexOutOfBoundsException e) {
	
}
```
- 무슨 일을 하는 코드인지 알 수 없다
- 무한루프를 돌다가 배열의 끝에 다다르면 ArrayIndexOutOfException을 터트린다
- 예외를 써서 루프를 종료한 이유는 무엇일까
- 잘못된 추론을 근거로 성능을 높여보려고 한것이다

### 위 코드가 잘못된 이유
1. 예외는 예외 상황에 쓸 용도로 설계되었다 (최적화에 별로 신경쓰지 않은 코드다)
2. 코드를 try-catch 블록 안에 넣으면 JVM이 적용할 수 있는 최적화가 제한된다
   - 예를 들어 JIT 컴파일러는 자주 사용되는 코드는 단순화하여 최적화한다
   - 근데 예외를 사용하면 정상 경로와 예외 경로 둘 다 고려해 최적화를 하지 않는다
3. 배열을 순회하는 표준 관용구는 앞서 걱정한 중복 검사를 수행하지 않는다. JVM이 알아서 최적화해 없애준다

### 예외는 어떻게 써야하는가?
- 오직 예외 상황에서만 써야 한다.
- 절대로 일상적인 제어 흐름용으로 쓰여서는 안딘다
- 표준적이고 쉽게 이해되는 관용구를 사용하고, 성능 개선을 목적으로 과하게 머리를 쓴 기법은 자제하라
- 잘 설계된 API라면 클라이언트가 정상적인 제어 흐름에서 예외를 사용할 일이 없게 해야 한다.
- 반복문에서는 예외보다는 상태 검사 메서드를 사용하라

```java
for (Iterator<Foo> i = fooCollection.iterator(); i.hasNext(); ) {
    Foo foo = i.next();
}
```

### 상태 검사 메서드 대신 사용할 수 있는 선택지
- 여러 스레드가 동시에 접근할 수 있거나 외부 요인으로 상태가 변할 수 있다면 옵셔널이나 특정 값을 사용한다
- 성능이 중요한 상황에서 상태 검사 메서드가 상태 의존적 메서드의 작업을 일부 중복 수행한다면 옵셔널이나 특정 값을 사용한다
- 다른 경우에는 상태 검사 메서드 방식이 조금 더 낫다

```java
@NoArgsConstructor(
	access = AccessLevel.PRIVATE
)
public class UserValidator {

	public static void checkIsUserDeleted(User user) {
		if (user.isDeletedUser()) { // 삭제된 유저인지 상태 검사 메서드 사용
			throw new DeletedUserException(); // 삭제된 유저이면 예외 발생
		}
	}
}

```

```kotlin
fun getMyDetailAttendanceBySession(
   query: GetMyAttendanceBySessionQuery
): MyDetailAttendanceBySessionResponse {
   val myAttendanceQueryResult = attendancePersistencePort
      .findMyDetailAttendanceBySession(query)
      ?: throw AttendanceNotFoundException() // 옵셔널이 비어있으면 예외 발생

   return AttendanceMapper.toMyDetailAttendanceBySessionResponse(myAttendanceQueryResult)
}

override fun findMyDetailAttendanceBySession(
   query: GetMyAttendanceBySessionQuery
): MyDetailAttendanceQueryModel? = // 옵셔널 사용
   dsl.select(
      ATTENDANCES.STATUS,
      ATTENDANCES.ATTENDED_AT,
      SESSIONS.WEEK,
      SESSIONS.EVENT_NAME,
      SESSIONS.DATE,
      SESSIONS.PLACE,
   )
      .from(ATTENDANCES)
      .join(SESSIONS).on(ATTENDANCES.SESSION_ID.eq(SESSIONS.SESSION_ID))
      .join(MEMBERS).on(ATTENDANCES.MEMBER_ID.eq(MEMBERS.MEMBER_ID))
      .where(query.toCondition())
      .fetchOne {
         MyDetailAttendanceQueryModel(
            attendanceStatus = it[ATTENDANCES.STATUS]!!,
            attendedAt = it[ATTENDANCES.ATTENDED_AT]
               ?.atZone(ZoneId.of("Asia/Seoul"))
               ?.toInstant(),
            sessionWeek = it[SESSIONS.WEEK]!!,
            sessionEventName = it[SESSIONS.EVENT_NAME]!!,
            sessionDate = it[SESSIONS.DATE]!!,
            sessionPlace = it[SESSIONS.PLACE]!!,
         )
      }
```