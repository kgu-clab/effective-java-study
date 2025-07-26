## item23. 태그 달린 클래스보다는 클래스 계층구조를 활용하라

### 잘못된 예시
```java
class Figure {
	enum Shape {RECTANGLE, CIRCLE}
    
	final Shape shape;
	
	double length;
	double width;
	
	double radius;
	
	Figure(double radius) {
		shape = Shape.CIRCLE;
		this.radius = radius;
    }
	
	Figure(double length, double width) {
		shape = Shape.RECTANGLE;
		this.length = length;
		this.width = width;
    }
	
	double area() {
		switch(shape) {
            case RECTANGLE:
				return length * width;
            case CIRCLE:
				return Math.PI * (radius * radius);
            default:
				throw new AssertionError(shape);
		}
    }
	
}
```
- 태그 달린 클래스는 장황하고, 오류를 내기 쉽고, 비효율적이다
- 태그 달린 클래스는 클래스 계층 구조를 어설프게 흉내낸 아류일 뿐이다

### 태그 달린 클래스의 단점들
1. 열거 타입 선언, 태그 필드, switch 문 등 쓸데없는 코드가 많다
2. 여러 구현이 혼합되어 있어서 가독성이 나쁘다
3. 메모리도 많이 사용한다
4. final로 선언하기 위해 의미 없는 필드까지 초기화해야한다
5. 런타임에서 에러가 발생한다
6. 새로운 의미를 추가하려면 모든 switch 문을 찾아 코드를 추가해야한다
7. 마찬가지로 하나라도 빠뜨리면 런타임 에러 발생한다
8. 인스턴스 타입 만으로는 현재 나타내는 의미를 알 수 없다

### 태그 달린 클래스를 클래스 계층구조로 바꾸는 방법
1. 계층구조의 root가 될 추상 클래스를 정의한다
2. 태그 값에 따라 동작이 달라지는 메서드를 루트 클래스의 추상 클래스로 선언한다
3. 동작이 일정한 메서드를 루트 클래스에 일반 메서드로 추가한다
4. 모든 하위 클래스에서 공통으로 사용하는 데이터 필드들도 전부 루트 클래스로 올린다
5. 구체 클래스를 정의한다
6. 각 하위 클래스에는 각자의 의미에 해당하는 데이터 필드를 넣는다

### 계층 구조로 변환한 예
```java
abstract class Figure{
	abstract double area();
}

class Circle extends Figure {
	final double radius;
	
	Circle(double radius) { this.radius = radius; }
    
    @Override double area() { return Math.PI * (radius * radius); }
}

class Rectangle extends Figure {
	final double length;
	final double width;

	Rectangle(double length, double width) {
		this.length = length,
        this.width = width;
	}

	@Override double area() { return length * width; }
}
```

### 계층 구조로 변환한 클래스의 장점
- 쓸데없는 코드가 모두 사라졌다
- 살아남은 필드가 모두 final 이다
- case 문 때문에 런타임 오류가 발생할 일도 없다
- 독립적으로 계층 구조를 확장하고 사용할 수 있다
- 유연성은 물론 컴파일 타임 타입 검사 능력을 높여준다



