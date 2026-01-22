# 아이템 79 - 과도한 동기화는 피하라

### 과도한 동기화는 성능을 떨어뜨리고 , DeadLock에 빠뜨리고, 예측할 수 없는 오동작을 이르킨다.

## 응답 불가와 안전 실패를 피하려면 동기화 메서드나 동기화 블록 안에서는 제어를 절대로 클라이언트에 양도하면 안 된다.

동기화된 영역 안에서는 재정의할 수 있는 메서드는 호출하면 안 되며, 클라이언트가 넘겨준 함수 객체를 호출해서도 안 된다. 동기화된 영역을 포함한 클래스 관점에서는 이런 메서드는 모두 외계인이다. 그 메서드가 무슨 일을 할지 알지 못하며 통제할 수도 없다. `외계인 메서드(alien method)`가 하는 일에 따라 동기화된 영역은 예외를 일으키거나, 교착상태에 빠지거나 데이터를 훼손할 수도 있다.

→ 외계인 메서드란?

주로 동시성(concurrency) 환경에서 **동작을 예측하거나 통제할 수 없는 외부 메서드**

### 동기화 블록에서의 외계인 메서드 호출

```java
public class ObservableSet<E> extends ForwardingSet<E> {
    public ObservableSet(Set<E> set) { super(set); }
    
    private final List<SetObserver<E>> observers
            = new ArrayList<>();

    public void addObserver(SetObserver<E> observer) {
        synchronized(observers) {
            observers.add(observer);
        }
    }

    public boolean removeObserver(SetObserver<E> observer) {
        synchronized(observers) {
            return observers.remove(observer);
        }
    }

    private void notifyElementAdded(E element) {
        synchronized(observers) {
            for (SetObserver<E> observer : observers)
                observer.added(this, element);
        }
    }
    
    @Override public boolean add(E element) {
        boolean added = super.add(element);
        if (added)
            notifyElementAdded(element);
        return added;
    }

    @Override public boolean addAll(Collection<? extends E> c) {
        boolean result = false;
        for (E element : c)
            result |= add(element);  // notifyElementAdded를 호출한다.
        return result;
    }
}
```

어떤 집합을 감싼 래퍼 클래스이고, 이 클래스의 클라이언트는 집합에 원소가 추가되면 알람을 받는 관찰자 패턴이다

→ 관찰자들은 addObserver와 removeObserver 메서드를 호출해 구독을 신청하거나 해지한다. 두 경우 모두 다음 콜백 인터페이스의 인스턴스를 메서드에 건넨다.

```java
public interface SetObserver<E> {
    // ObservableSet에 원소가 더해지면 호출된다.
    void added(ObservableSet<E> set, E element);
}
```

### 예외 발생

ObservableSet은 잘 작동할 것으로 보이지만, 클라이언트 코드가 다음과 같다면 어떨까?

```java
public static void main(String[] args) {
    ObservableSet<Integer> set =
            new ObservableSet<>(new HashSet<>());

    set.addObserver(new SetObserver<>() {
            public void added(ObservableSet<Integer> s, Integer e) {
                System.out.println(e);
                if (e == 23) // 값이 23이면 자신을 구독해지한다.
                    s.removeObserver(this);
            }
        });

    for (int i = 0; i < 100; i++)
        set.add(i);
}
```

이 프로그램은 0부터 23까지 출력한 후 관찰자 자신을 구독 해지한 다음 종료하길 기대했지만, 실제로는 23까지 출력한 후 ConcurrentModificationException 을 던진다.

그 이유는 관찰자의 added 메서드 호출이 일어난 시점이 notifyElementAdded 가 관찰자들의 리스트를 순회하는 도중이기 때문이다.

observers.remove 메서드를 호출할 때, notifyElementAdded 메서드에서 이 리스트를 순회하는 도중이기 때문에 허용되지 않은 동작이다. notifyElementAdded 메서드 내의 동기화 블록은 동시 수정이 일어나지 않도록 보장하지만, 정작 콜백을 거쳐 되돌아와 수정하는 것까지 막지는 못한다.

### 교착상태

이번엔 구독해지를 하는 관찰자가 removeObserver 를 직접 호출하지 않고 실행자 서비스(ExecutorService)를 사용해 다른 스레드에게 부탁하는 예를 보자.

```java
public static void main(String[] args) {
    ObservableSet<Integer> set =
            new ObservableSet<>(new HashSet<>());

	// 쓸데없이 백그라운드 스레드를 사용하는 관찰자
    set.addObserver(new SetObserver<>() {
        public void added(ObservableSet<Integer> s, Integer e) {
            System.out.println(e);
            if (e == 23) {
                ExecutorService exec =
                        Executors.newSingleThreadExecutor();
                try {
                    exec.submit(() -> s.removeObserver(this)).get();
                } catch (ExecutionException | InterruptedException ex) {
                    throw new AssertionError(ex);
                } finally {
                    exec.shutdown();
                }
            }
        }
    });

    for (int i = 0; i < 100; i++)
        set.add(i);
}
```

이 프로그램을 실행하면 예외는 발생하지 않지만 교착상태에 빠진다.

백그라운드 스레드가 s.removeObserver 를 호출하면, 관찰자를 잠그려 하지만 락을 얻을 수 없다. notifyElementAdded 메서드를 실행 중이기 때문에 메인 스레드가 락을 쥐고 있기 때문이다. 그리고 메인 스레드는 백그라운드 스레드가 관찰자를 제거하기를 기다리고 있다. (교착상태)

### 외계인 메서드 호출을 동기화 블록 바깥으로 옮기기

이런 문제는 외계인 메서드 호출을 동기화 블록 바깥으로 옮기면 해결할 수 있다.

notifyElementAdded 메서드에서라면 관찰자 리스트를 복사해 쓰면 락 없이도 안전하게 순회할 수 있다.

```java
private void notifyElementAdded(E element) {
    List<SetObserver<E>> snapshot = null;
    synchronized(observers) {
        snapshot = new ArrayList<>(observers);
    }
    for (SetObserver<E> observer : snapshot)
        observer.added(this, element);
}
```

이 방법 외에도 자바의 동시성 컬렉션 라이브러리의 `CopyOnWriteArrayList` 를 사용하는 방법도 있다. 이는 ArrayList 를 구현한 클래스로, 내부를 변경하는 작업은 항상 깨끗한 복사본을 만들어 수행하도록 구현되어 있다. 내부의 배열은 절대 수정되지 않으니 순회할 때 락이 필요 없어 매우 빠르다. 다른 용도로 쓰인다면 끔찍이 느리겠지만, 수정할 일이 드믈고 순회만 빈번히 일어나는 관찰자 리스트 용도로는 최적이다.

```java
private final List<SetObserver<E>> observers =
        new CopyOnWriteArrayList<>();

public void addObserver(SetObserver<E> observer) {
    observers.add(observer);
}

public boolean removeObserver(SetObserver<E> observer) {
    return observers.remove(observer);
}

private void notifyElementAdded(E element) {
    for (SetObserver<E> observer : observers)
        observer.added(this, element);
}
```

### 열린 호출 (open call)

동기화 영역 바깥에서 호출되는 외계인 메서드를 열린 호출(open call) 이라고 한다. 외계인 메서드는 얼마나 오래 실행될지 알 수 없는데, 동기화 영역 안에서 호출된다면 그 동안 스레드는 보호된 자원을 사용하지 못하고 대기해야만 한다. 따라서 열린 호출은 실패 방지 효과 외에도 동시성 효율을 크게 개선해준다.

**기본 규칙은 동기화 영역에서는 가능한 한 일을 적게 하는 것이다.** 락을 억고, 공유 데이터를 검사하고, 필요하면 수정하고, 락을 놓는다.

### 과도한 동기화의 비용

멀티코어가 일반화된 오늘날, 과도한 동기화가 초래하는 진짜 비용은 락을 얻는 데 드는 CPU 시간이 아니다.

바로 **경쟁하느라 낭비하는 시간**, 즉 병렬로 실행할 기회를 잃고 모든 코어가 메모리를 일관되게 보기 위한 지연시간이 진짜 비용이다. **가상머신의 코드 최적화를 제한**한다는 점도 숨겨진 비용이다.

### 가변 클래스 지침

가변 클래스를 작성한다면 두 선택지 중 하나를 따르자.

1. 동기화를 전혀 하지 말고, 그 클래스를 동시에 사용해야 하는 클래스가 외부에서 알아서 동기화하게 하자.
2. 동기화를 내부에서 수행해 스레드 안전한 클래스로 만들자. 단, 클라이언트가 외부에서 객체 전체에 락을 거는 것보다 동시성을 월등히 개선할 수 있을 때만 선택해야 한다.

선택하기 어렵다면 동기화하지 말고, 대신 문서에 "스레드 안전하지 않다"고 명시하자.

만약 클래스 내부에서 동기화하기로 했다면, 락 분할(lock splitting), 락 스트라이핑(lock striping), 비차단 동시성 제어(nonblocking concurrency control) 등 다양한 기법을 동원해 동시성을 높여줄 수 있다.