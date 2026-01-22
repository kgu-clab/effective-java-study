### item84 프로그램의 동작을 스레드 스케줄러에 기대지 말라

- 잘 작성된 프로그램이라면 스레드 스케줄링 정책에 좌지우지 돼서는 안 된다. 정확성이나 성능이 스레드 스케줄러에 따라 달라지는 프로그램은 다른 플랫폼에 이식하기 어렵다.
- 좋은 프로그램을 작성하는 가장 좋은 방법은 실행 가능한 `스레드의 평균적인 수를 프로세서 수보다 지나치게 많아지지 않도록 하는 것이다. 또 실행 준비가 된 스레드들은 맡은 작업을 완료할 때까지 계속 실행되도록 만들자.` (대기 중인 스레드는 실행 가능하지 않다.)
- 실행 가능한 스레드 수를 적게 유지하는 방법은 각 스레드가 무언가 유용한 작업을 완료한 후에는 다음 일거리가 생길 때까지 대기하도록 하는 것이다. `스레드가 당장 처리해야 할 작업이 없다면 실행돼서는 안 된다.`
- 스레드는 절대 바쁜 대기(busy waiting) 상태가 되면 안 된다.
  -> 공유 객체의 상태가 바뀔 때까지 쉬지 않고 검사해서는 안 된다는 뜻이다.
- 예시

```java
public class SlowCountDownLatch {
  private int count;

  public SlowCountDownLatch(int count) {
    if (count < 0)
      throw new IllegalArgumentException(count + " < 0");
    this.count = count;
  }

  public void await() {
    while(true) { // 여기서 바쁜 대기 상태가 발생
      synchronized(this) {
        if (count == 0)
          return;
      }
    }
  }

  public sychronized void countDown() {
    if (count != 0)
      count--;
  }
}
```

- 하나 이상의 스레드가 필요도 없이 실행 가능한 상태인 시스템은 흔하게 볼 수 있다. 이런 시스템은 성능과 이식성이 떨어질 수 있다.
