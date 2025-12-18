## item61. 박싱된 기본 타입보다는 기본 타입을 사용하라

- 자바의 데이터 타입은 두 가지로 나뉜다
  - int, double, boolean 같은 기본타입
  - String, List 같은 참조 타입

- 기본 타입(이하 primitive type)에 대응하는 참조타입이 있으며 이를 박싱된 기본 타입이라고 한다 (이하 Wrapper class)
- int, double, boolean에 대응하는 Wrapper class 는 각각 Integer, Double, Boolean

- 오토박싱/언박싱 덕분에 두 타입을 구분하지 않고 사용할 수 있지만 분명한 차이가 있으니 어떤 타입을 사용하는지는 상당히 중요하다

### 기본타입과 박싱된 기본 타입의 차이
1. primitive type 은 값만 가지고 있으나, Wrapper class 는 그에 더해 식별성(identity)을 가진다
   - 두 인스턴스의 값이 같아도 서로 다르다고 식별될 수 있다
2. primitive type 의 값은 언제나 유효하나, Wrapper class 의 값은 null일 수 있다
3. primitive type 이 Wrapper class 보다 시간과 메모리 사용 측면에서 효율적이다.

### 문제 상황 1
```java
Comparator<Integer> naturalOrder = (i, j) -> i - j ? -1: (i == j ? 0 : 1);
```
- 같은 값을 가진 Integer 객체를 넣었을 떄, 값이 같으므로 0을 반환할 것으로 기대한다
- 하지만 실제로는 1을 출력한다
- 이유가 무엇일까
  - naturalOrder 는 첫번째 검사에서 값을 비교한다
  - 두번째 검사에서는 참조의 식별성을 검사하게 된다
  - 즉, i 와 j 가 같은 값을 가지더라도 서로 다른 객체이므로 false 를 반환하게 된다
- 값을 비교하기 위해 Wrapper class 에 == 연산자를 사용하면 안된다

### 문제 상황 2
```java
public class Unbelievable {
	static Integer i;
	
    public static void main(String[] args) {
        if (i == 42) {
            System.out.println("믿을 수 없군");
        }
    }
}
```
- 위 프로그램은 NullPointerException 을 던진다
- primitive type 과 Wrapper class 를 비교하면 자동으로 언박싱이 일어난다
- 근데 i 의 초기값이 null 이므로 예외가 발생하는 것이다

### 문제 상황 3
```java
public static void main(String[] args) {
	Long sum = 0L;
	for (long i = 0; i <= Integer.MAX_VALUE; i++) {
		sum += i;
	}
	System.out.printIn(sum);
}
```
- 박싱과 언박싱이 반복해서 일어나 성능이 매우 안좋다
- 사실상 sum+=i 는 sum = Long.valueOf(sum.longValue() + i); 와 같은 코드이다
- 따라서 불필요한 객체가 엄청나게 많이 생성된다

```java
class Scratch {
	public static void main(String[] args) {
		long primitiveTime = measurePrimitiveSum();
		long wrapperTime = measureWrapperSum();

		System.out.println("\n[결과 비교]");
		System.out.println("기본 타입 long 실행 시간: " + primitiveTime + "ms");
		System.out.println("Wrapper 클래스 Long 실행 시간: " + wrapperTime + "ms");
	}

	private static long measurePrimitiveSum() {
		long start = System.currentTimeMillis();

		long sum = 0L;
		for (long i = 0; i <= Integer.MAX_VALUE; i++) {
			sum += i;
		}

		long end = System.currentTimeMillis();
		System.out.println("\n기본 타입 long 합계: " + sum);
		return end - start;
	}

	private static long measureWrapperSum() {
		long start = System.currentTimeMillis();

		Long sum = 0L;
		for (long i = 0; i <= Integer.MAX_VALUE; i++) {
			sum += i;
		}

		long end = System.currentTimeMillis();
		System.out.println("Wrapper 클래스 Long 합계: " + sum);
		return end - start;
	}
}

```
```
기본 타입 long 합계: 2305843008139952128
Wrapper 클래스 Long 합계: 2305843008139952128

[결과 비교]
기본 타입 long 실행 시간: 545ms
Wrapper 클래스 Long 실행 시간: 2638ms
```

### Wrapper 클래스는 언제 써야 하는가?
1. 컬럭센의 원소, 키, 값으로 쓴다
2. 일반화 하여 말하면 타입 매개변수에는 무조건 Wrapper class 를 써야 한다
3. 리플렉션을 통해 메서드를 호출할 떄도 Wrapper class 를 써야 한다
