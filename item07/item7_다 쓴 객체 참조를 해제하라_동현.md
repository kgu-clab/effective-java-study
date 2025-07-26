# item7 : 다 쓴 객체 참조를 해제하라

자바는 가비지 컬렉터(동적으로 할당했던 메모리 영역 중 필요없어진 영역을 자동으로 회수) 기능을 갖춘 언어이다.

하지만 그렇다고 메모리 관리에 더이상 아무 신경 쓰지 않아도 된다는 말은 아니다.

스택을 간단히 구현한 코드를 살펴보자.

```java
public class Stack {
	private Object[] elements;
	private int size = 0;
	private static final int DEFAULT_INITIAL_CAPACITY = 16;
	
	public Stack() {
		elements = new Object[DEFAULT_INITIAL_CAPACITY];
	}
	
	public void push(Object e) {
		ensureCapacity();
		elements[size++] = e;
	}
	
	public Object pop() {
		if (size == 0)
			throw new EmptyStackException();
		return elements[--size];
	}
	
	//배열 크기를 늘려야 할 때 대략 두 배씩 늘림
	private void ensureCapacity() {
		if (elements.lengtt == size)
			elements = Arrays.copyOf(elements, 2 * size + 1);
		}
	}
```

이 스택을 사용하는 프로그램을 오래 실행하다 보면 가비지 컬렉션과 메모리 사용량이 늘어나 성능저하 / OutOfMemoryError(메모리 공간 부족)이 발생할 수 있다.

해당 코드에서는 스택이 커졌다가 줄어들 때 스택에서 꺼내진 객체들을 회수하지 않는다.

스택이 객체들의 다 쓴 참조를 여전히 가지고 있기 때문이다.

## 해결법

해당 참조를 다 사용했을 때 null 처리(참조 해제)하면 된다.

```java
public Object pop() {
	if (size == 0)
		throw new EmptyStackException();
	Object result = elements[--size];
	elements[size] = null; // 다 쓴 참조 해제
	return result;
}
```

## 하지만 모든 객체를 null처리하는 것은 바람직하지 않다.

- 모두 null 처리를 하는것은 필요이상으로 지저분하게 만들 뿐, null처리는 예외적인 경우여야 한다.
- 가장 좋은 방법 : 그 참조를 담은 변수를 유효범위 밖으로 밀어내는 것

## null 처리는 언제 해야 할까? + Stack은 왜 메모리 누수에 취약할까?
- 스택은 자기 메모리를 직접 관리한다.
- 이 스택은 객체 ‘자체’가 아니라 객체 ‘참조’를 담는 elements 배열로 저장소를 만들어 원소들을 관리한다.
  - 활성 영역의 원소들 = 사용된다
  - 비활성 영역의 원소들 = 사용되지 않는다
    -  이 사실은 프로그래머만 알고 가비지 컬렉터는 알 수 없다.
    -  가비지 컬렉터는 비활성 영역에서 참조하는 객체도 똑같이 유효한 객체이기 때문
    -  프로그래머는 비활성 영역이 되는 순간 null 처리해서 쓰지 않을 것임을 가비지 컬렉터에게 알려야 함

### 캐시 또한 메모리 누수를 일으킨다.
### 해결법
- WeakHashMap : 캐시 외부에서 키를 참조하는 동안만 엔트리가 살아있는 캐시가 필요한 상황에서만 사용
- LinkedHashMap : 시간이 지날수록 엔트리의 가치를 떨어트리는 방식. 오래된 데이터는 삭제된다. removeEldestEntry메소드를 사용. 

### 리스너 / 콜백의 메모리 누수
```java
ex)
public class Main {
	public static void First(){
		System.out.println("첫번째메소드 호출");
		Callback(); //등록 후 해지하지 않음
	}
	
	public static void Callback(){
		System.out.println("콜백메소드 호출");
	}
	
	public static void main(String[] args){
		First();
	}
}
```
콜백을 등록만 하고 해지하지 않는다면 계속 쌓여갈 것이다. 이 때 약한 참조로 저장하면 가비지 컬렉터가 즉시 수거해 간다.

### 해결법
- 약한 참조 사용
- WeakHashMap : HashMap의 Element를 자동으로 GC한다.


# 내용 요약
### 가비지 컬렉터가 있다고 메모리 관리에 아예 신경을 꺼도 된다는 것은 아니다.
### 다 쓴 참조는 null처리 해서 가비지 컬렉터에게 알려야 한다.
### 그렇다고 모든 객체를 null처리 하는 것은 바람직하지 않다!
### 메모리 누수의 주범
- 메모리를 직접 관리하는 클래스 스택 → 다 쓴 참조 null처리
- 캐시 → WeakHashMap / LinkedHashMap
- 콜백 → WeakHashMap