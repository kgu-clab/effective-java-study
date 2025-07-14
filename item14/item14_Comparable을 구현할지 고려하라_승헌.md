# 아이템 14 - Comparable을 구현할지 고려하라

## Comparable 인터페이스란?

객체 간의 **자연 순서(Natural Order)** 를 정의할 수 있게 해주는 인터페이스

### compareTo()의 특징

| 항목 | 설명 |
| --- | --- |
| 동치성 + 순서 | equals()는 동치성만, compareTo()는 **정렬 기준**까지 제공 |
| 제네릭 지원 | 타입 안정성을 보장함 |
| 정렬 가능 | Arrays.sort(), Collections.sort() 등에서 활용 |

작으면 **음수**, 같으면 **0**, 크면 **양수** 반환
비교 불가능한 타입은 ClassCastException 던짐

---

### 언제 유용한가?

- 정렬된 컬렉션(TreeSet, TreeMap) 사용 시
- 최소/최대값 계산 (Collections.min, max)
- 검색, 중복 제거 등에서 **성능 향상** 가능

```java
# 명령줄 인수 정렬 예 (중복 제거 + 알파벳 순)
java SortArgs Hello World Hello
# 출력 결과:
Hello
World

```

---

### 반드시 지켜야 할 세 가지 규칙

1. **대칭성**: sgn(x.compareTo(y)) == -sgn(y.compareTo(x))
2. **추이성**: x > y && y > z → x > z
3. **일관성**: x == y → x와 y는 z와 비교 시 결과가 같아야 함

### 왜 중요한가?

- 정렬된 컬렉션(TreeSet, TreeMap)에서 올바르게 작동하려면 필요
- Collections.sort, Arrays.sort 등에서 활용
- equals와 일관되지 않으면 **예상치 못한 동작 발생**

---

## 1. Comparable을 구현할지 고려해야 하는 이유

Comparable을 구현한다는 것은 **"이 클래스의 인스턴스들끼리는 자연스러운 순서가 있다"**는 걸 의미합니다.

즉, 개발자가 **이 클래스의 정렬 기준을 정했다**고 명시하는 겁니다.

### 왜 고려해야 할까?

자바는 정렬이 필요한 상황이 아주 많습니다.

- 리스트 정렬: Collections.sort(list)
- 배열 정렬: Arrays.sort(array)
- 자동 정렬되는 컬렉션: TreeSet, TreeMap
- 최댓값, 최솟값: Collections.max, Collections.min
- 중복 제거 및 정렬: 명령줄 인수를 정렬하거나 문자열 목록을 정렬할 때

이러한 상황에서 Comparable을 구현해두면 **모든 기능이 자동으로 잘 동작합니다.**

```java
Set<String> s = new TreeSet<>();
Collections.addAll(s, args);
System.out.println(s);
```

String이 Comparable을 구현하고 있기 때문에 아무 설정 없이도 알파벳순으로 정렬됩니다.

---

### compareTo() 메서드가 equals보다 더 강력한 이유?

- equals()는 단순히 "같다/다르다"만 판단
- compareTo()는 "작다/같다/크다" + 정렬의 우선순위까지 판단

즉, compareTo()는 **동치성 + 순서성**을 모두 판단합니다

---

### compareTo 규약을 잘못 구현하면?

예를 들어 `compareTo()`가 대칭성이나 추이성을 지키지 않으면,

- 정렬된 컬렉션에서 예외 발생
- 정렬 결과가 예측 불가능
- 중복 제거가 의도대로 안 됨
- 검색 성능 저하

마치 equals()나 hashCode() 규약을 안 지킨 것만큼 심각한 버그 발생 가능

---

### 번외

## Comparable vs Comparator

### Comparable (자연순서)

- 클래스 내부에 직접 implements Comparable<T> 구현
- 1개를 기준으로만 가능하고
- int compareTo(T o)
- 항상 동일한 기준으로 정렬됨.

```java
public class Person implements Comparable<Person> {
    public int age;
    public int compareTo(Person other) {
        return Integer.compare(this.age, other.age);
    }
}
```

---

### Comparator (사용자 정의 순서)

- 클래스 외부에서 별도로 정의
- 여러개 정의 가능
- int compare( T o1, T o2)
- 상황에 따른 다른 정렬 기준 제공 가능

```java
Comparator<Person> byName = Comparator.comparing(p -> p.name);
Comparator<Person> byAgeDesc = (p1, p2) -> Integer.compare(p2.age, p1.age);
```

**Comparable**: 자기소개서에 "나는 나이 순으로 정렬돼야 해요"라고 직접 써놓은 사람

**Comparator**: "이번엔 이름순으로, 다음엔 키순으로 정렬하자"고 외부 기준을 주는 것

---

핵심정리

순서를 고려해야 하는 값 클래스를 작성한다면 꼭 Comparable 인터페이스를 구현하
여, 그 인스턴스들을 쉽게 정렬하고, 검색하고, 비교 기능을 제공하는 컬렉션과 어우러
지도록 해야 한다. compareTo 메서드에서 필드의 값을 비교할 때 <와 > 연산자는 쓰
지 말아야 한다. 

그 대신 박싱된 기본 타입 클래스가 제공하는 정적 compare 메서드나
Comparator 인터페이스가 제공하는 비교자 생성 메서드를 사용하자.