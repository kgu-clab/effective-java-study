### item53 가변인수는 신중히 사용하라.

- 가변인수 메서드는 명시한 타입의 인수를 0개 이상 받을 수 있다.
- 가변인수 호출 시 동작 순서
  1. 인수의 개수와 길이가 같은 배열을 만든다.
  2. 인수들을 이 배열에 저장하여 가변인수 메서드에 건네준다.
  - 다음은 가변인수를 사용한 예시이다.
  ```java
  static int sum(int... args) {
    int sum = 0;
    for(int arg: args) sum+= arg;
    return sum;
  }
  ```
  위의 코드는 sum(1,2,3)을 하면 6을 sum()을 하면 0을 반환한다. 하지만 인수가 1개 이상이어야 할 때도 존재한다.
  ```java
  static int min(int... args) {
    if(args.length == 0)
      throw new IllegalArgumentException("인수가 1개 이상 필요합니다.");
    int min = args[0];
    for(int i = 1; i < args.length; i++)
      if(args[i] < min)
        min = args[i];
    return min;
  }
  ```
  코드를 보면 알 수 있듯이 매개변수로 받은 args에서 가장 작은 값을 반환하는 코드이다. 이 방식에는 몇 가지 문제점이 존재한다.
- 일단 기본적으로 코드가 지저분한다. min의 초기값을 코드에 항상 명시를 해놔야한다. 이건 가독성의 문제이기에 크게 문제가 안 될 수 있다.
- 더 큰 문제는 런타임을 실패한다는 것이다. 그 이유는 가변인수의 인수 개수는 런타임에 배열의 길이를 알 수 있는데 사용자가 인수를 0개만 넣어 호출을 하면 배열의 길이가 없음을 컴파일타임이 아닌 런타임에 깨닫게 되고 실패를 한다는 것이다.
- 이를 해결하기 위해선 매개변수를 2개 받도록 하면 된다. 즉, 첫 번째로는 평범한 매개변수를 받고, 가변인수는 두 번째로 받으면 앞의 문제를 해결할 수 있다.

  ```java
  static int min(int firstArg, int... remainingArgs) {
      int min = firstArg;
      for(int arg: remainingArgs)
        if(arg < min)
          min = arg;
      return min;
    }
  ```

  이렇듯 가변인수는 인수 개수가 정해지지 않았을 때 아주 유용하다. 이쯤에서 드는 생각이 그럼 언제 가변인수를 쓸까?

  답은 printf이다. printf는 가변인수와 한 묶음으로 자바에 도입되었고, 이때 핵심 리플렉션 기능(아이템65)도 재정비되었다. printf와 리플렉션 모두 가변인수 덕을 본 것이다.

- 그런데 성능이 민감한 상황이면 가변인수가 문제가 될 수 있다. 가변인수 메서드는 호출될 때마다 배열을 새로 하나 할당하고 초기화한다. 다행히, 이 비용을 감당할 수는 없지만 가변인수의 유연성이 필요할 때 선택할 수 있는 패턴이 있다.
- 예를 들면, 메서드 호출의 95%가 인수를 3개 이하로 사용한다고 할 때, 인수가 0개인 것부터 4개인 것까지 총 5개를 다중정의한다. 마지막 다중정의 메서드가 인수 4개 이상인 5%의 호출을 담당하는 것이다.
  ```java
  public void foo() {}
  public void foo(int a1) {}
  public void foo(int a1, int a2) {}
  public void foo(int a1, int a2, int a3) {}
  public void foo(int a1, int a2, int a3, int... rest) {}
  ```
  이러한 방식은 대표적으로 EnumSet의 정적 팩터리에서 열거 타입 집합 생성 비용을 최소로 할 때 사용하고 있다.
