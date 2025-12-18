# 아이템 52 - 다중 정의는 신중히 사용해라

### 컬렉션을 집합, 리스트, 그 외로 구분하고자 할 때

```java
public class CollectionClassifier {
	public static String classify(Set<?> s) {
		return "집합,,;
	}
	public static String classify(List<?> 1st) {
		return ''리스트,,;
	}
	public static String classify(Collection<?> c) {
		return "그 외,,;
	}
	public static void main(String[] args) {
		Collection<?>[] collections = {
			new HashSet<String>(),
			new ArrayList<BigInteger>(),
			new HashMap<String, String>().values()
		}；
		for (Collection<?> c : collections)
		System.out.println(classify(c));
		}
}
```

### 우리가 예상하는 건

“집합”, “리스트”, “ 그 외” 로 되어야 할 것 같지만 

”그 외” x 3 번이 나온다

왜 그런걸까 ?

## 이유는 바로 다중정의된 세 classify중 어느 메서드를 호출할지가 컴파일 타임에 정해지기 때문이다.

- 런타임에는 타입이 매번 달라짐
- but 호출할 메서드를 선택하는 데는 영향을 주지 못함.

→ 그래서 컴파일타임의 매개변수 타입을 마지막으로 기준을 잡아 버리기 때문에 “그 외” 가 출력이 된다.

### 재정의한 메서드는 동적으로 선택되고, 다중정의한 메서드는 정적으로 선택 되기 때문이다.

## 하위에서 정의한 메서드가 실행 되는경우

```java
class Wine {
	String nameO { return "포도주"; }
	}
	class SparklingWine extends Wine {
		(aOverride String nameO { return "발포성 포도주"; }
	}
	class Champagne extends SparklingWine {
		(aOverride String name() { return "샴페인"; }
	}
	public class Overriding {
		public static void main(String[] args) {
			List<Wine> wineList = List.of(
				new Wine(), new SparklingWine(), new Champagne());
				
				for (Wine wine : wineList)
				System.out. printIn (wine.name());
		}
	}
```

### 이 프로그램은 “포도주”, 발포성 포도주, 샴페인 이 3개를 차례로 출력한다.

CollectionClassifier 예에서 프로그램의 원래 의도는 매개변수의 런타임 타입에 기초해 적절한 다중정의 메서드로 자동 분배하는 것이었다.

이 문제는 정적 메서드를 사용해도 좋다면  CollectionClassifier의 모든 classify 메서드를 하나로 합친 후 instanceof로 명시적으로 검사하면 말끔히 해 결된다

```java
public static String classify(Collection<?> c) {
	return c instanceof Set ? "집합" : 
					c instanceof List ? "리스트" : "그 외";
	}
```

 API 사용자가 매개변수를 넘기면서 어떤 다중정의 메서드가 호출될지를 모른다면 프로그램이 오작동하기 쉽다. → 그러니 최대한 다중정의가 혼동을 일으키는 상황을 피해야한다.

- 제일 좋은 방법은 아무래도 매개변수 수가 같은 다중정의는 만들지 말자.
- 다중정의하는 대신 가장 좋은 solution은 메서드 이름을 다르게 지어주자.

```java
public class SetList {
	public static void main(String!] args) {
		Set<Integer> set = new TreeSetoO;
		List<Integer> list = new ArrayListoO;
		
		for (int i = -3; i < 3; i++) {
			set.add(i); // -3, -2, -1 ,0 ,1 ,2 ,3
			list.add(i); // -3, -2, -1 ,0 ,1 ,2 ,3
		}
		
		for (int i = 0; i < 3; i++) {
			set.remove(i);
			list.remove(i);
		}
		
		System.out.println(set + " " + list);
		}
}
```

우리는 이걸 실행하면 기대를 [-3,-2,-1], [-3,-2,-1]로 할 것이다.

하지만 그렇지 않다

실제 출력값은 [-3,-2,-1] , [-2,0,2] 가 나온다

list는 앞 부분이 빠지면 다시 밀어서 채워진다

```
-3, -2, -1 ,0 ,1 ,2 ,3 →  0번째 제거 →  -2, -1 ,0 ,1 ,2 ,3

 -2, -1 ,0 ,1 ,2 ,3 → 1번째 제거 → -2,0 ,1 ,2 ,3 

-2,0 ,1 ,2 ,3  → 2번째 제거 → -2,0 ,2 ,3

3번째 제거 → -2,0,2
```

인수를 Integer로 형변환하여 올바른 다중정의 메서드를 선택하게 하면 해결된다. 혹은 Integer.valueOf를 이용해 i를 Integer로 변환한 후 list.remove에 전달해도 된다.

```java
for (int i = 0; i < 3; i++) {
	set.remove(i);
	list.remove((Integer) i); // 혹은 remove(Integer.valueOf(i))
}
```

혼란스러웠던 이유는 List<E> 인터페이스가 remove(Object)와 remove(int)를 다중정의했기 때문이다.

제네릭이 도입되기전에는 Object와 int가 근본적으로 달라서 문제가 없었지만

제네릭과 오토박싱이 등장하면서 두 메서드의 매개변수 타입이 더는 근본적으로 다르지 않게 되었다.

제네릭과 오토박싱의 등장 → List 인터페이스의 취약점 발생

---

### Other Case

```java
// 1번 . Th read의 생성자 호출
new Thread (System.out::println) .start();
// 2번. ExecutorService으I submit 메서드 호출
ExecutorService exec = Executors.newCachedThreadPool();
exec.submit(System.out::println);
```

이렇게 하면 1번은 정상작동이 되지만 2번은 정상 작동 되지 않는 오류가 발생

```java
<T> Future<T> submit(Callable<T> task);
```

하지만 모든 println 은 void 반환 → 다중정의 해소 (”적절한 다중정의 메소드를 찾는 알고리즘”) 은 이렇게 동작하지 않는다.

암시적 타입 람다식이나 부정확한 메서드 참조가 같은 인수 표현식은 목표 타입이 선택되기 전에는 그 의미가 정해지지 않기 때문에 적용성 테스트때 무시된다.

메서드를 다중정의할 때, 서로 다른 함수형 인터페이스라도 같은 위치의 인수로 받아서는 안된다.

### 인수를 포워드하여 두 메서드가 동일한 일을 하도록 보장하기

```java
public boolean contentEquals(StringBuffer sb) {
return contentEquals((CharSequence) sb);
}
```

StringBuffer로 들어와도 charSequence로 형 변환하여 비교하므로 다른 타입이 들어와도 문제가 되지 않는다.

---

## 핵심정리

프로그래밍 언어가 다중정의를 허용한다고 해서 다중정의를 꼭 활용하란 뜻은 아니다.

매개변수 수가 같다면?  → 다중정의를 피하는게 좋다.

다른 방법을 사용하여 사용자에게 fit 하게 바꿔줘야한다.