## item29. 이왕이면 제네릭 타입으로 만들라

### 자주 사용하는 타입 매개변수 명칭
| 타입 매개변수   | 의미               | 사용 예시               |
| --------- | ---------------- | ------------------- |
| `T`       | Type (일반적인 타입)   | `List<T>`, `Box<T>` |
| `E`       | Element (컬렉션 요소) | `List<E>`, `Set<E>` |
| `K`       | Key (Map의 키)     | `Map<K, V>`         |
| `V`       | Value (Map의 값)   | `Map<K, V>`         |
| `N`       | Number (숫자 타입)   | `GenericNumber<N>`  |
| `S, U, V` | 복수의 타입 (보조 타입)   | `Pair<T, U>`        |


### 일반 클래스를 제네릭 클래스로 만드는 방법 (아이템7 Stack 예시)
1. 클래스 선언에 타입 매개변수를 추가한다
```java
public class Stack {
	// BEFORE
}

public class Stack<E> {
    // AFTER : 제네릭 타입 E를 추가
}
```
2. Object를 적절한 타입 매개변수로 바꾸고 컴파일 한다
```java
public class Stack {
	private Object[] elements;
}

public class Stack<E> {
	private E[] elements; // E 타입으로 변경
}
```
- 이 때 오류가 발생한다
- E와 같은 실체화 불가 타입으로는 배열을 만들 수 없다
- 해결책은 두가지다 
  1. 제네릭 배열을 생성하는 제약을 대놓고 우회 하는 것 
   ```java
    @SuppressWarnings("unchecked")
    public Stack() {
        elements = (E[]) new Object[DEFAULT_INITIAL_CAPACITY]; 
    }
   ```
     - Object 배열을 생성한 다음 제네릭 배열로 형변환
     - 하지만 타입 안전하지 않다
     - 타입 안전함을 증명했다면 `@SuppressWarnings("unchecked")` 로 경고를 숨긴다
  2. elements를 Object[]로 바꾸는 것이다
  ```java
    public E pop() {
        if (size == 0) throw new EmptyStackException();
        
        @SuppressWarnings("unchecked") E result = (E) elements[--size]; 
  
        elements[size] = null; 
        return result;
    }
    ```
     - 배열이 반환한 원소를 E로 형변환하면 경고가 뜬다
     - E는 실체화 불가 타입이므로 런타임에 이뤄지는 형변환이 안전한지 증명할 방법이 없다
     - 이번에도 증명하고 경고를 숨김 처리 할 수 있다
     - 비검사 형변환을 수행하는 할당문에서만 숨겨보자

### 제네릭 배열 생성을 제거하는 두 방법의 장단점
- 첫번째 방법
  - 가독성이 좋다
  - 배열의 타입을 `E[]`로 선언하여 오직 E 타입 인스턴스만 받음을 보장한다
  - 코드도 더 짧다
  - 배열의 런타임 타입이 컴파일 타임 타입과 달라 힙 오염(Heap Pollution)을 일으킨다
- 두번째 방법
  - 배열에서 원소를 읽을 때마다 형변환을 해주어야 한다
  - 때문에 현업에서는 첫 번쨰 방법을 더 선호한다
  - 힙 오염을 일으키지 않는다

### 제네릭 타입 안에서 배열을 사용해도 되는 이유
- 아이템 28 '배열보다는 리스트를 우선하라' 와 모순되어 보인다
- 자바가 리스트를 기본 타입으로 제공하지 않기 때문이다
- HashMap도 성능을 높일 목적으로 배열을 사용한다

### 타입 매개변수에 관하여
- 대다수의 제네릭 타입은 타입 매개변수에 제약을 두지 않는다
- 단, 기본 타입은 사용할 수 없다 `Stack<int>` 같은..
- 타입 매개변수에 제약을 두는 경우도 있다
```java
class DelayQueue<E extends Delayed> implements BlockingQueue<E>
```

### 핵심 정리
- 제네릭으로 변환할 수 있는 타입은 제네릭 타입으로 변환하여 타입 안정성과 편의성을 확보하자



