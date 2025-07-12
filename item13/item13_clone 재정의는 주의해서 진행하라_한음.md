## item13. clone 재정의는 주의해서 진행하라

### Cloneable 인터페이스란?
- Cloneable은 복제해도 되는 클래스임을 명시하는 용도의 믹스인 인터페이스
- 하지만 의도한 목적을 제대로 이루지 못했다
  - clone 메서드가 Object에 protected로 정의되어 있기 때문
  - Cloneable을 구현하는 것만으로는 clone 메서드를 호출할 수 없다
- 하지만 Cloneable을 널리 쓰이고 있으므로 구현 방법을 알아두는 것이 좋다

### Clonable 인터페이스는 무슨 일을 할까?
- Object의 protected 메서드인 clone의 동작 방식을 결정한다
- Cloneable을 구현한 클래스에서 clone을 호출하면, 객체의 필드들을 하나 하나 복사한 객체를 반환한다
- 그렇지 않은 클래스에서는 CloneNotSupportedException 예외가 발생한다
- 실무에서 Cloneable을 구현한 클래스는 clone 메서드를 public으로 제공하며, 사용자는 당연히 복제가 제대로 이뤄리리라 기대한다.
- 그러나 생성자를 호출하지 않고도 객체를 생성할 수 있게 되므로, 깨지기 쉽고, 위험하고 모순적인 메커니즘을 만들 수 있다

```java
public class Main {
    public static void main(String[] args) {
        User original = new User("Alice", 25);
        User copy = original.clone();  // "복제가 당연히 잘 되겠지"라고 기대함
    }
}
```
### clone 메서드를 재구현 해보자
- 먼저 super.clone()을 호출해 Object의 clone 메서드를 사용한다
- 만약 모든 필드가 기본 타입이거나, 불변 객체라면, super.clone()으로 충분하다
```java
public class Main {
    public static void main(String[] args) {
        Point p1 = new Point(10, 20);
        Point p2 = p1;  // 복사 없이 그대로 참조

        System.out.println(p1 == p2);  // true — 복사할 이유가 없음, 불필요한 메모리 소비 방지
    }
}
```

```java
@Override
public PhoneNumber clone() {
    try {
        return (PhoneNumber) super.clone(); 
    } catch (CloneNotSupportedException e) {
        throw new AssertionError(); 
    }
}
```
- super.clone 호출을 try-catch로 감싸는 이유는, Object clone이 CloneNotSupportedException라는 checked 예외를 던지기 때문

```java
public class Stack{
	private Object[] elements;
	private int size;
	private static final int DEFAULT_CAPACITY = 16;
	
	// ..... 생략
}
```
- 이제 Stack 클래스에 clone 메서드를 추가해보자
- 반환된 Stack 클래스의 elements 필드는 원본 Stack 클래스의 elements 필드와 같은 배열을 참조한다
- 둘 중 하나를 수정하면 다른 하나도 영향을 받는다
- 프로그램이 이상하게 동작하거나 NullPointerException이 발생할 수 있다
- clone 메서드는 사실상 생성자처럼 동작하므로, 원복 객체에 아무런 해를 끼지지 않는 동시에 복제된 객체의 불변식을 보장해야 한다
- 따라서, clone 메서드에서 배열을 새로 할당하고, 원본 배열의 요소를 복사해야 한다
```java
@Override
public Stack clone() {
    try {
        Stack cloned = (Stack) super.clone();
        cloned.elements = elements.clone(); 
        return cloned;
    } catch (CloneNotSupportedException e) {
        throw new AssertionError();
    }
}
```

```java
public class HashTable implements Cloneable {
	private Entry[] buckets;
	
	private static class Entry {
		final Object key;
		Object value;
		Entry next;
		
		// ...생략
    }
	
	@Override
    public HashTable clone() {
        try {
            HashTable cloned = (HashTable) super.clone();
            cloned.buckets = buckets.clone();
            return cloned;
        } catch (CloneNotSupportedException e) {
            throw new AssertionError();
        }
    }
}
```
- Stack에서 처럼 HashTable에서도 버킷 배열의 clone을 재귀적으로 호출해보자
- 이렇게 하면 이 배열은 원본과 같은 연결 리스트를 참조하여 원본과 복제본 모두 예기치 않게 동작할 수 있다
- 따라서 아래와 같이 clone 메서드를 구현한다

```java
@Override
public HashTable clone() {
    try {
        HashTable cloned = (HashTable) super.clone();
        cloned.buckets = new Entry[buckets.length];
        for (int i = 0; i < buckets.length; i++) {
            if (buckets[i] != null) {
                cloned.buckets[i] = buckets[i].deepCopy(); 
            }
        }
        return cloned;
    } catch (CloneNotSupportedException e) {
        throw new AssertionError();
    }
}
```
- 먼저 Entry에 깊은 복사를 지원하도록 deepCopy 메서드를 추가한다
- 이 메서드는 Entry 객체를 복사하고, next 필드도 재귀적으로 복사한다
- 그 다음, 적절한 크기의 새로운 버킷 배열을 할당한 다음, 원래의 버킷 배열을 순회하며 깊은 복사를 수행한다
- 단, 재귀 호출은 리스트가 길어지면 스택오버플로는 일으킬 위험이 있으므로, deepCopy 메서드에서 반복문을 사용해 구현할 수도 있다

### 복잡한 가변 객체를 복제하는 마지막 방법
- 먼저 super.clone을 호출해 객체의 모든 필드를 초기 상태로 설정한 다음, 원본 객체의 상태를 다시 생성하는 고수준 메서드를 호출한다
- HashTable의 경우, buckets 필드를 새로운 버킷 배열로 초기화한 다음, put(key, value) 메서드를 사용해 원본 객체의 모든 엔트리를 복사하면 된다
- 이처럼 고수준 API를 활용하면 간단하고 우아한 코드를 얻지만, 저수준에서 처리할 때보다 느리다
  - 내부적으로 해시 계산, 충돌 처리, 리사이징 검사, 링크 설정 등 여러 로직을 포함하기 때문

### 실무에서 Cloneable 구현을 잘 사용하지 않는 이유
- 불변 객체 설계가 보편화되면서 복제 필요성이 줄어듦
- 복사 생성자와 복사 팩터리라는 더 나은 객체 복사 방식을 사용할 수 있다
```java
public static Yum newInstance(Yum yum) {
    // 생략...
}
```
- 복사 생성자와 복사 팩터리는 언어 모순적이고 위험천만한 객체 생성 메커니즘을 사용하지 않는다
- 엉성하게 문서화된 규약에 기대지 않고, 정상적인 final 필드 용법과도 충돌하지 않는다
- 불필요한 검사 예외를 던지지 않고 형변환도 필요치 않다
- 해당 클래스가 구현한 인터페이스를 인수로 받을 수도 있다 (코드 결합도 낮추고, 유연성 높임)
