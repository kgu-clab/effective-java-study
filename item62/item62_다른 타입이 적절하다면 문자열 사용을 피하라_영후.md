
String은 텍스트를 표현하기에는 좋지만, 이 외의 용도로도 흔히 사용하고는 한다. 하지만 문자열을 써서는 안되는 사례가 있음을 유의해야 한다.

### 문자열은 다른 값 타입을 대신하기에 적합하지 않다

- 많은 사람들은 데이터를 받을때 문자열을 사용하지만, 그 받은 데이터가 정말 문자열인 경우에만 String을 사용해야 한다.
- 기본 타입이든 참조 타입이든 적절한 값 타입이 있으면 그것을 사용하고, 없다면 직접 작성하라.

#### 1. 문자열은 열거 타입을 대신하기에 적합하지 않다.
- 상수를 열거할 때는 열거 타입이 낫다. (아이템 34)


#### 2. 문자열은 혼합 타입을 대신하기에 적합하지 않다.

- 여러 요소를 하나의 문자열로 표현하는건 좋지 않다.
```java
String compoundKey = className + "#" + i.next();
```
- 이렇게 결합하게 되면 개별로 접근해야 할 때 일일이 파싱해야 한다.
- 적절한 equals, toString, compareTo 메서드를 제공할 수도 없다.
- 이런 경우 private 정적 멤버 클래스를 사용한 전용 클래스를 만드는 것이 좋다.(아이템 24)

#### 3. 문자열은 권한을 표현하기에 적합하지 않다.
- 예를 들어 아래와 같이 문자열을 키로 사용해 각 스레드를 구분해보자
```java
public class ThreadLocal {
	private Thread Local() { } // 객체 생성 불가
	// 현 스레드의 값을 키로 구분해 저장한다.
	public static void set(String key, Object value);
	
	// (키가 가리키는) 현 스레드의 값을 반환한다.
	public static Object get(String key); }
```
- 이 방식은 이 키가 클라이언트 별로 고유함을 보장하지 못한다는 문제점이 있다.
- 만약 A클라이언트와 B클라언트가 같은 키를 공유하게 된다면 덮어씌워지는 문제가 생긴다.
- 이를 악용해서 같은키를 넣고 다른 객체를 꺼내오게 만드는 등 보안 이슈도 있다.


```java
public class ThreadLocal {

private ThreadLocal() { } // 객체 생성 불가

public static class Key { // (권한)
	Key() {}
}
// 위조 불가능한 고유 키를 생성한다.
public static Key getKey() {
	return new Key();
}

public static void set(Key key, Object value);
public static Object get(Key key);
}
```
- 따라서 문자열 대신 고유한 키를 사용해야 하며, 이때 키를 권한(capacity)이라고도 한다.
- 위의 코드에서는, get(),set()이 정적 메서드일 이유가 없으므로 Key의 인스턴스 메서드로 바꾸고, Key 자체가 스레드 지역 변수가 되도록 만들 수 있다.
- 이제 Key를 굳이 ThreadLocal로 감쌀 이유가 없으므로 벗기면 아래의 코드가 된다.

```java
public final class ThreadLocal {
	public ThreadLocal();
	public void set(Object value);
	public Object get();
}
```
- 이 코드에서는 Object를 반환하므로 사용 시 형변환이 필요하므로, 이를 위해 아래와 같이 바꾼다.

```java
public final class ThreadLocal<T> {
	public ThreadLocal();
	public void set(T value);
	public T get();
}

```
- 이것은 실제 java.lang.ThreadLocal과 흡사하다.
- 이것이 문자열 기반 API보다 안전하다.


> [! info]
더 적합한 데이터 타입이 있거나 새로 작성할 수 있다면, 문자열을 쓰고 싶은 유혹을 뿌리쳐라. 문자열은 잘못 사용하면 번거롭고, 덜 유연하고, 느리고, 오류 가능성도 크다. 문자열을 잘못 사용하는 흔한 예로는 기본 타입, 열거 타입, 혼합 타입이 있다.