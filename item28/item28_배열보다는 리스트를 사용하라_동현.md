# item25 : 배열보다는 리스트를 사용하라

## 배열과 제네릭 타입의 차이
### 1. 공변 vs 불공변
- 공변 : 부모인 Super[]와 자식인 Sub[] 배열이 있다면 함께 변한다.

#### 배열은 공변이다.
```java
Object[] arr = new String[3];  // String[]을 Object[]로 취급 가능
arr[0] = 123;  // 런타임에 ArrayStoreException 발생!
//컴파일은 되지만 런타임때 오류!
```

#### 제네릭은 불공변이다.
```java
List<Object> list = new ArrayList<String>();
//애초에 컴파일부터 안됨 => 더 안전함!
```
배열 : 실수를 런타임에서야 알게 된다.

리스트 : 실수를 컴파일 때 바로 알아차릴 수 있다.

### 2. 실체화 vs 타입소거
#### 배열의 특징
- **실체화** : **'런타임에'** 자신이 담는 원소의 타입을 인지하고 확인한다. => ArrayStoreException
#### 제네릭의 특징
- **타입 소거** : **'컴파일 후'** 타입 정보가 사라진다.
  - 타입 정보를 **컴파일 타임에만** 검사하며 런타임에는 알 수조차 없다.



## 💡 예시 : 생성자에서 컬렉션을 받는 Chooser 클래스
### 1단계 : 프로토타입
```java
public class Chooser {
    private final Object[] choiceArray;

    public Chooser(Collection choices) {
        choiceArray = choices.toArray();
    }

    public Object choose() {
        Random rnd = ThreadLocalRandom.current();
        return choiceArray[rnd.nextInt(choiceArray.length)];
    }
}
```
### 2단계 : 제네릭 사용! (But 배열 사용)
```java
public class Chooser<T> {
    private final T[] choiceArray;

    public Chooser(Collection<T> choices) {
        // (T[]) 형변환 : 안전하지 않은 형변환으로 경고
        choiceArray = (T[]) choices.toArray();
    }
    //choose는 그대로
}
```
### → 경고 발생.
- 제네릭은 원소 타입 정보가 소거되어 런타임에는 타입을 알 수 없음.
- T의 타입을 알 수 없어 동작은 하지만 안전성을 보장하지는 못한다. (컴파일러 경고)
- 안전하다고 확신한다면 주석을 남기고 애노테이션을 달아 경고를 숨기자.

### 3단계 : 배열 대신 리스트를 사용!
```java
public class Chooser<T> {
    private final List<T> choiceList;

    public Chooser(Collection<T> choices) {
        //경고 없음. 안전함!
        choiceList = new ArrayList<>(choices);
    }

    public T choose() {
        Random rnd = ThreadLocalRandom.current();
        return choiceList.get(rnd.nextInt(choiceList.size()));
    }
}
```
### → 배열 대신 리스트 사용시
- 컴파일러 경고 X
- 타입 안전 보장
- 더 명확하고 읽기 쉬운 코드

## 기억할 점
- 에러는 컴파일 시점에 잡는 게 더 좋다.
- 제네릭과 배열은 안어울린다.

## 배열 or 리스트?
- 배열은 성능이 중요하고, 타입이 확실할 때에만 사용하자.
- 그 외에 상황에서는 리스트가 더 안전하고 명확하다.

## 핵심 정리
- 배열 : 공변 / 실체화
- 제네릭 : 불공변 / 타입 정보 소거
- 배열과 제네릭(리스트)은 서로 반대의 특성을 지님.

  |     | 배열        | 리스트      |
  |-----|-----------|----------|
  | 컴파일 | 타입 정보 검사X | 타입 정보 검사 |
  | 런타임 | 타입 정보 검사  | 타입 정보 소거됨 |
- 웬만한 상황에서 배열보다는 리스트를 사용하자.