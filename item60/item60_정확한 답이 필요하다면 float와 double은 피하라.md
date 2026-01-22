### item60 정확한 답이 필요하다면 float과 double은 피하라.

- float과 double 타입은 과학과 공학 계산용으로 설계되었다. 이진 부동소수섬 연산에 쓰이며, 넓은 범위의 수를 빠르게 정밀한 `근사치`로 계산하도록 설계되었다.
- 앞의 말에서 알 수 있듯이 `근사치`를 계산하는 것이기 때문에 정확한 결과가 필요할 때는 사용하면 안 된다. (0.1 혹은 10의 음의 거듭제곱 수를 표현할 수 없기 때문이다.)
- 예를 들면

  ```java
  public static void main(String[] args) {
    double funds = 1.00;
    int itemsBought = 0;
    for(double price = 0.10; funds >= price; price += 0.10) {
      funds -= price;
      itemsBought++;
    }
    System.out.println(intemsBought + "개 구입");
    System.out.println("잔돈(달러)" + funds);
  }
  ```

  원래의 계산대로면 `0.1 + 0.2 + 0.3 + 0.4 = 1.0`으로 총 4개의 사탕을 구매하고 잔돈은 0원이 남아야 하지만, 실제로는 3개의 사탕을 구매하고 잔돈이 0.399999999999달러가 남게 된다.

  그렇다면, 이 문제를 해결하기 위해선 어떻게 해야될까? BigDecimal로 교체해주면 된다.

  ```java
  public static void main(String[] args) {
    final BigDecimal TEN_CENTS = new BigDecimal("0.10");

    int itemsBought = 0;
    BigDecimal funds = new BigDecimal("1.00");

    for(BigDecimal price = TEN_CENTS;
          funds.compareTo(price) >= 0;
          price = price.add(TEN_CENTS)) {
      funds = funds.subtract(price);
      itemsBought++;
    }
    System.out.println(intemsBought + "개 구입");
    System.out.println("잔돈(달러)" + funds);
  }
  ```

  부정확한 값이 사용되는 걸 막기 위해 BigDecimal의 생성자 중 문자열을 받는 생성자를 사용했다.

  이렇게 하면 앞의 double 코드에서 발생했던 오류를 말끔히 해결할 수 있다. 하지만 코드에서도 보이듯 문제점이 있다.

  1. 기본 타입보다 훨씬 쓰기가 불편하고 가독성이 떨어진다.
  2. 훨씬 느리다.

- 이 방법도 마음에 들지 않으면 BigDecimal 대신 int 혹은 long을 사용하면 된다. 다만 그럴 경우 다룰 수 있는 값의 크기가 제한되고, 소수점을 직접 관리해줘야 한다. long은 double의 타입만 바꿔주면 되기 때문에 생략하고 int의 예를 들어보겠다.

  int로 생성할 때는 그냥 간단하게 소수점을 없애면 된다. 즉, 달러대신 센트로 계산하는 것이다.

  ```java
  public static void main(String[] args) {
    int funds = 100;
    int itemsBought = 0;
    for(int price = 10; funds >= price; price += 10) {
      funds -= price;
      itemsBought++;
    }
    System.out.println(intemsBought + "개 구입");
    System.out.println("잔돈(센트)" + funds);
  }
  ```

`결론! 언제나 그렇듯 완전무결한 코드는 없다.`
