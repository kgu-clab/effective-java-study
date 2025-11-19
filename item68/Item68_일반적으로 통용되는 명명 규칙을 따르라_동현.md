# item68. 일반적으로 통용되는 명명 규칙을 따르라

# 1. 철자 규칙
### 패키지, 클래스, 인터페이스, 메서드, 필드, 타입 변수

## 패키지, 모듈
```java
//모두 소문자로
java.util. ~       //계층을 점으로 구분
com.google       //나의 조직 밖에서도 사용될 패키지라면 인터넷 도메인 이름을 역순으로
javax.accessibility. ~ // 표준라이브러리, 선택적 패키지들 : java, javax로 시작
java.awt. ~        //약어도 사용
```


## 클래스, 인터페이스
```java
public interface SpringDataJpaMemberRepository
public class JdbcMemberRepository
//파스칼 케이스 : 맨 첫글자를 대문자로 + 이후 단어의 첫글자도 대문자로
class HttpUrl
//모두 대문자인 단어들이여도 파스칼 케이스를 따르도록 한다.
```

## 메서드, 필드 이름
```java
public String getName
public void searchByCode
//카멜 케이스 : 맨 첫글자를 소문자로 + 이후 단어의 첫글자를 대문자로
```

## (예외)상수 필드
```java
static final MIN_VALUE
//대문자, 단어 사이는 언더바(_)로 구별
```

## 지역변수
```java
//카멜  케이스. 하지만 약어도 허용 (예를들어 반복문의 i)
for(int i=0; i<5; i++){ ... }
```

## 타입 매개변수
```java
//한 문자로 표현
타입 T
컬렉션 원소 타입 E
맵의 키 K
맵의 값 V
예외 X
등등 T, U, V or T1, T2, T3..
```

# 2. 문법 규칙
## 클래스, 인터페이스
```java
public class Thread implements Runnable { ... }
public class PriorityQueue<E> extends AbstractQueue<E> { ... }
public class Car { ... }
//객체를 생성할 수 있는 클래스 : 단수 명사 or 명사구

Collection / Collectors
//객체를 생성할 수 없는 클래스 : 복수형

public interface Runnable { .. }
public interface Comparable<T> { .. }
public interface Iterable<T> { .. }
//인터페이스 : ~able / ~ible
```

## 메서드
```java
public String createForm(..) {..}
public void readAllTrades(..) {..}
public void drawImage(..) {..}
//동작 수행 메서드 : 동사, 동사구

public boolean isBuy() {..}
public boolean isEmpty() {..}
//boolean 반환 메서드 : is로 시작. 드물게 has로 시작

public int size() {..}
public long getTime() {..}
//인스턴스 속성 반환 메서드 : 명사, 명사구, get
```

## 타입 변환 메서드

```java
public String toString() { .. }  //객체를 문자열로 '새로' 반환
public Object[] toArray() { .. } //객체를 배열로 '새로' 반환
// 다른 타입의 또다른 객체를 반환 
// toType형태 (toString, toArray)
List<String> originalList = new ArrayList<>();
originalList.add("a");
originalList.add("b");

Object[] newArray = originalList.toArray();
newArray[0] = "c";



public static <T> List<T> asList(T... a) { .. }
// 배열을 리스트'뷰'로 반횐. (원본 배열 공유)
// 배열을 리스트'처럼'사용할 수 있게 해줌.
// 객체 내용을 '다른 뷰'로 : asType형태 
String[] originalArray = {"Apple", "Banana", "Cherry"};
List<String> listView = Arrays.asList(originalArray);




public int intValue() { .. }
//객체의 값을 기본 타입 값으로 반환
//typeValue 형태
Integer numberObject = Integer.valueOf(100);
int primitiveInt = numberObject.intValue();
```

# 핵심요약 및 정리
## 1. 이름에는 다 나름 정해진 규칙이 있다.
## 2. 유동적으로 잘 살펴보며 다르게 적용되는 규칙들을 따르자.