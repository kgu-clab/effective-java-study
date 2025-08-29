- 제네릭 메서드
:메서드 내부에서 사용할 타입을 호출 시점에 외부에서 결정할 수 있도록 만든 메서드
: 제네릭 클래스가 존재하듯, 메서드도 제네릭으로 만들 수 있음
:정적 유틸리티 메서드는 보통 제네릭
*정적 유틸리티: 객체생성X, 클래스명으로 호출, static 메서드
ex) Collections의 알고리즘 메서드(binarySearch, sort)등은 모두 제네릭 메서드

- 제네릭 메서드 작성법
  ex1)문제 코드(컴파일은 되지만 경고가 발생)

```java
public static Set union(Set s1, Set s2) {
Set result = new HashSet(si);
result.addAll(s2);
return result;
}
```

→문제 원인: Set과 HashSet을 타입 인자 없이 사용하게 되면서 타입 안정성을 잃음

→수정 방법: 해당 메서드를 타입 안전하게 만들어야 함
1. 메서드 선언에서 세 집합(입력 두개(s1, s2), 반환 1개(result))의 원소 타입을 타입 매개변수로 명시
2. 메서드 안에서도 이 타입 매개변수만 사용하도록 수정
3. 타입 매개변수 목록은 메서드의 제한자와 반환 타입 사이에 둠
   *타입 매개변수 목록: 타입 매개변수들을 선언함
   Q)타입 매개변수 목록을 메서드 선언시에 명시하는 이유는?
   A)해당 메서드 안에서 E라는 이름의 타입 변수를 사용할 것임을 컴파일러에게 알려주기 위함

    ex2)수정 코드

```java
public static <E> Set<E> union(Set<E> s1, Set<E>s2) {
    Set<E> result=new HashSet<>(s1);
    result.addAll(s2);
    return result;
}
```

→타입 매개변수 목록은 <E>이고 반환 타입은 Set<E>

→union 메서드의 집합 3개(입력 2개, 반환 1개)의 타입이 모두 같아야 함
이유: 제네릭 메서드 안에서는 메서드 선언부에 명시한 타입 매개변수만 사용할 수 있기 때문
*이는 한정적 와일드카드 타입을 사용하여 유연하게 개선할 수 있음

ex3)수정 코드를 호출한 예제 코드

```java
public static void main(String[] args_){
    Set<String> guys = Set.of("톰", "딕","해리");
    Set<String> stooges = Set.of("래리", "모에", "컬리");
    Set<String> aflCio = union(guys, stooges);
    System.out.printIn(aflCio);
}
```

→프로그램 실행 결과: [모에, 톰, 해리, 래리, 컬리, 딕]

- 제네릭 싱글턴 팩터리
  필요성:
  불변 객체를 여러 타입으로 활용할 수 있게 만들어야 할 때, 제네릭은 런타임에 타입 정보가 소거 됨. 따라서 하나의 객체를 어떤 타입으로든 매개변수화할 수 있음
  →이것이 가능하게 하기 위해 요청한 타입 매개변수에 맞게 매번 그 객체의 타입을 바꿔주는 정적 팩터리가 필요, 이를 ‘제네릭 싱글턴 팩터리’ 라고 함

- 제네릭 싱글턴 팩터리 사용 예제→항등함수를 담은 클래스
  :항등함수 객체는 상태가 없으므로 매번 새로 생성하는 것은 낭비
  →제네릭의 소거방식을 활용해 제네릭 싱글턴을 사용해 생성할 수 있음

:예제 코드1

```java
private static UnaryOperator<Object> IDENTITY_FN = (t) -> t;

@SuppressWarnings("unchecked")
public static <T> UnaryOperator<T> identityFunction() {
    return (UnaryOperator<T>) IDENTITY_FN;
}
```

→IDENTITY_FN을 UnaryOperator<T>로 형변환하면 비검사 형변환 경고가 발생하지만 항등함수(입력 값을 변형없이 그대로 반환함)이기 때문에 T가 어떤 타입 이든 타입 안전성이 깨지지 않아 UnaryOperator<T>를 사용해도 타입 안전

→SuppressWarnings 애너테이션을 추가하여 오류나 경고 없이 컴파일 하도록 함

:예제 코드 2(위 코드 활용 ver)

```java
public static void main(String[] args) {
String[] strings = {"삼베","대마", "나일론"};
UnaryOperator<String> sameString = identityFunction();
for (String s : strings)
System.out.printIn(sameString.apply(s));
Number[] numbers = { 1, 2.0, 3L };
UnaryOperator<Number> sameNumber = identityFunction();
for (Number n : numbers)
System.out.println(sameNumber.apply(n));
}
```

→코드1의 제네릭 싱글턴을 UnaryOperator<String>과 UnaryOperator<Number>로 사용

→identityFunction() 호출 시 <T>가 각각 String과 Number로 추론

→apply(s)와 apply(n) 함수로 입력 문자열과 숫자를 그대로 반환하여 원본 값 출력

- 재귀적 타입 한정
  : 자기 자신이 들어간 표현식을 사용하여 타입 매개변수의 허용 범위를 한정할 수 있음
  :예제코드

    ```java
    public interface Comparable<T> {
    int compareTo(T o);
    }
    ```

  →타입 매개변수 T는 Comparable<T>를 구현한 타입이 비교할 수 있는 원소의 타입을 결정하게 됨
    
  →모든 타입은 자신과 같은 타입의 원소와만 비교할 수 있기 때문에, String은 Comparable<String>을 구현하고 Interger는 Comparable<Interger>을 구현하게 됨
  : 주로 타입의 자연적 순서를 정하는 Comparable 인터페이스와 함께 쓰임
  : Comparable을 구현한 원소의 컬렉션을 입력받는 메서드들은 보통 그 원소들을 정렬, 검색, 최솟값/최댓값 탐색을 위해 사용됨→이를 위해 컬렉션에 담긴 모든 원소가 상호 비교될 수 있어야 함
  :예제 코드

    ```java
    public static <E extends Comparable<E>> E max(Collection<E> c) {
        if (c.isEmptyO)
            throw new IllegalArgumentException("컬렉션이 비어 있습니다");
    E result = null;
    for (E e : c)
        if (result = null || e.compareTo(resuIt) > 0)
            result = Objects.requireNonNull(e);
    return result;
    }
    ```

  →타입 한정인 <E extends Comparable<E>>는 “모든 타입 E는 자신과 비교할 수 있다”라는 의미
  →컬렉션에 담긴 원소의 자연적 순서를 기준으로 최댓값을 계산하게 됨