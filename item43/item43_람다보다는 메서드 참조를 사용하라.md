### item43 람다보다는 메서드 참조를 사용하라.

- 자바에는 람다보다 함수 객체를 간결하게 만드는 메서드 참조(method reference)라는 방법이 있다.

  이를 설명하기 위해 merge 메서드를 예시로 들어보겠다.

  ```java
  map.merge(key, 1, (count, incr) -> count + incr);
  ```

  위의 예시는 람다를 이용해 merge 메서드를 사용한 것이다. merge 메서드는 키, 값, 함수를 인수로 받으며, 주어진 키가 맵 안에 아직 없다면 주어진 {키, 값} 쌍을 그대로 저장한다. 반대로 키가 이미 있다면 (인수로 받은)함수를 현재 값과 주어진 값에 적용한 다음, 그 결과로 현재 값을 덮어쓴다. 즉, 맵에 {키, 함수의 결과} 쌍을 저장한다.

  위의 식도 충분히 간단하지만 메서드 참조를 쓰면 이것보다 훨씬 간결하게 코드 작성이 가능해진다.

  ```java
  map.merge(key, 1, Integer: :sum);
  ```

  (추가 설명) -> 람다는 기존에 존재하는 메서드를 그대로 호출하는 것이라면, 메서드 참조는 그 메서드를 직접 가리키는 방식이다.

  매개변수 수가 늘어날수록 메서드 참조로 제거한 수 있는 코드양도 늘어난다. 하지만 어떤 람다에서는 매개변수의 이름 자체가 프로그래머에게 좋은 가이드가 되기도 한다.

- 람다로 할 수 없는 일이라면 메서드 참조로도 할 수 있다.(마지막에 보충 설명) 보통 메서드 참조를 사용하는 편이 더 짧고 간결하므로 람다를 대신할 좋은 대안이 되어준다. 즉, 람다로 작성할 코드를 새로운 메서드에 담은 다음, 람다 대신 그 메서드 참조를 사용하는 식이다.

  하지만 언제나 그렇듯 늘 그런 것은 아니다. 때론 람다가 메서드 참조 보다 간결할 때가 있다. 주로 메서드와 람다가 같은 클래스에 있을 때 그렇다.

  ```java
  service.execute(GoshThisClassIsHumangous: :action); //메서드 참조
  service.execute((): :action); //람다식
  ```

- 메서드 참조의 유형

  1. 정적 메서드 참조(위에서 설명한 것)

  ```java
  Function<String, Integer> func = (s) -> Integer.parseInt(s);
  Function<String, Integer> func = Integer: :parseInt;
  ```

  2. 특정 객체의 인스턴스 메서드 참조

  ```java
  String str = "hello";
  Supplier<Integer> sup = () -> str.length();
  Supplier<Integer> sup = str: :length;
  ```

  3. 임의 객체의 인스턴스 메서드 참조

  ```java
  List<String> list = Arrays.asList("a", "b", "c");

  // 람다식
  list.forEach(s -> System.out.println(s));

  // 메서드 참조
  list.forEach(System.out::println);
  ```

  4. 생성자 참조(클래스& 배열)

  ```java
  //생성자 메서드 참조
  Supplier<List<String>> sup = () -> new ArrayList<>();
  Supplier<List<String>> sup = ArrayList::new;

  //배열 메서드 참조
  IntFunction<String[]> lambda = size -> new String[size];
  IntFunction<String[]> methodRef = String[]::new;
  ```

- 보충 설명

  람다로는 불가능하나 메서드 참조로는 가능한 유일한 형태가 하나 있는데 바로 제네릭 함수 타입 구현이다.

  함수형 인터페이스의 추상 메서드가 제네릭일 수 있듯이 함수 타입도 제네릭일 수 있다. 이는 메서드 참조 표현식으로는 구현할 수 있지만, 제네릭 람다식이라는 문법은 존재하지 않기 때문에 람다식으로는 불가능하다.
