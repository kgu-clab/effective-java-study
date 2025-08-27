## item37. Ordinal 인덱싱 대신 EnumMap을 사용하라

- 이따금 배열이나 리스트에서 원소를 꺼낼 때 `ordinal` 메서드로 인덱스를 얻는 코드가 있다

### Ordinal 이란 무엇인가
```java
enum Day {
    MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY
}

public class Main {
    public static void main(String[] args) {
        System.out.println(Day.MONDAY.ordinal());    // 0
        System.out.println(Day.TUESDAY.ordinal());   // 1
        System.out.println(Day.WEDNESDAY.ordinal()); // 2
    }
}
```
- `Ordinal` 메서드란 `enum` 내부에서 인덱스처럼 동작한다.
  - `MONDAY` → `enum`에서 첫 번째로 선언됨 → 인덱스 0 
  - `TUESDAY` → `enum`에서 두 번째로 선언됨 → 인덱스 1 
  - `WEDNESDAY` → `enum`에서 세 번째로 선언됨 → 인덱스 2
- 이처럼 `ordinal` 메서드는 `enum` 상수의 순서를 나타내는 정수를 나타냄

### 기본 예시 코드
```java
class Plant {
    enum LifeCycle { ANNUAL, PERENNIAL, BIENNIAL }
    
    final String name;
	final LifeCycle lifeCycle;
    
    Plant(String name, LifeCycle lifeCycle) {
        this.name = name;
        this.lifeCycle = lifeCycle;
    }
}
```
- 식물을 나타내는 `Plant` 클래스로 내부에는 생애주기를 나타내는 `LifeCycle` `enum`이 있다.
- `LifeCycle`은 각각 `ANNUAL(한해살이)`, `PERENNIAL(여러해살이)`, `BIENNIAL(두해살이)`을 나타낸다.
- `final` 키워드가 붙어있으므로 초기화 이후 `불변(immutable)`이 된다

### ordinal을 배열의 인덱스로 사용하면 안되는 이유
- 식물들을 배열 하나로 관리하고 생애주기로 묶어보고자 한다.
```java
Set<Plant>[] plantByLifeCycle = (Set<Plant>[]) new Set[Plant.LifeCycle.values().length];
// 생애주기별로 총 3개의 집합이 배열로 생성됨.

for (int i = 0; i < plantByLifeCycle.length; i++)
    plantByLifeCycle[i] = new HashSet<>();

for (Plant p : garden)
    plantByLifeCycle[p.lifeCycle.ordinal()].add(p);
// garden을 순회하며 각 식물에 대해 ordinal() 메서드로 인덱스를 얻어 배열에 접근하고 있다
```
- 문제가 한가득이다
- 배열은 제네릭과 호환되지 않으니 비검사 형변환을 수행해야 하고 깔끔히 컴파일 되지 않는다.
  - 배열 타입은 `Set<Plant>[]`, 생성하려는 배열은 `Set[]`
  - 제네릭 배열을 생성하는게 불가능 하므로, 강제 캐스팅을 수행하고 그 과정에서 비검사 형변환 경고 발생.
- 가장 심각한 문제는 정확한 정숫값을 사용한다는 것을 보증해야 한다는 점이다.
  - 잘못된 값을 사용하면 잘못된 동작을 묵묵히 수행하거나 `ArrayIndexOutOfBoundsException`을 던진다.
  - 예를 들어 배열의 크기는 `3(0~2)`인데 3 이상의 값을 사용하면 `ArrayIndexOutOfBoundsException`이 발생한다.
- 하지만 열거 타입을 키로 사용하도록 설계한 `EnumMap`이 라는 해결책이 있다.

### EnumMap을 사용한 개선된 코드
```java
Map<Plant.LifeCycle, Set<Plant>> plantByLifeCycle = new EnumMap<>(Plant.LifeCycle.class);
// 배열 대신에 열거 타입을 키로 사용하는 Map을 사용한다.

for (Plant.LifeCycle lc : Plant.LifeCycle.values())
    plantByLifeCycle.put(lc, new HashSet<>());

for (Plant p : garden)
    plantByLifeCycle.get(p.lifeCycle).add(p);
```
- 안전하지 않은 형변환을 제거하였다
- 배열 인덱스를 계산하는 과정에서 오류가 날 가능성도 원천봉쇄된다
- EnumMap의 생성자가 받는 키 타입의 Class 객체는 한정적 타입 토큰으로, 런타임 제네릭 타입 정보를 제공한다.
  - 즉, 타입에 대한 정보만 제공하고 런타임에는 소거 됨.
- 내부에서 배열을 사용하지만 구현 방식을 안으로 숨겨서 Map의 타입 안정성과 배열의 성능을 모두 얻어냈다.
```java
public class EnumMap<K extends Enum<K>, V> extends AbstractMap<K, V> implements java.io.Serializable, Cloneable {
  /**
   * The {@code Class} object for the enum type of all the keys of this map.
   *
   * @serial
   */
  private final Class<K> keyType;
  
  /**
   * All of the values comprising K.  (Cached for performance.)
   */
  private transient K[] keyUniverse; // enum 상수들이 저장된 배열
  
  /**
   * Array representation of this map.  The ith element is the value
   * to which universe[i] is currently mapped, or null if it isn't
   * mapped to anything, or NULL if it's mapped to null.
   */
  private transient Object[] vals; 
  // 각 enum 키에 대응하는 값들이 저장되는 배열이며, ordinal() 값이 배열의 인덱스로 매핑되어 빠른 접근이 가능
  
  public V put(K key, V value) {
      typeCheck(key);
  
      int index = key.ordinal(); // ordinal() 메서드를 사용하여 enum 키의 인덱스를 얻음
      Object oldValue = vals[index];
      vals[index] = maskNull(value);
      if (oldValue == null)
        size++;
      return unmaskNull(oldValue);
  }
}
```

### 스트림을 사용해 맵을 관리한 코드
- 스트림을 사용하면 코드를 더 줄일 수 있다
- 다만 `EnumMap`을 사용하지 않고 고유한 맵 구현체를 사용하면 `EnumMap`을 써서 얻은 공간과 성능 이점이 사라진다
- `Collectors.groupingBy` 메서드는 `mapFactory`에 원하는 맵 구현체를 명시해 호출할 수 있다
```java
System.out.println(Arrays.stream(garden)
    .collect(groupingBy(p -> p.lifeCycle, 
        () -> new EnumMap<>(Plant.LifeCycle.class), toSet()))); // EnumMap 구현체를 명시해 호출
```
- `EnumMap` 버전은 언제나 식물의 생애주기당 하나씩의 중첩 맵을 만들지만, 스트림 버전에서는 해당 생애주기에 속하는 식물이 있을떄만 만든다.
- 정원에 한해살이와 여러해살이 식물만 살고 두해살이는 없다면 `EnumMap` 버전에서는 맵을 3개 만들고 스트림 버전에서는 2개만 만든다.
```java
public static <T, K, D, A, M extends Map<K, D>> 
Collector<T, ?, M> groupingBy(Function<? super T, ? extends K> classifier,
Supplier<M> mapFactory,
Collector<? super T, A, D> downstream) {
    // 생략 ...
  
    BiConsumer<Map<K, A>, T> accumulator = (m, t) -> {
        K key = Objects.requireNonNull(classifier.apply(t), "element cannot be mapped to a null key");
        A container = m.computeIfAbsent(key, k -> downstreamSupplier.get());
        // Map에 key가 없으면 downstreamSupplier.get()으로 새 값을 생성해서 넣고, Map에 key가 이미 있으면 기존 값을 그대로 사용
        downstreamAccumulator.accept(container, t);
	};
    // 생략 ... 
    return new CollectorImpl<>(mangledFactory, accumulator, merger, finisher, CH_NOID);
}
```

### 두 열거 타입을 매핑해야하는 기본 예시 코드
```java
public enum Phase {
    SOLID, LIQUID, GAS;
    
    public enum Transition {
        MELT, FREEZE, BOIL, CONDENSE, SUBLIME, DEPOSIT;
        
        private static final Transition[][] TRANSITIONS = {
            { null,         MELT,        SUBLIME },   // SOLID 에서 출발
            { FREEZE,      null,        BOIL    },   // LIQUID 에서 출발
            { DEPOSIT,     CONDENSE,    null    }    // GAS 에서 출발
        };
        
        public static Transition from(Phase from, Phase to) {
            return TRANSITIONS[from.ordinal()][to.ordinal()];
        }
    }
}
```
- 위 코드에서 컴파일러는 `ordinal`과 배열 인덱스의 관계를 알 도리가 없다
- `ArrayIndexOutOfBoundsException`이나 `NullPointerException`을 던질 수도 있고 이상하게 동작할 수도 있다
- 상태의 가짓수가 늘어나면 제곱해서 커지며 `null`로 채워지는 칸도 늘어날 것이다.

### 중첩 EnumMap을 사용한 개선된 코드
- 전이 하나를 얻으려면 이전 상태(from)와 이후 상태(to) 두 개가 필요하니, 맵 2개를 중첩하면 쉽게 해결할 수 있다
```java
public enum Phase {
    SOLID, LIQUID, GAS;
    
    public enum Transition {
        MELT(SOLID, LIQUID), FREEZE(LIQUID, SOLID),
        BOIL(LIQUID, GAS), CONDENSE(GAS, LIQUID),
        SUBLIME(SOLID, GAS), DEPOSIT(GAS, SOLID);

        Transition(Phase from, Phase to) {
            this.from = from;
            this.to = to;
        }

        private static final Map<Phase, Map<Phase, Transition>> m =
            Stream.of(values()).collect(
                groupingBy(t -> t.from,
                    () -> new EnumMap<>(Phase.class),
                    toMap(t -> t.to, t -> t, (x, y) -> y, () -> new EnumMap<>(Phase.class))
                )
            );
	}
}
```
- 맵을 초기화하는 코드는 다소 복잡하다
- 맵을 초기화하기 위해 `Collector` 2개를 차례로 사용했다
  - 첫 번째 수집기인 `groupingBy`는 `Transition`을 `from` 상태별로 묶고
  - 두 번째 수집기인 `toMap`은 `to`를 `Transition`에 매핑하는 `EnumMap`을 만든다
- 내부에서는 맵들의 맵이 배열들의 배열로 구현되니 낭비되는 공간과 시간도 거의 없이 명확하고 안전하고 유지보수하기 좋다
