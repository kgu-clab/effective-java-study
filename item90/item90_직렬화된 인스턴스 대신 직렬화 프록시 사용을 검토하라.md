### item90 직렬화된 인스턴스 대신 직렬화 프록시 사용을 검토하라

- 직렬화: 데이터를 전송하기 위해 바이트 형태로 변환하는 기술
- 생성자가 아닌 직렬화를 통해 생성된 객체는 버그와 보안 문제가 일어날 가능성이 커진다. 이 위험을 줄이기 위해 있는 방법이 바로 `직렬화 프록시 패턴`이다.
- 직렬화 프록시 패턴은

1. 바깥 클래스의 핵심 데이터만 들고 있는 중첩 클래스를 설계해 private static으로 선언한다. 이 중첩 클래스가 바로 바깥 클래스의 직렬화 프록시다.(중첩 클래스의 생성자는 단 하나여야 하고, 바깥 클래스를 매개변수로 받아여 한다.)

```java
public class Period implements Serializable { //바깥 클래스
    private final Date start;
    private final Date end;

    public Period(Date start, Date end) {
        if (start.compareTo(end) > 0) throw new IllegalArgumentException();
        this.start = start;
        this.end = end;
    }

    private static class SerializationProxy implements Serializable { //중첩 클래스(직렬화 프록시)
        private final Date start;
        private final Date end;

        SerializationProxy(Period p) { //논리적 상태 정밀
            this.start = p.start;
            this.end = p.end;
        }
    }
}
```

2. 바깥 클래스에 writeReplace 메서드를 추가한다.

```java
private Object writeReplace() { //자바 직렬화 시스템이 이 메서드를 발견하면, Period 대신 Proxy를 저장한다.
  return new SerializationProxy(this);
}
```

3. 보안상의 이유로 readObeject 메서드를 바깥 클래스에 추가한다.

```java
//공격자가 Period를 직접 역직렬화 하려는 것을 막아준다.
private void readObeject(ObjectInputStream stream) throws InvalidObjectException {
  throw new InvaildObjectException("프록시가 필요합니다.")
}
```

4. 역직렬화 시 진짜 객체인 Period로 바꾸기 위한 readResolve 메서드를 SerializationProxy 클래스에 추가하면 끝이다.

```java
private Object readResolve() {
  return new Period(start, end);
}
```

- 직렬화 프록시 패턴의 한계

1. 클라이언트가 멋대로 확장할 수 있는 클래스에는 적용할 수 없다.
2. 객체 그래프에 순환이 있는 클래스에도 적용할 수 없다. 이런 객체의 메서드를 직렬화 프록시의 readResolve 안에서 호출하려 하면 ClassCastException이 발생할 것이다. 직렬화 프록시만 가졌을 뿐 실제 객체는 아직 만들어진 것이 아니기 때문이다.
3. 이 패턴이 강력함과 안전성을 주는 만큼 성능의 저하가 있을 수 있다.
