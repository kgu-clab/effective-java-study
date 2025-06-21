## item10. equals 추이성 위반으로 인한 무한재귀 예제

```java
class Point {
    protected final int x, y;

    public Point(int x, int y) {
        this.x = x;
        this.y = y;
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof Point)) return false;
        Point p = (Point) o;
        return p.x == x && p.y == y;
    }
}
```

```java
class ColoredPoint extends Point {
    private final String color;

    public ColoredPoint(int x, int y, String color) {
        super(x, y);
        this.color = color;
    }

    @Override
    public boolean equals(Object o) {
        if (!(o instanceof Point)) return false;
        if (!(o instanceof ColoredPoint))
            return o.equals(this);

        ColoredPoint cp = (ColoredPoint) o;
        return super.equals(cp) && color.equals(cp.color);
    }
	
}
```

```java
class SmellPoint extends ColoredPoint {
    private final String smell;

    public SmellPoint(int x, int y, String color, String smell) {
        super(x, y, color);
        this.smell = smell;
    }

    @Override
    public boolean equals(Object o) {
        if (!(o instanceof Point)) return false;
        if (!(o instanceof SmellPoint))
            return o.equals(this); // ⚠️ 무한 재귀 발생 가능

        SmellPoint sp = (SmellPoint) o;
        return super.equals(sp) && smell.equals(sp.smell);
    }
}
```

```java
public class Main {
    public static void main(String[] args) {
        ColoredPoint cp = new ColoredPoint(1, 2, "red");
        SmellPoint sp = new SmellPoint(1, 2, "red", "rose");

        System.out.println(cp.equals(sp)); // StackOverflowError 발생
    }
}
```
---
### 무한 재귀 발생 흐름
1. cp.equals(sp) 호출 
2. sp는 ColoredPoint가 아니므로 → sp.equals(cp) 호출 
3. cp는 SmellPoint가 아니므로 → 다시 cp.equals(sp)
4. 위 과정이 무한 반복 
5. StackOverflowError 발생
---
### StackOverflowError가 터지는 이유
JVM은 함수 호출 시 마다 콜스택(stack frame) 을 하나씩 쌓는다. 재귀가 끝없이 호출되면 이 스택이 꽉 차게 되고, JVM은 더 이상 함수 호출을 담을 공간이 없어져서 StackOverflowError 예외를 발생시킴



