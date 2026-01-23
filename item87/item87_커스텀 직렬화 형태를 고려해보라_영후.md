
## 기본 직렬화

- 개발 일정에 쫓기는 경우 일단 동작만 하도록 구현한 후, 다음 릴리즈에 제대로 다시 구현하는 방식을 택하는 것이 좋다.
- 하지만 클래스가 Serializable을 구현하고 기본 직렬화 형태를 사용한다면 영원히 기본 직렬화 형태에 발이 묶이게 된다.(BigInteger 등)
- 따라서 유연성, 성능, 정확성 등 다양한 측면에서 충분히 고려해본 이후 기본 직렬화 형태가 현재 필요한 형태와 동일한 경우에만 기본 직렬화를 사용해야 한다.
- 기본 직렬화는 어떤 객체의 물리적 모델(객체 데이터, 객체와 연결된 객체들과 그 위상들)까지 담아내려 하지만, 이상적인 직렬화 모델은 오롯이 논리적인 모습만을 표현해야 한다는 점에서 거리가 있다.

### 기본 직렬화가 적합한 경우
- 객체의 물리적 표현과 논리적 내용이 같은 경우 기본 직렬화를 사용하더라도 무방하다.
```java
public clas Name implements Serializable {
	/**
	* 성. null이 아니어야 함.
	* @serial
	*/
	private final String lastName;
	
	/**
	* 이름. null이 아니어야 함.
	* @serial
	*/
	private final String firstName;
	
	/**
	* 미들네임. 없다면 null.
	* @serial
	*/
	private final String middleName;
	
	// 나머지 코드는 생략
}
```
- 성명은 논리적으로 위 3개의 문자열로 구성된다.
- 따라서 위의 필드들은 논리적 구성 요소를 정확히 반영한 경우이다.

- 또한 모두 private이지만 문서화 주석이 달려 있는데, 이 필드들은 직렬화에 포함되는 공개 API이기 때문이다.
- @serial 태그가 private 필드의 설명을 포함하라고 자바독에 알려주며, 이 내용은 API 문서에서 직렬화 형태를 설명하는 페이지에 기록된다.

#### readObject
- 기본 직렬화가 적합하더라도, 불변식 보장과 보안을 위하여 readObject 메서드를 제공해야 하는 경우가 많다. 예를 들어 Name 클래스에서는, lastName과 firstName 필드가 null이 아님을 보장해야 한다. (아이템 88,90)

### 기본 직렬화가 적합하지 않은 경우
```java
public final class StringList implements Serializable {
	private int size = 0;
	private Entry head = null;
	
	private static class Entry implements Serializable {
		String data;
		Entry next;
		Entry previous;
	}
	
	// 나머지 코드는 생략
}
```
- 위 클래스는 기본 직렬화 형태에 적합하지 않은 예로, 문자열 리스트를 표현하고 있다.
- 논리적으로는 일련의 문자열을 표현하지만, 물리적으로는 이중 연결 리스트를 사용한다.
- 만약 여기에 기본 직렬화를 사용하면, 각 노드의 양방향 연결 정보를 포함해 모든 Entry를 기록하게 된다.

#### 이 경우 문제점
1. **공개 API가 내부 표현 방식에 영구히 묶인다.**
	- 앞의 예에서 StringList.Entry가 공개 API가 되어 버리기 때문에, 추후 내부 프로세스를 변경하더라도 Entry를 비롯한 연결 리스트 코드를 절대 제거할 수 없게 된다.
2. **너무 많은 공간을 차지할 수 있다.**
	- 앞의 직렬화 형태는 모든 엔트리와 연결 정보를 기록하지만, 가치는 없다.
	- 이러한 정보는 저장하거나 전송 할 때 비효율적이다.
3. **시간이 너무 많이 걸린다.**
	- 위의 경우 다음 참조를 따라가는 정도로 충분하지만, 참조가 복잡해지면 마찬가지로 복잡한 그래프 순회를 거쳐야 한다.
4. **스택 오버플로를 일으킬 수 있다.**
	- 기본 직렬화 과정은 재귀 순회를 통해 객체 그래프를 기록하기 때문에 스택 오버플로를 일으킬 수 있다.
	- 이 스택 오버플로는 플랫폼 구현이나 플래그 등에 따라 발생하는 상황이 바뀌기도 한다.

#### 더 심각한 예시
- StringList 객체를 직렬화 하거나 역직렬화 하더라도 성능은 떨어질 지언정 정확성은 떨어지지 않는다.
- 하지만 불변식이 세부 구현에 따라 달라진다면 이 정확성도 깨질 수 있다.
- 예시로 해시 테이블은 해시코드를 계산하고 이를 토대로 버킷과 엔트리를 연결 짓는데, 그 계산 방식은 구현에 따라 달라지기 때문에 기본 역직렬화를 사용하면 객체가 훼손될 수 있다.

### 해결 방법
- StringList를 위한 합리적인 직렬화 방법은 아래와 같을 것이다.
```java
public final class StringList implements Serializable {
	private transient int size = 0;
	private transient Entry head = null; //transient를 통해 기본 직렬화에서 제외한다.
	
	// 이제는 직렬화되지 않는다.
	private static class Entry {
		String data;
		Entry next;
		Entry previous;
	}
	
	// 지정한 문자열을 이 리스트에 추가한다.
	public final void add(String s) { ... }
	
	/**
	 * 이 {@code StringList} 인스턴스를 직렬화한다.
	 *
	 * @serialData 이 리스트의 크기(포함된 문자열의 개수)를 기록한 후
	 * ({@code int}), 이어서 모든 원소를(각각은 {@code String})
	 * 순서대로 기록한다.
	*/
	private void writeObject(ObjectOutputStream s) throws IOException { 
		s.defaultWriteObject();
		s.writelnt(size);
		
		// 모든 원소를 올바른 순서로 기록한다.
		for (Entry e = head; e != null; e = e.next)
			s.writeObject(e.data);
	}
		
	private void readObject(ObjectlnputStream s)
			throws IOException, ClassNotFoundException {
		s.defaultReadObject();
		int numElements = s.readlnt();
		
		// 모든 원소를 읽어 이 리스트에 삽입한다.
		for (int i = 0; i < numElements; i++)
			add((String) s.readObject());
	}
	
	// 나머지 코드는 생략
}
```
- 이는 물리적인 표현 없이, 논리적인 구성만 담는다.
- 문자열의 개수를 적고, for문으로 순회하며 모든 문자열을 적는다.
- defaultRead/WriteObject는 static하지 않고 transient하지 않은 모든 필드를 스트림에 작성하는 메서드이다.
	- 여기서는 모든 필드가 transient이지만, 이렇게 해두어야 상호 호환 가능하다.
	- 또한 defaultWriteObject 메서드는 모든 인스턴스 필드를 직렬화 하기 때문에, 해당 객체의 논리적 상태와 무관한 필드라고 확신할 때만 transient 한정자를 생략해야 한다.(즉 커스텀 직렬화를 할때에는 대부분 transient로 선언한다.)
- 이 커스텀 직렬화를 사용하면 공간과 시간은 절반정도 소모하면서, 스택 오버플로우도 전혀 발생하지 않을 것이다.

## 동기화
- 기본 직렬화와는 별개로 동기화 메커니즘을 직렬화에도 적용해야 한다.
- 예를 들어 모든 메서드를 synchronized로 선언하여 스레드 안전하게 만든 객체는, writeObject도 synchronized로 선언해야 한다.
```java
private synchronized void writeObject(ObjectOutputStream s)
		throws IOException { 
	s.defaultWriteObject();
}
```

## UID
- 어떤 직렬화 형태를 택하든 직렬화 가능한 클래스에는 직렬 버전 UID를 부여하자
```java
private static final long serialVersionUID = ＜무작위로 고른 long 값>;
```
- 직렬 버전 UID는 클래스를 업데이트하며 기존 클래스와 직렬화 인스턴스가 호환되는 지 여부를 판단하는 데 사용된다.
- 이를 직접 부여하면 UID의 잠재적인 호환성 문제를 해결할 수 있고, 속도도 더 빨라진다.
- 만약 호환을 끊고 싶다면 UID 값을 바꿔주면 되며, 반대로 호환을 유지해야 한다면 구버전의 값을 그대로 가져와야 한다.