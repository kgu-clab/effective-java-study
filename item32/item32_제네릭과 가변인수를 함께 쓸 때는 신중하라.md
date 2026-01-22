## item32 제네릭과 가변인수를 함께 쓸 때는 신중하라.

- 가변인수는 메서드에 넘기는 인수의 개수를 클라이언트가 조절할 수 있지만, 구현 방식에서 허점이 있다.

  -> 가변인수 메서드를 호출하면 가변인수를 담기 위한 배열이 자동으로 하나 만들어지는데, 내부로 감춰야 했을 이 배열이 클라이언트에 노출되는 문제가 생겼다.

  즉, _varargs 매개변수에 제네릭이나 매개변수화 타입이 포함되면 컴파일 경고가 발생한다._

  -> varargs 매개변수는 배열이다. 여기에 제네릭 타입을 넣으면 제네릭 배열(JAVA에서 지원하지 않음)이 된다. 배열과 제네릭의 타입 시스템 차이에 의해 타입 안전성이 붕괴되고, 이로 인해 *힙 오염*이 발생된다.

```java
  static void dangerous(List<String>... stringLists) {
    List<Integer> intList = List.of(42);
    Object[] objects = stringLIsts;
    objects[0] = intList;                     //힙 오염 발생
    String s = stringLists[0].get(0);   //ClassCastException
    //마지막 줄에 컴파일러가 생성한 (보이지 않는) 형변환이 숨어있기 때문이다.
  }
```

- 제네릭 배열을 프로그래머가 직접 생성하는 건 허용하지 않으면서 제네릭 varargs 매개변수를 받는 메서드를 선언할 수 있는 이유가 무엇일까? -> 즉, 왜 오류가 아닌 경고로 끝일까?

      제네릭이나 매개변수화 타입의 varargs 매개변수를 받는 메서드가 실무에서 매우 유용하기 때문이다. 그래서 언어 설계자는 이 모순을 수용하기로 했다. (실제 자바 라이브러리도 이런 메서드를 여럿 제공한다.)

- 자바 7이전에는 @SuppressWarnings("unchecked")로 일일히 경고를 숨겨야 했지만, 자바 7에서 @SafeVarargs 애너테이션(메서드 작성자가 그 메서드가 타입 안정함을 보장하는 장치)이 추가 되어 작성자가 클라이언트 측에서 발생하는 경고를 숨길 수 있게 되었다.
- 메서드가 안전한지는 어떻게 확신할 수 있을까?

  -> 메서드가 이 배열에 아무 것도 저장하지 않고 그 배열의 참조가 밖으로 노출되지 않는다면 타입 안전하다. 바꿔 말해서, 이 varargs 매개변수 배열이 호출자로부터 그 메서드로 순수하게 인수들을 전달하는 일만 한다면 그 메서드는 안전하다.

- 단, 이것이 무조건적으로 옳은 것은 아니다.

  예를 들어,

  ```java
  static <T> T[] pickTwo(T a, T b, T c) {
    switch(ThreadLocalRandom.current().nextInt(3)) {
      case 0: return toArray(a, b);
      case 1: return toArray(b, c);
      case 2: return toArray(c, a);
    }
    throw new AssertionError(); //도달할 수 없다.
  }

  public static void main(String[] args) {
    String[] attributes = pickTwo("좋은","빠른","저렴한");
  }
  ```

  - pickTwo의 반환값을 attributes에 저장하기 위해 String[]로 형변환하는 코드를 컴파일러가 자동 생성한다는 점을 놓쳤다. Object[]는 String[]의 하위 타입이 아니므로 이 형변환은 실패한다.

    _따라서, 제네릭 varargs 매개변수 배열에 다른 메서드가 접근하도록 허용하면 안전하지 않다!_

    - 물론 여기에도 예외 2가지가 있다.

    1. @SafeVarargs로 제대로 애노테이트된 또 다른 varargs 메서드에 넘기는 것은 안전하다.
    2. 그저 이 배열 내용의 일부 함수를 호출만 하는(varags를 받지 않는) 일반 메서드에 넘기는 것도 안전하다.

- @SafeVarargs 애너테이션을 사용할 때 정하는 규칙 : 제네릭이나 매개변수화 타입의 varargs 매개변수를 받는 모든 메서드에 @SafeVarargs를 달아야 한다.(안전하지 않은 varargs 메서드는 절대 작성해서는 안 된다.)

- 정리

  1. varages 매개변수 배열에 아무것도 저장하지 않는다.
  2. 그 배열(혹은 복제본)을 신뢰할 수 없는 코드에 노출하지 않는다.

  ***

  참고

  @SafeVarargs 애너테이션이 유일한 정답이 아니다. varargs 매개변수를 List 매개변수로 바꿀 수도 있다. (아이템 28의 내용)
