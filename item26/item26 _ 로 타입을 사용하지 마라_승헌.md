# 아이템 26 - 로 타입을 사용하지 마세요

## 선요약

- **로 타입을 사용하면 런타임에 예외**가 일어날 수 있으므로 사용하면 안 된다
- 로 타입은 제네릭이 도입되기 **이전 코드와의 호환성을 위해 제공**될 뿐이다

## 제네릭을 사용하는 이유

- 컴파일 시점에서 타입을 체크함으로써 **타입 안정성**을 제공함
- **타입체크와 형변환을 생략**함으로써 코드가 간결해짐

---

# 로 타입이란 ?

- 제네릭 타입에서 타입 매개변수를 전혀 사용하지 않는 경우를 의미함
- List<E>의 로 타입은 List다

```java
				// 제네릭을 지정하지 않은 로 타입
        List rawList = new ArrayList();  // Raw Type

        // 다양한 타입을 섞어서 넣을 수 있음 (타입 안정성 없음)
        rawList.add("Hello");
        rawList.add(123);
        rawList.add(true);

        // 꺼낼 때는 Object로 반환 → 형변환 필요
        String s = (String) rawList.get(0); // OK
        Integer i = (Integer) rawList.get(1); // OK
        // Integer num = (Integer) rawList.get(0); // 런타임 ClassCastException

        System.out.println(rawList);
```

### 로 타입을 사용하면 안 되는 이유

- 로 타입은 타입 선언에서 제네릭 타입 정보가 전부 지워진 것 처럼 작동한다.
- 즉, 컴파일 시점에서 타입을 체크하지 않고 런타임 시점에서 타입을 체크한다.

```java
// Stamp 인스턴스만 취급할 거다.
private final Collection stamps = ...;

// 실수로 동전 넣는다.
stamps.add(new Coin(...)); // 'unchecked call' 경고를 뱉지만 실행이 된다.
```

제네릭을 활용하면 타입 선언 자체에서 알 수 있다.

```java
private final Collection<Stamp> stamps = ... ;
```

타입 안전성이 확보되어서 컴파일 시에 다른 클래스를 넣으려하면 오류가 발생한다.

**로 타입을 쓰면 제네릭이 주는 안전성과 표현력을 잃는다.**

## 쓰지말자.

그럼 왜 만들어 놨을까? 호환성 때문이다. 제네릭이 자바 1.5에 나오면서 이전 코드와 호환을 해야하기 때문이다. 그래서 로 타입을 지원하고 제네릭 구현에는 소거 사용하기로 했다.

### 에러는 가능한 한 발생 즉시 우리 개발 환경에 보이는 것이 좋다 즉

→ 이상적으로는 컴파일 할 때 발견하는 것이 좋다… → 공감.

만약 로타입을 쓴다고 가정해보자

```jsx
public class Item26 {

    public static void main(String[] args) {
        List list = new ArrayList();
        list.add(10);
        list.add(20);
        list.add("30");

        Integer i = (Integer)list.get(2); //런타임 에러 발생

        System.out.println(list);
    }
}
```

이러면

즉 → 문제를 겪는 코드와 원인을 제공하는 코드가 물리적으로 상당히 떨어져 있다.

## 로 타입의 대안

1. 임의 객체를 허용한다.

```java
public class Item26 {

    public static void main(String[] args) {
        List<String> strings = new ArrayList<>();
        unsafeAdd(strings, Integer.valueOf(42)); // 컴파일 에러 발생
        String s = strings.get(0); // 컴파일러가 자동으로 형변환 코드를 넣어준다
    }

    private static void unsafeAdd(List<Object> list, Object o) {
        list.add(o);
    }
}
```

컴파일 구역에서 자체 차단을 시켜버림 

1. 비한정적인 와일드카드 타입

```java
public int numElementsInCommon(Set<?> s1, Set<?> s2) {
    int result = 0;
    for (Object o1 : s1)
        if (s2.contains(o1))
            result++;
    return result;
}
```

비 한정적인 와일드 카드 타입을 사용해 더욱 안전하게 코드를 작성할 수 있다.

## 로타입은 언제쓸까 ?

## 1. Class 리터럴에서의 로타입 사용 예외

- **Java 명세 제약**
    
    제네릭 파라미터를 붙인 타입은 .class에 사용할 수 없음
    
    - 허용:
        - List.class
        - String[].class
        - int.class
    - 금지:
        - List<String>.class
        - List<?>.class

```java

Class<?> c1 = List.class;
Class<?> c2 = String[].class;
Class<?> c3 = int.class;

// 컴파일 에러
Class<List<String>> c4 = List<String>.class;
Class<List<?>>    c5 = List<?>.class;
```

## 2. instanceof 연산자와 로타입 vs 와일드카드

- **런타임 동작**
    
    제네릭은 타입 소거(type erasure) 되어, raw 타입이나 <?> 모두 같은 클래스 판별
    
- **문법 차이**
    - 허용:
        
        ```java
        if (obj instanceof List)      { /* OK */ }
        if (obj instanceof List<?>)    { /* OK */ }
        ```
        
    - 금지:
    
    ```java
    if (obj instanceof List<String>) { /* 컴파일 에러 */ }
    ```