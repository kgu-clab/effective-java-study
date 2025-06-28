## item11. equals를 재정의하려거든 hashCode도 재정의하라

### hashCode에 대한 기본 개념
- 해시값이란, 객체를 빠르게 식별하기 위한 짧은 숫자(또는 문자열)
- 해시 값은 왜 필요한가?
    1. 빠른 검색: HashMap, HashSet은 데이터를 저장할 때 hashCode()로 버킷(bucket) 위치를 결정, O(1) 시간에 값을 찾을 수 있음
    2. 비교 최적화: 객체를 일일이 다 비교하는 대신 해시값만 비교해서 빠르게 판단 가능
- `hashCode()` 란 무엇인가
  - Java에서 객체의 고유한 정수 값을 반환하는 메서드
  - `Object` 클래스에 정의되어 있고, 모든 클래스가 상속 받는다

---
### 개요
- equals를 재정의한 클래스 모두에서 hashCode도 재정의해야 한다
- 그렇지 않으면 hasCode 일반 규약을 어기게 되어, HashMap이나 HashSet에서 문제를 일으킬 것이다.

### hashCode 일반 규약
1. 애플리케이션이 실행되는 동안, hashCode 메서드는 몇 번을 호출해도 일관되게 항상 같은 값을 반환해야 한다
2. equals 메서드가 두 객체를 같다고 판단하면, 두 객체의 hashCode 메서드는 똑같은 값을 반환해야 한다
3. equals 메서드가 두 객체를 다르다고 판단했더라도, hashCode 메서드는 같은 값을 반환할 수 있다. 단, 다른 객체에 대해서는 다른 값을 반환해야 해시테이블의 성능이 좋아진다.

### 두 번째 조항에 대한 문제
- 논리적으로 같은 객체는 같은 해시코드를 반환해야 한다.
- 아이템10에서 봤듯이 equals는 물리적으로 다른 두 객체를 논리적으로 같다고 할 수 있다
- 하지만 Object의 기본 hasCode 메서드는 이 둘이 전혀 다르다고 판단해, 서로 다른 값을 반환한다 (규약 위반)

```java
public class Main {
    public static void main(String[] args) {
        Person p1 = new Person("Alice");
        Person p2 = new Person("Alice");

        System.out.println("equals: " + p1.equals(p2));         // true
        System.out.println("hashCode p1: " + p1.hashCode());    // 다름
        System.out.println("hashCode p2: " + p2.hashCode());    // 다름
    }
}
```

### hashCode를 재정의하지 않아 동치성 위반하는 예시
```java
Map<PhoneNumber, String> m = new HashMap<>();
m.put(new PhoneNumber(707, 867, 5309), "제니");
```
- `m.get(new PhoneNumber(707, 867, 5309))`를 호출하면, 어떤 값이 반환될까?
- "제니"가 반환될 것 같지만, 실제로는 null이 반환된다
- HashMap에 "제니"를 넣을 때 사용한 PhoneNumber와, 꺼내려할 때 사용한 PhoneNumber는 서로 다른 hashCode를 반환하기 때문이다
- 그 결과, get이 엉뚱한 해시 버킷에 가서 객체를 찾으려 한 것

### 최악의 hashCode 구현
```java
@Override public int hashCode() {
    return 42; // 모든 객체가 같은 해시코드를 반환
}
```
- 동치인 모든 객체에 똑같은 값만 내어주므로 모든 객체가 해시테이블의 버킷 하나에 담겨 마치 연결 리스트(LinkedList)처럼 동작
- 이 경우, 해시테이블의 성능은 O(n)이 되어버린다
- Java 8+ 부터는 TreeNode를 사용해 성능을 개선했지만, 여전히 O(log n)으로 느리다

### 좋은 hashCode 구현 요령
1. int 변수 result를 선언하고, 객체의 첫번째 핵심 필드 c를 2.a로 초기화한다
2. 나머지 핵심 필드에 대해 각각 다음 작업을 수행한다
   - 기본 타입 필드면 Type.hashCode(f)를 수행한다
   - 참조 타입 필드면 f.hashCode()를 수행한다 (재귀적 호출)
   - 참조 타입 필드가 null이면 0을 반환한다
   - 배열이라면, 각 핵심 원소의 해시코드를 계산한 다음 갱신한다
   - 배열의 모든 원소가 핵심 원소라면, Arrays.hashCode를 사용한다
3. result에 31을 곱하고, 핵심 필드의 해시코드를 더한다

### 전형적인 hashCode 구현
```java
@Override
public int hashCode() {
    int result = Short.hashCode(areaCode);
	result = 31 * result + Short.hashCode(prefix);
	result = 31 * result + Short.hashCode(lineNumber);
	result;
}
```
- 곱할 숫자를 31로 정한 이유는 홀수이면서 소수이기 때문
- 비결정적 요소는 전혀 없으므로 동치인 인스턴스들은 같은 해시코드를 가질 것이다
- Object 클래스의 hash 메서드를 사용할 수도 있지만, 배열 만들고.. 박싱 언박싱.. 느리다!
- hash는 성능이 민감하지 않은 상황에서만 사용하자


### 추가 고려 사항
- 클래스가 불변이고, 해시코드를 계산하는 비용이 크다면, 매번 새로 계산하기 보다는 캐싱하는 방식을 고려해야한다.
- 성능을 높인답시고 해시코드를 게산할 때 핵심 필드를 생략해서는 안된다. 해시 품질이 나빠져 해시테이블의 성능을 심각하게 떨어뜨림
- 해시코드 계산 규칙에 대해 자세한 내용을 공표하지 말자. 그래야 추후에 게산 방식을 바굴 수도 있다

