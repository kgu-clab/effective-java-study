# item 16 : public 클래스에서는 public 필드가 아닌 접근자 메서드를 사용하라


```java
class Point{
    public double x;
    public double y;
}
```
- 데이터 필드에 직접 접근할 수 있음
  - 캡슐화 이점이 없음
  - API 수정이 없으면 내부 표현을 바꿀 수 없음
  - 불변식이 보장되지 않음
  - 외부 접근 시 부수 작업을 수행할 수 없음

### 필드를 모두 private으로 바꾸고 접근자(getter, setter 메소드)를 추가
```java
class Point {
    private double x;
    private double y;
    
    public Point(double x, double y) {
    this.x = x;
    this.y = y;
    }
    
    public double getX() {return x;}
    public double getY() {return y;}
    
    public void setX(double x) {this.x = x;}
    public void setY(double y) {this.y = y;}
}
```
패키지 외부에서 접근 가능한 클래스는 접근자 제공으로 인해 클래스 내부 표현 방식을 언제든 바꿀 수 있는 유연성을 얻는다.

public 클래스가 필드를 공개하면 이를 사용하는 클라이언트가 생겨남으로써 내부 표현 방식을 마음대로 바꿀 수 없게 된다.

package-private(default)클래스 혹은 private 중첩 클래스라면 데이터 필드를 노출해도 상관없다.

→ 접근자 방식보다 훨씬 깔끔함!

### private 중첩 클래스
```java
public class Calculator {
   
   // private 중첩 클래스 - 필드를 public으로 노출
   private static class Result {
       public double value;    // public 필드
       public boolean isValid; // public 필드
       
       Result(double value, boolean isValid) {
           this.value = value;
           this.isValid = isValid;
       }
   }
   
   public void divide(double a, double b) {
       Result result = new Result(a / b, b != 0);
       
       // 직접 필드 접근
       if (result.isValid) {
           System.out.println("결과: " + result.value);
       }
   }
}
```

### 필드를 노출하지 않는 경우
```java
class Point {
    private double x, y;
    
    Point(double x, double y) {
        this.x = x;
        this.y = y;
    }
    
    public double getX() { return x; }
    public double getY() { return y; }
    public void setX(double x) { this.x = x; }
    public void setY(double y) { this.y = y; }
}

public class Graphics {
    public double distance(Point p1, Point p2) {
        return Math.sqrt((p1.getX() - p2.getX()) * (p1.getX() - p2.getX()) + 
                        (p1.getY() - p2.getY()) * (p1.getY() - p2.getY()));
    }
}
```

### 필드를 노출하는 경우가 더 좋을때도 있다. (가독성과 단순성 측면)
```java
class Point {
    public final double x, y;  // 필드 직접 노출 (final로 불변보장)

    Point(double x, double y) {
        this.x = x;
        this.y = y;
    }
}

public class Graphics {
    public double distance(Point p1, Point p2) {
        return Math.sqrt((p1.x - p2.x) * (p1.x - p2.x) +
                (p1.y - p2.y) * (p1.y - p2.y));
    }
}
```

# 핵심정리 및 요약
### public 클래스는 '절대' 가변 필드를 직접 노출해서는 안된다.
### 불변 필드라면 노출해도 덜 위험하긴 하지만 그래도 private로 접근을 제어하는게 좋다.
### package-private 클래스나 private 중첩 클래스는 필드를 노출하는 편이 좋을때도 있다.