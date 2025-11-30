## item40 @Override 애너테이션을 일관되게 사용하라

- @Override는 프로그래머에게 가장 중요한 애너테이션일 것이다. 이것은 메서드 선언에만 달 수 있고, 이 애너테이션이 달렸다는 것은 상위 타입의 메서드를 재정의했다는 것을 의미한다.
- @Override 애너테이션을 일관되게 사용하면 여러 버그들을 예방해준다. 다음의 Bigram 프로그램을 살펴보자.(바이그램, 여기서는 영어 알파벳 2개로 구성된 문자열을 표현)

```java
public class Bigram {
  private final char first;
  private final char second;

  public Bigram(char first, char second) {
    this.first = first;
    this.second = second;
  }
  public boolean equals(Bigram b) {
    return b.first == first && b.second = second;
  }
  public int hashCode() {
    return 31 * first + second;
  }

  public static void main(String[] args) {
    Set<Bigram> s = new HashSet<>();

    for(int i = 0; i < 10; i++)
      for(char ch = 'a'; ch <= 'z'; ch++)
        s.add(new Bigram(ch,ch));
    System.out.println(s.size());
  }
}
```

- 위의 코드는 a부터 z까지 26개의 바이그램을 10번 반복해 집합에 추가한 다음, 집합의 크기를 출력하려는 것으로 보인다. Set은 중복을 허용하지 않기 때문에 결과는 26으로 나올길 원했던 것 같다. 하지만 실제로는 260이 출력된다. 그 원인을 하나 하나 살펴보자.

  1. 작성자는 equals 메서드를 재정의하려 한 것으로 보이고 hashCode도 함께 재정의해야 한다는 사실을 잊지 않았다. 하지만 equals는 재정의가 아니라 _다중정의_ 해버렸다.
  2. Object의 equals를 재정의하려면 매개변수 타입을 Object로 해야했는데 그렇게 하지 않았다. 그래서 Objcet에서 상속한 equals와는 별개인 equals를 새로 정의한 꼴이 됐다.
  3. Object의 equals는 ==연산자와 똑같이 객체 식별성만을 확인한다. 따럿 같은 소문자를 소유한 바이그램 10개의 각각이 서로 다른 객체로 인식되고 결과가 260이 된 것이다.

  이제 이를 해결하기 위해 equals 메서드에 @Override를 붙이면

  ```java
  Bigram.java:10: method does not override or implement a method from a supertype
    @Override public boolean equals(Bigram b)
  ```

  꼴의 컴파일 오류가 발생한다. 앞서 말했듯 매개변수 타입이 Object 타입이어야 하는데 Bigram이므로 오류가 발생한 것이다. 이를 올바르게 수정하면 아래와 같은 형태이다.

  ```java
  @Override
  public boolean equals(Object o) {
    if(!(o instanceof Bigram)) return false;
    Bigram b = (Bigram) o;
    return b.first == first && b.second == second;
  }
  ```

- 중요!

  _상위 클래스의 메서드를 재정의하려는 모든 메서드에 @Override 애너테이션을 달자!_

  예외는 한 가지 뿐이다. 구체 클래스에서 상위 클래스의 추상 메서드를 재정의할 때는 굳이 @Override를 달지 않아도 된다. 물론 재정의 메서드 모두에 @Override를 일괄로 붙여두는게 좋아 보인다면 그래도 상관없다.

- 한편, IDE는 @Override를 일관되게 사용하도록 부추기기도 한다. @Override가 달려있지 않은 메서드가 실제로는 재정의를 했다면 경고를 준다. 또 @Override는 클래스뿐 아니라 인터페이스의 메서드를 재정의할 때도 사용할 수 있다.디폴트 메서드를 지원하기 시작하면서, 인터페이스의 메서드를 구현한 메서드에도 @Override를 다는 습관을 들이면 시그니처가 올바른지 재차 확신할 수 있다.
