예전에는 함수 타입을 표현할 떄 추상 메서드를 하나만 담은 인터페이스나 클래스를 사용했다.

이때 이런 인터페이스의 인스턴스를 '함수 객체'라고 부른다.

JDK 1.1 이후에는 익명 클래스를 사용해서 함수 객체를 만들었다.

아래 함수를 익명 클래스를 활용한 정렬 함수의 예이다.
```java
Collections.sort(words, new Comparator<String>() {
	public int compare(String s1, String s2) {
		return Integer.compare(s1.length(), s2.length());
	}
})；
```

하지만 코드가 너무 길다.

### 람다식

자바8에 오면서 위의 함수 객체는 람다식을 활용해서 만들 수 있게 되었다.

람다는 익명 클래스와 비슷한 기능을 수행하지만 훨씬 명확하고 간결하게 표현할 수 있다.

```java
Collections.sort(words,
	(s1, s2) -> Integer.compare(s1.length(), s2.length()));
```

여기서 람다식이나 s1,s2의 타입은 따로 명세해주지 않아도 알아서 타입 추론을 해준다.

코드의 간결함을 위해 타입을 더 명확히 명세해줄 필요가 있거나 컴파일러가 오류를 뱉을때를 제외하면 타입 명세를 생략하는 것이 좋다.

> [!info] 
> 컴파일러는 타입을 추론하기 위한 대부분의 정보를 제네릭에서 가져온다. 따라서 람다를 사용해야 한다면 더욱이 로 타입을 지양하고 제네릭을 활용해야 한다.


위의 코드에서 비교자 생성 메서드를 활용하면 코드를 줄일 수 있으며, List 인터페이스의 sort 메서드를 활용하면 더 줄어들 수 있다.
```java
Collections.sort(words, comparinglnt(String::length));

words.sort(comparinglnt(String::length));
```

### 활용

아이템 34에서는 Operation 열거 타입에서 각 상수별 apply 메서드를 재정의했다.
```java
public enum Operation {
	PLUS("+") {
		public double apply(double x, double y) { return x + y; }
	},
	MINUS("-") {
		public double apply(double x, double y) { return x - y; }
	},
	TIMES("*") {
		public double apply(double x, double y) { return x * y; }
	},
	DIVIDE("/") {
		public double apply(double x, double y) { return x / y; }
	}
	
	private final String symbol;
	
	Operation(String symbol) { this.symbol = symbol; }
	
	@Override public String toString() { return symbol; }
	public abstract double apply(double x, double y);
}
```
- 이렇게 구현하는 방식보다는 열거 타입에 인스턴스 필드를 두는 방식이 좋다.
- 이때 람다를 활용하면 아래와 같은 형태로 구현할 수 있다.


```java
public enum Operation {
	PLUS ("+",(x, y) —> x + y),
	MINUS ("-",(x, y) -> x - y),
	TIMES ("*", (x, y) —> x * y),
	DIVIDE("/", (x, y) -> x / y);
	
	private final String symbol;
	private final DoubleBinaryOperator op;
	
	Operation(String symbol, DoubleBinaryOperator op) {
		this.symbol = symbol;
		this.op = op;
	}
	
	@Override public String toString() { return symbol; }

	public double apply(double x, double y) {
		return op.applyAsDouble(x, y);
	}
}
```
- 각 상수별 동작을 람다로 구현해 생성자에 넘기면, 생성자는 이 람다를 인스턴스 필드에 저장해둔다.
> [!info]
> DoubleBinaryOperator 인터페이스는 함수 인터페이스 중 하나로, 저장된 람다를 double로 실행할 수 있게 한다.


### 주의점

- 코드 자체로 로직이 설명되지 않거나 코드가 길어진다면 쓰면 안된다.
	- 람다는 메서드나 클래스와는 달리 이름이 없으며, 문서화도 할 수 없기 때문
	- 만약 3줄을 넘어간다면 더 줄이거나 람다를 쓰지 않는것이 좋다.
- 람다는 인스턴스 필드나 메서드를 사용하지 못한다.
- 추상 클래스의 인스턴스를 만들어야 하거나, 추상 메서드가 여러개인 인터페이스의 인스턴스를 만들 때는 익명 클래스를 사용해야 한다.
- 만약 함수 객체가 자기 자신을 참조해야 한다면 익명 클래스를 활용해야 한다.
	- 람다는 자기 자신이 아닌 바깥 인스턴스를 가리키기 때문
- 람다(그리고 익명 클래스)는 직렬화 방식이 VM별로 다르기 때문에 절대 삼가야 한다.
	- 만약 직렬화가 필요하다면 private 정적 중첩 클래스를 사용해야 한다.


### 정리
> 람다를 활용하면 작은 함수 객체를 간결하게 구현할 수 있다.
> 다만 항상 람다를 남용해서는 안되며, 상황에 알맞게 람다와 익명 클래스를 활용하자.