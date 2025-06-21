## item10. equals는 일반 규약을 지켜 재정의하라

- equals 메서드는 재정의하기 쉬워 보이지만 곳곳에 함정이 도사리고 있다
---
- 다음 상황 중 하나에 해당한다면 재정의하지 말아야 한다
1. 각 인스턴스가 본질적으로 고유함
   - 예) Thread, InputStream, OutputStream 처럼 동작하는 개체
   - Thread는 각각 고유한 실행 흐름을 가지므로 equals() 재정의 의미 없음
```java
public static void main(String[] args) {
	Thread t1 = new Thread();
	Thread t2 = new Thread();

	System.out.println(t1.equals(t2));
}
```
2. 인스턴스의 논리적 동치성을 검사할 일이 없는 경우
   - 논리적 동치성이란, 두 객체가 의미상으로 같은지를 판단하는 것
   - Object의 equals가 참조값을 비교하기 때문에, Object.equals()를 그대로 사용해도 충분하다
3. 상위 클래스에서 재정의한 equals가 하위 클래스에서도 딱 들어맞는경우
4. 클래스가 private 이거나 package-private이고, equals를 호출할 일이 없는 경우

---
- equals를 재정의 해야할 때는 언제일까?
- 주로 Integer, String 같은 값 클래스가 해당된다. 값을 비교해야하기 때문
```java
public static void main(String[] args) {
	String s1 = new String("hello");
	String s2 = new String("hello");

	System.out.println(s1 == s2);        // false (참조 비교)
	System.out.println(s1.equals(s2));   // true  (값 비교)
}
```
---
### equals를 재정의할 때는 다음 규약을 지켜야 한다
1. 반사성
   - 객체는 자기 자신과 같아야 한다
   - 사실 어기기가 더 어렵다
2. 대칭성
   - 서로에 대한 동치 여부에 똑같이 답해야 한다
   - 아래 예시에서 CaseInsensitiveString은 일반 String을 알지만 String은 CaseInsensitiveString을 알지 못한다
```java
public final class CaseInsensitiveString {
    private final String s;

    public CaseInsensitiveString(String s) {
        this.s = Object.requireNonNull(s);
    }

    @Override
    public boolean equals(Object o) {
        if (o instanceof CaseInsensitiveString) {
            return s.equalsIgnoreCase(((CaseInsensitiveString) o).s);
        }
        if (o instanceof String) { // 일반 문자열과도 비교를 시도함
            return s.equalsIgnoreCase((String) o);
        }
        return false;
    }
}

public static void main(String[] args) {
    CaseInsensitiveString cis = new CaseInsensitiveString("hello");
    String str = "HELLO";

    System.out.println(cis.equals(str)); // true
    System.out.println(str.equals(cis)); // false (대칭성 위반)
}
```
3. 추이성
   - 첫번째 객체와 두번째 객체가 같고, 두번째 객체와 세번째 객체가 같다면, 첫번째와 세번째 객체도 같아야 한다
   - 구체 클래스를 확장해 새로운 값을 추가하면서 equals 규약을 만족시키는 방법은 없다
   - 상속 대신 컴포지션을 사용하면 된다 (아이템 18 참고)
```java
class Point {
    final int x, y;
    Point(int x, int y) {
        this.x = x; this.y = y;
    }

    @Override
    public boolean equals(Object o) {
        if (!(o instanceof Point)) return false;
        Point p = (Point) o;
        return x == p.x && y == p.y;
    }
}

class ColoredPoint extends Point {
    final String color;

    ColoredPoint(int x, int y, String color) {
        super(x, y);
        this.color = color;
    }

    @Override
    public boolean equals(Object o) {
        if (!(o instanceof Point)) return false;
        if (!(o instanceof ColoredPoint)) return o.equals(this); 

        ColoredPoint cp = (ColoredPoint) o;
        return super.equals(cp) && color.equals(cp.color);
    }
}

public static void main(String[] args) {
	Point p = new Point(1, 2);
	ColoredPoint cp1 = new ColoredPoint(1, 2, "red");
	ColoredPoint cp2 = new ColoredPoint(1, 2, "blue");

	// 1. p와 cp1은 x, y만 비교 → true
	System.out.println(p.equals(cp1)); // true

	// 2. p와 cp2도 x, y만 비교 → true
	System.out.println(p.equals(cp2)); // true

	// 3. cp1과 cp2는 color까지 비교 → false
	System.out.println(cp1.equals(cp2)); // false 

}
```
4. 일관성
   - 두 객체가 같다면 영원히 같아야 한다
   - 클래스가 불변이든 가변이든 equals의 판단에 신뢰할 수 없는 자원이 끼어들어서는 안됨
```java
class MutableStringWrapper {
    private String value;

    public MutableStringWrapper(String value) {
        this.value = value;
    }

    public void setValue(String v) {
        this.value = v;
    }

    @Override
    public boolean equals(Object o) {
        if (!(o instanceof MutableStringWrapper)) return false;
        return value.equalsIgnoreCase(((MutableStringWrapper) o).value);
    }

    @Override
    public int hashCode() {
        return value.toLowerCase().hashCode();
    }
}
```
5. null-아님
    - NullPointerException을 피하기 위해, equals 메서드는 null과 비교할 때 false를 반환해야 한다
    - 명시적으로 null을 검사할 수도 있지만 instanceof 연산자를 사용하면 null을 안전하게 처리할 수 있다
    - instanceof 연산자는 null을 검사할 때 false를 반환하기 때문이다
```java
@Override
public boolean equals(Object o) {
    if (!(o instanceof MyType)) return false;
    // 나머지 비교 로직
}
```
---
### 지금까지의 내용을 종합한 양질의 equals 메서드 구현 방법
1. == 연산자를 사용해 입력이 자기 자신의 참조인지 확인한다
2. instanceof 연산자를 사용해 입력이 올바른 타입인지 확인한다
3. 입력을 올바른 타입으로 형변환한다
4. 입력 객체와 자기 자신의 대응되는 핵심 필드들이 모두 일치하는지 하나씩 검사한다
---
### 핵심 정리
- 꼭 필요한 경우가 아니면 euals를 재정의하지 말자. 많은 경우에 Object의 equals가 원하는 비교를 정확히 수행해준다. 재정의해야 할 때는 그 클래스의 핵심 필드를 모두 빠짐없이, 다섯가지 규약을 확실히 지켜가며 비교해야 한다.
