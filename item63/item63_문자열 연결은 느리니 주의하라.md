### item63 문자열 연결은 느리니 주의하라

- 문자열 연결 연산자(+)는 문자열을 하나로 합쳐주는 편리한 수단이다. 그런데 크기가 작은 문자열 표현을 만들 때는 괜찮지만, 본격적으로 사용하기 시작하면 성능 저하를 감내하기 어렵다.
- `문자열 연결 연산자로 문자열 n개를 잇는 시간은 n^2에 비례한다.` -> 문자열은 불변이라서 두 문자열을 연결할 경우 양쪽의 내용을 모두 복사해야 하기 때문이다.

  - 예를 들면

  ```java
  public String statement() {
    String result = "";
    for (int i = 0; i < numItems(); i++) {
      result += lineForItem(i);
    }
    return result;
  }
  ```

  위의 코드는 result 변수에 아이템의 내용들을 하나하나 이어붙이는 코드이다. 앞서 말했듯이 아이템의 품목이 많지 않다면 큰 문제가 되지 않지만, 품목이 많을 경우 이 메서드는 심각하게 느려질 수 있다.

  - 성능을 포기하고 싶지 않다면 String 대신 StringBuilder를 사용하자.

  ```java
  public String statement2() {
    StringBuilder b = new StringBuilder(numItems() * LINW_WIDTH);
    for (int i = 0; i < numItems(); i++) {
      b.append(lineForItem(i));
    }

    return b.toString();
  }
  ```

  statement2가 약 6.5배 빠르다는 결과를 받을 수 있다. statement 메서드의 수행 시간은 품목 수의 제곱이 비례해 늘어나고 statement2는 선형으로 늘어나므로, 품목 수가 늘어날수록 성능 격차는 점점 벌어질 것이다.

- 핵심 정리

  -> `많은 문자열을 연결할 때는 문자열 연결 연산자(+)는 피하자.`
