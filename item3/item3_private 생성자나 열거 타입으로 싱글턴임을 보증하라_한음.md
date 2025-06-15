## item3. private 생성자나 열거 타입으로 싱글턴임을 보증하라

![img.png](../images/singleton_pattern.png)
- 싱글턴(singleton)이란 인스턴스를 오직 하나만 생성할 수 있는 클래스이다
- 그런데 클래스를 싱글턴으로 만들면 이를 사용하는 클라이언트를 테스트하기가 어려워질 수 있다
- 가짜 객체를 만들어서 테스트하기가 어렵기 때문이다
---
- 싱글턴을 만드는 방식은 둘 중 하나이다
- 모두 생성자는 private으로 감춰두고, 유일한 인스턴스에 접근할 수 있는 수단으로 public static 멤버를 하나 둔다

1. private static final 방식
```java
public class Singleton {
    public static final Singleton INSTANCE = new Singleton(); // 멤버가 public

    private Singleton() {
        // 생성자는 private으로 감춰둔다
    }
}
```
- 단, AccessibleObject.setAccessible() 같은 리플렉션 API를 사용하면 private 생성자에 접근할 수 있다

2. 정적 팩터리 메서드를 public static으로 제공하는 방식
```java
public class Singleton {
	private static final Singleton INSTANCE = new Singleton(); // 멤버가 private

    private Singleton() {
        // 생성자는 private으로 감춰둔다
    }

    public static Singleton getInstance() {
        if (instance == null) {
            instance = new Singleton();
        }
        return instance;
    }
}
```
- 이제 싱글턴 객체는 `Singleton.getInstance()`를 통해서만 접근할 수 있다
- 이 방법의 장점은 스레드별로 다른 인스턴스를 넘겨주게 할 수 있다 (싱글턴을 스레드별로 적용)
- 두번째 장점은 정적 팩터리를 제네릭 싱글턴으로 만들 수 있다는 점이다 (아이템 30 참고)
- 마지막은 정적 팩터리의 메서드 참조를 공급자(supplier)로 사용할 수 있다는 점이다

3. 원소가 하나인 열거타입을 선언하는 것
```java
public enum Singleton {
    INSTANCE; // 유일한 인스턴스
}
```


