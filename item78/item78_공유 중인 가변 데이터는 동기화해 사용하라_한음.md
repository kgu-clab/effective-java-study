## item78. 공유 중인 가변 데이터는 동기화해 사용하라

- synchronized 키워드는 특정 메서드나 블록을 동시에 한 스레드만 수행할 수 있도록 보장한다

### 동기화를 바라보는 관점
- 한 스레드가 변경하는 상태를 다른 스레드가 보지 못하게 한다
- 동기화 된 스레드는 이전 수정의 최종 결과를 보게 한다

### 동기화가 필요한 이유
- 성능을 높이려면 동기화하지 말아야 겠다 생각하기 쉬운데 위험한 발상이다
- 자바 언어 명세는 스레드가 필드를 읽을 때 항상 '수정이 완전히 반영 된 값'을 얻는다고 보장하지만
- 한 스레드가 저장한 값이 다른 스레드에게 보이는가는 보장하지 않는다

### 왜 한 스레드가 저장한 값의 변경을 보장할 수 없을까?
- 스레드는 락을 획득하기 전까지의 쓰기를 자신의 캐시에 복붙한다
- 다른 스레드가 값을 변경해도 자신의 캐시에는 반영되지 않는다

### 자바 최적화로 인한 동기화 보장 안됨 예시
```java
public class StopThread {
    private static boolean stopRequested;

    public static void main(String[] args) throws InterruptedException {
        Thread backgroundThread = new Thread(() -> {
            int i = 0;
            while (!stopRequested) {
                i++;
            }
        });
        backgroundThread.start();

        TimeUnit.SECONDS.sleep(1);
        stopRequested = true;
    }
}
```
- 1초 뒤에 프로그램이 종료될 것 처럼 보인다
- 그런데 실제로는 종료되지 않는다
- JDK 내부적으로 다음과 같은 최적화가 이뤄지기 때문

```java
// 원래 코드
while(!stopRequested) {
    i++;
}

// 최적화한 코드
if (!stopRequested) {
    while (true) { // 무한 루프
        i++; 
    }
}
```
- synchronized 키워드를 사용하면 이 문제를 해결할 수 있다

```java
public class StopThread {
    private static boolean stopRequested;
	
	private static synchronized void requestStop() {
		stopRequested = true;
	}
	
	private static synchronized boolean stopRequested() {
		return stopRequested;
    }

    public static void main(String[] args) throws InterruptedException {
        Thread backgroundThread = new Thread(() -> {
            int i = 0;
            while (!stopRequested()) {
				i++;
            }
        });
        backgroundThread.start();

        TimeUnit.SECONDS.sleep(1);
        requestStop();
    }
}
```
- 쓰기 메서드와 읽기 메서들르 모두 동기화 했음에 주목하자
- 쓰기만 동기화해서는 동작을 보장할 수 없다!

### 동기화 대안: volatile
- 매번 읽기와 쓰기를 동기화 하는 대신에 volatile 키워드를 사용할 수 있다
- volatile 키워드는 변수에 대한 모든 읽기와 쓰기가 주 메모리에서 이뤄지도록 보장한다

```java
public class StopThread {
    private static volatile boolean stopRequested;

    public static void main(String[] args) throws InterruptedException {
        Thread backgroundThread = new Thread(() -> {
            int i = 0;
            while (!stopRequested) {
                i++;
            }
        });
        backgroundThread.start();

        TimeUnit.SECONDS.sleep(1);
        stopRequested = true;
    }
}
```
- 단, volatile 키워드는 원자성을 보장하지 않는다
- 즉, 특정 블록이 읽고 쓰는 중간에 다른 스레드가 개입할 수 있다

### 정리
- 여러 스레드가 가변 데이터를 공유한다면 그 데이터를 읽고 쓰는 동작은 반드시 동기화 해야 한다.
- synchronized 키워드를 사용하거나 volatile 키워드를 사용하자
