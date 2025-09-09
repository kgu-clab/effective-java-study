# 명명 패턴보다 애너테이션을 사용하라

### 명명 패턴의 단점
1. 오타가 나면 안된다.
    - Junit은 이 메서드를 무시하고 지나침
2. Junit이 명확하게 개발자 의도를 알아채진 못한다.
3. 프로그램 요소를 매개변수로 전달할 방법이 없다.
    - 예외 타입을 테스트에 매개변수로 전달해야할 때, 컴파일러는 알아차릴 수 없다.

### 애너테이션은 이 모든 문제를 해결한다.

```java
import java.lang.annotation.*;

@Retention(RetentionPolicy.RUNTIME)
//@Test가 런타임에도 유지되어야함
@Target(ElementTyep.METHOD)
//@Test가 반드시 메서드 선언에만 사용되어야 함
public @interface test{
}
```
메타애너테이션 : @Retention, @Target같은 애니테이션의 애너테이션

### @Test 애너테이션 적용
```java
public class Sample {
    @Test public static void m1() { }
    public static void m2() { }
    @Test public static void m3() {
        throw new RuntimeException("실패");
    }
    public static void m4() { }
    @Test public void m5() { }
    public static void m6() { }
    @Test public static void m7() {
        thorw new RuntimeException("실패");
    }
    public static void m8() { }
}
```
Test 메서드 4개중 1개는 성공, 2개는 실패, 1개는 잘못 사용
나머지 메서드는 테스트 도구가 무시할 것이다.

## 1. 마커 애너테이션 처리 프로그램
```java
import java.lang.reflect.*;

public class RunTests {
   public static void main(String[] args) throws Exception {
       int tests = 0;
       int passed = 0;
       Class<?> testClass = Class.forName(args[0]);
       for (Method m : testClass.getDeclaredMethods()) {
           if (m.isAnnotationPresent(Test.class)) {
               tests++;
               try {
                   m.invoke(null);
                   passed++;
               } catch (InvocationTargetException wrappedExc) {
                   Throwable exc = wrappedExc.getCause();
                   System.out.println(m + " 실패: " + exc);
               } catch (Exception exc) {
                   System.out.println("잘못 사용한 @Test: " + m);
               }
           }
       }
       System.out.printf("성공: %d, 실패: %d%n",
               passed, tests - passed);
   }
}
}
```

- @Test 애너테이션 메서드를 차례로 호출
- InvocationTargetException 외의 예외는 @Test 애너테이션을 잘못 사용한 것.
- 해당 프로그램으로 Sample 실행시 = 성공 1, 실패 3

## 2. 명시한 예외를 던져야하는 테스트 메서드용 애너테이션
```java
import java.lang.annotation.*;

@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface ExceptionTest{
    Class<? extends Throwable> value();
    //Throwable을 확장한 클래스의 class 객체
}
```
class 리터럴은 애너테이션 매개변수의 값으로 사용됨

```java
public class Sample2 {
    @ExceptionTest(ArithmeticException.class)
    public static void m1(){ //성공
        int i=0;
        i = i/i;
    }
    @ExceptionTest(ArithmeticException.class)
    public static void m2(){ //다른 예외 발생
        int[] a = new int[0];
        int i = a[1];
    }
    @ExceptionTest(ArithmeticException.class)
    public static void m3(){ } //실패
}
```
### 이 애너테이션을 활용하기 위해 테스트 도구를 수정.

```java
if (m.isAnnotationPresent(ExceptionTest.class)) {
tests++;
        try {
        m.invoke(null);
       System.out.printf("테스트 %s 실패: 예외를 던지지 않음%n", m);
   } catch (InvocationTargetException wrappedEx) {
Throwable exc = wrappedEx.getCause();
Class<? extends Throwable> excType =
        m.getAnnotation(ExceptionTest.class).value();
       if (excType.isInstance(exc)) {
passed++;
        } else {
        System.out.printf(
               "테스트 %s 실패: 기대한 예외 %s, 발생한 예외 %s%n",
               m, excType.getName(), exc.getClass().getName());
        }
        } catch (Exception exc) {
        System.out.println("잘못 사용한 @ExceptionTest: " + m);
   }
           }
```
이 코드는 애너테이션 매개변수의 값을 추출해 테스트 메서드가 올바른 예외를 던지는지 확인.

- 문제없이 컴파일 => 애너테이션 매개변수가 올바른 타입
- 컴파일에는 존재했으나 런타임에는 존재하지 않음 => TypeNotPresentException을 던짐

## 3. 여러개의 예외, 그중 하나가 발생하면 성공하게하는 코드
```java
import java.lang.annotation.*;

@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface ExceptionTest{
    Class<? extends Throwable>[] value();
    // 매개변수를 객체 배열로 수정
}
```
테스트 러너 또한 수정
```java
if (m.isAnnotationPresent(ExceptionTest.class)) {
   tests++;
   try {
       m.invoke(null);
       System.out.printf("테스트 %s 실패: 예외를 던지지 않음%n", m);
   } catch (Throwable wrappedExc) {
       Throwable exc = wrappedExc.getCause();
       int oldPassed = passed;
       Class<? extends Throwable>[] excTypes = 
           m.getAnnotation(ExceptionTest.class).value();
       for (Class<? extends Throwable> excType : excTypes) {
           if (excType.isInstance(exc)) {
               passed++;
               break;
           }
       }
       if (passed == oldPassed)
           System.out.printf("테스트 %s 실패: %s %n", m, exc);
   }
}
```



# 핵심정리? (미흡하게 정리하여 죄송합니다ㅠ)
- 명명패턴보다는 애너테이션을 활용하자
    - 애너테이션으로 할 수 있는 일을 명명 패턴으로 처리할 이유는 없다.
- 애너테이션을 선언하고 이를 처리하는 부분에서 코드의 양이 늘어나고, 복잡해져 오류 가능성이 커질 수 있다는 것을 명심하자.
- 자바 프로그래머라면 자바가 제공하는 애너테이션을 타입들은 사용할줄 알아야 한다.
