### 비트 필드 대신 EnumSet을 사용하라

- 정수 열거 상수(비트 필드 열거 상수)
  열거한 값들이 주로 집합으로 사용될 때, 예전에는 각 상수에 서로 다른 2의 거듭제곱 값을 할당한 **정수 열거 패턴**을 사용해 옴
  →예시코드

    ```java
    public class Text {
    public static finalint STYLE_BOLD = 1<<0; // 1
    public static finalint STYLE_ITALIC = 1<<1; // 2
    public static finalint STYLE_UNDERLINE = 1<< 2; // 4
    public static finalint STYLE_STRIKETHROUGH = 1<<3; // 8
    // 매개변수 styles는 0개 이상의 STYLE, 상수률 비트별 OR한 값이다.
    public void applyStylesint styles () {}
    }
    ```


- 비트필드

    ```java
    text.applyStyles(STYLE_BOLD | STYLE_ITALIC):
    ```

  다음과 같이 비트별 OR를 사용해 여러 상수를 하나의 집합으로 모으게 될 시 만들어지는 집합을 **비트 필드**라고 함

  장점: 비트 필드 사용시, 비트별 연산을 사용해 합집합과 교집합 같은 집합 연산을 효율적으로 수행할 수 있음
  단점1: 비트 값이 그대로 출력되면 단순한 정수 열거 상수를 출력할 때보다 해석하기가 더 어려움
  단점2: 비트 필드 하나에 녹아 있는 모든 원소를 순회하기 까다로움
  단점3: 최대 몇 비트가 필요한지를 API 작성 시 미리 예측하여 적절한 타입을 미리 선택해야 함
  →API를 수정하지 않고는 비트 수를 더 늘릴 수 없기 때문

- java.util 패키지의 EnumSet 클래스
  비트 필드의 더 나은 대안으로서 열거 타입 상수의 값으로 구성된 집합을 효과적으로 표현해 줌

장점1: Set 인터페이스를 완벽히 구현함
장점2: 타입 안전함
장점3: 다른 어떤 Set 구현체와도 함께 사용 가능함
장점4: EnumSet의 내부는 비트 벡터로 구현되어 있어 비트 필드에 비견되는 성능을 보여줌
→removeAll과 retainAll 같은 대량 작업은 비트를 효율적으로 처리할 수 있는 산술 연산을 써서 구현됨
장점5: 비트를 직저 다룰 때 흔히 겪을 수 있는 난해한 작업을 EnumSet이 다 처리해줌

- 수정 코드:

    ```java
    public class Text {
    public enum Style { BOLD, ITALIC, UNDERLINE, STRIKETHROUGH }
    // 어떤 Set을 넘겨도 되나, EnumSet이 가장 쫗다.
    public void applyStyles(Set<Style> styles) { }
    }
    ```

  추가 코드:

    ```java
    text.applyStyles(EnumSet.of(Style.BOLD, Style.ITALIC));
    ```

  →추가 코드는 수정 코드의 appleStyles 메서드에 EnumSet 인스턴스를 건네는 클라이언트 코드
  →EnumSet은 집합 생성 등 다양한 기능의 정적 팩터리를 제공하며, 그 중 of 메서드를 사용함
  →appleStyles 메서드가 EnumSet<Style>이 아닌 Set<Style>을 받은 이유는 ""인터페이스로 받는게 일반적으로 좋은 습관""이며, ""다른 Set 구현체를 넘기더라도 처리할 수 있게 됨""
  (즉, 클라이언트는 Set 구현을 통해 그 자리에서 EnumSet이나 HashSet 등 원하는 구현체를 넣어 호출 가능하다)