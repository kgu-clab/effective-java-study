## item27 비검사 경고를 제거하라

- 모든 비검사 경고를 제거한다면 그 코드는 타입 안정성이 보장된다. 즉, 런타임에 ClassCastException이 발생할 일이 없고, 우리가 의도한 대로 잘 동작하리라 확신할 수 있다.
- 경고를 제거할 수는 없지만 타입 안전하다고 확신할 수 있다면 @SuppressWarnings("unchecked") 애너테이션을 달아 경고를 숨길 수 있다.
- 만약 안전하다고 검증된 비검사 경고를 그대로 두면, 진짜 문제를 알리는 새로운 경고가 나와도 눈치채지 못할 수 있다. 따라서 안전함이 검증됐다면 숨겨주는 습관을 가지도록 하자.
- @SuppressWarnings 애너테이션은 개별 지역변수 선언부터 클래스 전체까지 어떤 선언에도 달 수 있지만 항상 가능한 한 좁은 범위에 적용하는 것이 좋다. (보통 변수 선언, 아주 짧은 메서드 혹은 생성자) 클래스같은 큰 범위에 적용하면 심각한 경고를 놓칠 수 있기 때문이다.
- 한 줄이 넘는 메서드나 생성자에 달린 @SuppressWarnings 애너테이션을 발견하면 지역변수 선언 쪽으로 옮기자.
- 예를 들어,

```java
  public <T> T[] toArray(T[] a) {
    if (a.length < size)
      return (T[]) Arrays.copyOf(elements, size, a.getClass());
    System.arraycopy(elements, 0, a, 0, size);
    if(a.length > size)
      a[size] = null;
    return a;
  }
```

ArrayList의 toArray 메서드를 컴파일 하면 아래와 같은 경고가 발생한다.

```java
  ArrayList.java:305: warning: [unchecked] unchecked cast
        return (T[]) Arrays.copyOf(elements, size, a.getClass());
    required: T[]
    found: Object[]
```

@SuppressWarnings 애너테이션은 선언에만 달 수 있기 때문에 return문에는 달지 못한다. 그렇다면 메서드 자체에 달고 싶겠지만, 범위가 필요이상으로 넓어지게 된다. 그 대신 반환값을 담을 지역변수를 하나 선언하고 그 변수에 @SuppressWarnings 애너테이션을 달아주면 된다.

- _@SuppressWarnings 애너테이션을 사용할 때면 그 경고를 무시해도 안전한 이유를 항상 주석으로 남겨야 한다._
