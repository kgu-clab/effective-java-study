## item22 인터페이스는 타입을 정의하는 용도로만 사용하라

- 인터페이스의 역할: 클래스가 어떤 인터페이스를 구현한다는 것은 자신의 인스턴스로 무엇을 할 수 있는지를 클라이언트에 얘기해주는 것이다. (인터페이스는 오직 이 용도로만 사용해야 한다.)
- 이 지침을 어긴 케이스: 상수 인터페이스
- 상수 인터페이스: 메서드 없이, 상수를 뜻하는 static final 필드로만 가득 찬 인터페이스

```java
  public interface Constants {
    static final int WIDTH = 800;
    static final int HEIGHT = 600;
    static final String TITLE = "My App";
  }
  //상수 인터페이스를 상속해서 사용
  public class MyWindow implements Constants {
    public void print() {
        System.out.println("창 너비: " + WIDTH);
    }
  }
```

- 클래스 내부에서 사용하는 상수는 내부 구현에 해당된다. 즉, 상수 인터페이스를 구현하는 것은 이 내부 구현을 클래스의 API로 노출하는 행위다.
- 그 이유는 사용자의 입장에서 위의 코드는 아래의 코드처럼 보인다.

  ```java
  MyWindow window = new MyWindow();
  window instanceof Constants //true!
  ```

  즉,외부에서는 *이 클래스가 Constants라는 타입을 구현하고 있다*고 인식하게 된다.

- 상수 인터페이스는 사용자에게 아무런 의미가 없고 오힐 사용자에게 혼란을 주기도 하며, 더 심하게는 클라이언트 코드가 내부 구현에 해당하는 이 상수들에 종속되게 한다.
- 예를 들어,

  ```java
  if (userInput.equals(Constants.TITLE)) {
    ...
  }
  ```

  이런 코드를 사용하다가 Constants.TITLE이 더 이상 필요가 없어져서 지우고 싶어도 이걸 사용하는 모든 사용자의 코드가 깨지거나 이런 오류를 방지하기 위해 억지로 인스페이스를 계속 유지시켜야 한다.

---

- 그러면 이런 상수들의 묶음은 어떻게 표시해야 좋을까?

1. 특정 클래스나 인터페이스와 강하게 연관된 상수라면 그 클래스나 인터페이스 자체에 추가해야 한다. 대펴적으로 Integer와 Double에 선언된 MAX/MIN_VALUE가 있다.
2. 열거 타입으로 나타내기 적합한 상수라면 열거 타입으로 만들어 공개하면 된다.
3. 인스턴스화할 수 없는 유틸리티 클래스에 담아서 공개하면 된다.
   위의 예시를 유틸리티 클래스를 바꾸어 보면 아래와 같은 코드가 된다.

```java
public class Constants {
  private Constants() {} //인스턴스화 방지

  public static final int WIDTH = 800;
  public static final int HEIGHT = 600;
  public static final String TITLE = "My App";
}
```

- 유틸리티 클래스에 정의된 상수를 클라이언트에서 사용하려면 클래스 이름까지 함께 명시해야 한다. (_Constants.WIDTH_) -> 정적 임포트하여 클래스 이름은 생략할 수도 있다.

---

- 핵심 정리
  - _인터페이스는 타입을 정의하는 용도로만 사용해야 한다. 상수 공개용 수단으로 사용하지 말자._
