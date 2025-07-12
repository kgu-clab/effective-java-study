## item20 추상 클래스보다는 인터페이스를 우선하라

- 자바가 제공하는 다중 구현 케머니즘은 인터페이스와 추상 클래스 2가지이다.
- 이 둘의 차이점: 추상 클래스가 정의한 타입을 구현하는 클래스는 반드시 추상 클래스의 하위 클래스가 되어야 한다. 이는 자바의 기본인 단일 상속에 있어 커다란 제약을 안고 있는 것이다.
- 반면 인터페이스가 선언한 메서드를 모두 정의하고 그 일반 규약을 잘 지킨 클래스라면 다른 어떤 클래스를 상속했든 같은 타입으로 취급된다.

### 기존 클래스에도 손쉽게 새로운 인터페이스를 구현해넣을 수 있다.
1. 인터페이스가 요구하는 메서드를 추가한다.
2. 클래스 선언에 implements 구문만 추가하면 끝!
- Comparable, Iterable, AutoCloseable 등이 이런 식으로 추가됐다.
- 반면 기존 클래스에 새로운 추상 클래스르 끼워넣는 것은 어렵다. 단일 상속에 의해 이미 다른 클래스를 상속받고 있을 경우에도 문제가 생긴다.
- 또 두 클래스가 같은 추상 클래스를 확장하길 원한다면 그 추상 클래스는 계층구조상 두 클래스의 공통 조상이어야 한다. 이때 새로 추가된 추상 클래스의 모든 자손이 이를 강제로 상속하게 되는 문제점 또한 존재한다.

### 인터페이스는 `믹스인(mixin)` 정의에 안성맞춤이다.
- `믹스인`: 클래스가 구현할 수 있는 타입, 믹스인을 구현한 클래스에 원래의 '주된 타입' 외에도 특정 선택적 행위를 제공한다고 선언하는 효과를 준다.
- 예를 들어, Comparable은 자신이 구현한 인스턴스들끼리 순서를 정할 수 잇다고 선언하는 믹스인 인터페이스이다.
- 즉, 믹스인은 대상 타입의 주된 기능에 선택적 기능을 '혼합'하는 것이다.
- 이 경우에도 추상 클래스는 단일 상속이라는 구조때문에 오류를 유발할 수 있다.

### 인터페이스로는 계층구조가 없는 타입 프레임워크를 만들 수 있다.
- 타입을 계층적으로 정의하면 수 많은 개념을 구조적으로 잘 표현할 수 있지만, 현실에는 계층을 엄격히 구분하기 어려운 개념도 있다.
- 이를 클래스로 구현하려면 n개의 속성을 지원하기 위해선 2^n개의 조합이 필요하며, 흔히 조합 폭발이라고 부르는 현상이 발생한다.

### (래퍼 클래스 관용구와 함께 사용하면)인터페이스는 기능을 향상시키는 안전하고 강력한 수단이 된다.
- 타입을 추상 클래스로 정의해두면 그 타입에 기능을 추가하는 방법은 상속뿐이다. 상속해서 만든 클래스는 래퍼 클래스보다 활용도가 떨어지고 깨지기는 더 쉽다.
- 인터페이스의 메서드 중 구현 방법이 명백한 것이 있다면, 그 구현을 디폴트 메서드로 제공해 프로그래머들의 일감을 덜어줄 수 있다.
  `템플릿 메서드 패턴`
- 인터페이스와 추상 골격 구현 클래스를 함께 제공하는 식으로 인터페이스와 추상 클래스의 장점을 모두 취하는 방법이다.
    - 인터페이스로는 타입을 정의하고, 필요한 디폴트 메서드 몇 개도 함께 제공한다. 그리고 골격 구현 클래스는 나머지 메서드들까지 구현한다.
    - 관례상 인터페이스 이름이 Interface라면 골격 구현 클래스의 이름은 AbstractInterface라고 짓는다.

  ```java
  static List<Integer> intArrayAslist(int[] a){
    Object.requireNonNull(a);

    return new AbstractList<>(){
      @Override public Integer get(int i){
        return a[i];
      }

      @Override public Integer set(int i, Integer val){
        int oldVal = a[i];
        a[i] = val;
        return oldVal;
      }

      @Override public int size(){
        return a.length;
      }
    }
  }
  ```
- 골격 구현 클래스를 이용하지 않고 인터페이스를 직접 구현하여 골격 구현 클래스를 우회적으로 이용하는 시뮬레이트한 다중 상속 방법도 있다.

- 골격 구현 작성 방법
    1. 인터페이스를 잘 살펴 다른 메서드들의 구현에 사용되는 기반 메서드들을 선정한다.(이 기반 메서드들은 골격 구현에서 추상 메서드가 될 것임)
    2. 기반 메서드들을 사용해 직접 구현할 수 있는 메서드를 디폴트 메서드로 제공한다.(equals같은 Object의 메서드는 디폴트 메서드로 제공하면 안 된다!)
    3. 기반 메서드나 디폴트 메서드로 만들지 못한 메서드가 남아 있다면, 이 인터페이스를 구현하는 골격 구현 클래스를 하나 만들어 남은 메서들을 작성해 넣는다. 골격 구현 클래스에는 필요한 public이 아닌 필드와 메서드를 추가해도 된다.
  ```java
  public abstract class AbstractMapEntry<K, V> implements Map.Entry<K, V> {
    @Override public V setValue(V value){
      throw new UnsupportedOperationException();
    }

    @Override public boolean equals(Object o){
      if (o == this) return true;
      if (!(o instanceof Map.Entry)) return false;
      Map.Entry<?,?> e = (Map.Entry) o;
      return Objects.equals(e.getKey(), getKey()) && Objects.equals(e.getValue(), getValue());
    }

    @Override public int hashCode() {
      return Objects.hashCode(getKey()) ^ Objects.hashCode(getValue());
    }

    @Override public String toString(){
      return getKey() + "=" + getValue;
    }
  }
  ```
    - Map.Entry 인터페이스나 그 하위 인터페이스로는 이 골격 구현을 제공할 수 없다. 디폴트 메서드는 equals, hashCode, toString같은 Object 메서드를 재정의할 수 없기 때문이다.

--- 
### 핵심 정리
- 일반적으로 다중 구현용 타입으로는 인터페이스가 가장 적합하다. 복잡한 인터페이스라면 구현하는 수고를 덜어주는 골격 구현을 함께 제공하는 방법을 꼭 고려해보자. 골격 구현은 '가능한 한' 인터페이스의 디폴트 메서드로 제공하여 그 인터페이스를 구현한 모든 곳에서 활용하도록 하는 것이 좋다. '가능한 한'이라고 한 이유는, 인터페이스에 걸려 있는 구현상의 제약 떄문에 골격 구현을 추상 클래스로 제공하는 경우가 더 흔하기 때문이다.