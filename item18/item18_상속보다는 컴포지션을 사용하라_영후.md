
같은 패키지 내부에서 같은 프로그래머가 제어하는 클래스의 상속은 안전하다.
확장할 목적으로 설계된 클래스의 상속도 안전하다.
하지만 다른 클래스의 구체 클래스를 상속하는, **구현 상속**은 위험하다.

## 상속의 문제점

**상속은 캡슐화를 깨뜨린다.**

상위 클래스의 구현이 갑자기 달라졌을때, 이를 상속한 하위 클래스는 고장날 수 있다.
상위 클래스가 확장을 고려하며 문서화까지 해두지 않는 한, 하위 클래스가 항상 업데이트 하는 상위 클래스를 따라가야 한다.

예를 들어 아래와 같은 HashSet 코드가 있다고 가정하자.
```java
public class InstrumentedHashSet extends HashSet {
	// 추가된 원소의 수
	private int addCount = 0;
	
	public Inst rumentedHashSet() {
	}
	
	public InstrumentedHashSet(int initCap, float loadFactor) {
		super(initCap, loadFactor);
	}
	
	@Override
	public boolean add(E e) {
		addCount++; return super.add(e);
	}
	
	@Override
	public boolean addAll(Collection c) {
		addCount += c.sizeO;
		return super.addAll(c);
	}
	
	public int getAddCount() {
		return addCount;
	}
}
```
이 클래스에 addAll 메서드로 원소 3개를 더해보자.

```java
InstrumentedHashSet s = new InstrumentedHashSet<>();
s.addAll(List.of("틱", "탁탁", "펑"));
```
이제 getAddCount를 호출하면, 원하는 값인 3이 아닌 6을 반환한다.
- 왜냐하면 HashSet의 addAll 메서드는 add 메서드를 활용하였기 때문이다.
- addAll을 호출하면 addCount에는 3이 더해진다.
- 이후 addAll에서는 각 원소마다 InstrumentedHashSet의 add를 호출한다.
- 때문에 원소마다 addCount에 1을 더해 총 6이 되는 것이다.
- 이러한 addAll 메서드의 구현 방식은 당연히 문서화 되어있지 않을 것이다.
- 만약 이를 감안해 수정하더라도, 언제 내부 로직이 바뀌며 고장날지 모른다.

또한 보안 관련 문제도 있다.
- 하위 클래스에서는 모든 추가 메서드를 재정의해 원소를 검사하고 있다고 생각해보자.
- 근데 상위 클래스에서 새로운 원소 추가 메서드가 갑자기 생겨버리고 말았다.
- 해당 메서드는 하위 클래스에서 아직 재정의하지 못했고, 검사하지 못한 원소를 마음대로 하위 클래스에 추가될 수 있다.
- 실제로 HashTable과 Vector가 컬렉션 프레임워크에 추가되자 많은 보안 구멍이 생겼다.

만약 재정의는 전혀 하지 않고, 하위 클래스에서 새 메서드만 추가한다면 나아질 수 있다.
- 하지만 만약 상위 클래스에서 새로 만든 메서드와 시그니처가 겹치면 하위 클래스는 컴파일하지 않는다.
- 심지어 반환타입도 같다면 위와 똑같은 문제가 발생한다.


## 대신에
**컴포지션**
- 기존 클래스를 확장하는 대신, 새로운 클래스를 만들고 private 필드로 기존 클래스의 인스턴스를 참조하게 하자.(컴포지션, Composition)
- 새 클래스의 인스턴스 메서드들은 기존 클래스를 호출 후 결과를 반환한다.(전달, forwarding)
- 새 클래스의 메서드들은 전달 메서드(forwarding method)라 불린다.

컴포지션을 활용하면 새 클래스는 기존 클래스의 영향에서 완전히 벗어날 수 있다.

예시로 아래의 다시 구현한 코드를 보자

상속 대신 컴포지션을 사용한 래퍼 클래스
```java
public class InstrumentedSet extends ForwardingSet {
	private int addCount = 0;
	
	public InstrumentedSet(Set s) {
		super(s);
	}
	
	@Override
	public boolean add(E e) {
		addCount++;
		return super.add(e);
	}
	
	@Override
	public boolean addAll(Collection c) {
		addCount += c.sizeO;
		return super.addAll(c);
	}
	
	public int getAddCount() {
		return addCount;
	}
}
```
재사용할 수 있는 전달 클래스
```java
public class ForwardingSet implements Set {
	private final Set s;
	
	public ForwardingSet(Set s) {
		this.s = s;
	}
	
	public void clear()               {s.clear(); }
	public boolean contains(Object o) { return s.contains(o); }
	public boolean isEmpty()          { return s.isEmpty(); }
	public int size()                 { return s.size(); }
	public Iterator<E> iterator()     { return s.iterator(); }
	public boolean remove(Object o)   { return s.remove(o); }
	public boolean containsAll(Collection<?> c) { return s.containsAll(c); }
	public boolean removeAll(Collection c) { return s.retainAll(c); }
	public Object[] toArray()                 { return s.toArray();}
	public <T> T[] toArray(T[] a)             { return s.toArray(a);}
	@Override public boolean equals(Object o) { return s.equals(o);}
	@Override public int hashCode()           { return s.hashCode();}
	@Override public String toString()        { return s.toString();}
}	
```


- HashSet을 상속하는 대신 Set 인터페이스를 구현하여 훨씬 견고하고 유연해졌다.
- 상속 방식은 구체 클래스마다 따로 상속해야 하지만, 컴포지션 방식을 활용하면 어떤 Set 구현체든 받아낼 수 있다.

위의 코드에서 InstrumentedSet 클래스는 Set 인터페이스를 감싸고 있기 때문에 래퍼 클래스라고 불린다.
계층 기능을 덧씌운다는 의미에서 데코레이터 패턴이라고도 부른다.

## 상속은

- 상속은 반드시 진짜 하위 타입인 경우에만 사용하자.
- 컴포지션을 사용해야 할 곳에 상속을 사용하면, API는 묶이고, 성능은 제한되고, 사용자를 혼란스럽게 한다.
- 상속을 사용할 때에는 부모 클래스의 API에 결함이 없는지 자문하자.
- 해당 결함이 자손 클래스에 그대로 옮겨와도 되는지 자문하자.
- 컴포지션은 결함을 숨길 수 있지만, 상속은 안된다.


## 정리

> 상속은 강력하지만 캡슐화를 해친다는 문제가 있다. 상속은 상위 클래스와 하위 클래스가 순수한 is-a 관계일 때만 써야 한다. is-a 관계일 때도 안심할 수만은 없는게, 하위 클래스의 패키지가 상위 클래스와 다르고, 상위 클래스가 확장을 고려해 설계되지 않았다면 여전히 문제가 될 수 있다. 상속의 취약점을 피하려면 상속 대신 컴포지션과 전달을 사용하자. 특히 래퍼 클래스로 구현할 적당한 인터페이스가 있다면 더욱 그렇다. 래퍼 클래스는 하위 클래스보다 견고하고 강력하다.