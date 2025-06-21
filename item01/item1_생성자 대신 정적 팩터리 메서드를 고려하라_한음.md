## item1. 생성자 대신 정적 팩토리 메서드를 고려하라

- 클라이언트가 클래스의 인스턴스를 얻는 전통적인 수단은 **public 생성자**이다
```java
public class User {
    private final String name;

    public User(String name) { // 생성자
        this.name = name;
    }
}
```

- 하지만 생성자와 별도로 **정적 팩터리 메서드(static factory method)**를 제공할 수도 있다
- 다음 코드는 Boolean 타입의 정적 팩토리 메서드이다
```java
public class Boolean {
    private final boolean value;
	
    public static Boolean valueOf(boolean b) { // 정적 팩토리 메서드: boolean 값을 받아 Boolean 객체를 반환
        return b ? Boolean.TRUE : Boolean.FALSE;
    }
}
```
- public 생성자 대신 정적 팩터리 메서드를 제공하면 다음과 같은 장점이 있다

### 정적 팩토리 메서드의 장점
1. **이름을 가질 수 있다**
   - 생성자 자체 만으로는 반환될 객체의 특정을 제대로 설명할 수가 없다
   - 반면, 정적 팩터리 메서드는 이름을 가질 수 있어 코드의 가독성을 높인다.
   - 한 클래스에 시그니처가 같은 생성자가 여러개 필요할 거 같으면, 정적 팩터리 메서드를 사용하고, 각각의 차이를 잘 드러내는 이름을 지어주자
```java
public class User {
    private final String name;

    public User(String name) { // 생성자
        this.name = name;
    }

    public static User create(String name) { // 메서드의 목적을 더 잘 드러내준다
        return new User(name);
    }
}
```

2. **호출될 때마다 인스턴스를 새로 생성하지는 않아도 된다**
   - 정적 팩토리 메서드는 호출할 때마다 새로운 인스턴스를 생성하지 않고, 캐싱된 인스턴스를 반환할 수 있다
   - 질문: 성능을 상당히 끌어올려주는데, 이유가 뭘까요? (힌트: JVM)
   <!-- 정적 팩터리 메서드는 컴파일 타임에 클래스 로딩 시점에 함께 로드되어, JVM의 메서드 영역에 단 한 개의 인스턴스로 유지된다 -->
   

3. **반환 타입의 하위 타입 객체를 반환할 수 있는 능력이 있다**
    - 반환할 객체의 클래스를 자유롭게 선택할 수 있게 하는 엄청난 유연성을 선물한다
```java
public interface Animal {
    void speak();
}

public class Dog implements Animal {
    public void speak() {
        System.out.println("멍멍!");
    }
}

public class Cat implements Animal {
    public void speak() {
        System.out.println("야옹~");
    }
}

public class AnimalFactory {
    public static Animal getAnimal(String type) {
        if (type.equals("dog")) {
            return new Dog(); // Dog는 Animal의 하위 타입
        } else {
            return new Cat(); // Cat도 Animal의 하위 타입
        }
    }
}
```

4. **입력 매개변수에 따라 매번 다른 클래스의 객체를 반환할 수 있다**
   - 정적 팩토리 메서드는 입력 매개변수에 따라 다른 클래스의 객체를 반환할 수 있다
   - 위에 예시 코드 참고
```java
public class Main {
    public static void main(String[] args) {
        Animal dog = AnimalFactory.getAnimal("dog");
        dog.speak(); // 멍멍!

        Animal cat = AnimalFactory.getAnimal("cat");
        cat.speak(); // 야옹~
    }
}
```

5. **정적 팩터리 메서드를 작성하는 시점에는 반환할 객체의 클래스가 존재하지 않아도 된다**
   - 이는 API 설계 시 유용하게 활용될 수 있다
   - 다만, Spring Framework 같은 서비스 구현체에 대한 이해가 필요하니 넘어가도록 하자

### 정적 팩토리 메서드의 단점
1. **상속을 하려면 public이나 protected 생성자가 필요하니 정적 팩터리 메서드만 제공하면 하위 클래스를 만들 수 없다**
    - Boolean 클래스는 생성자가 private로 선언되어 있어, 하위 클래스를 만들 수 없다
    - valueOf 메서드로만 인스턴스를 얻도록 강제하고 있다 (장점이 될 수도, 단점이 될 수도)
```java
public final class Boolean {
    private final boolean value;

    private Boolean(boolean value) {
        this.value = value;
    }

    public static final Boolean TRUE = new Boolean(true);
    public static final Boolean FALSE = new Boolean(false);

    public static Boolean valueOf(boolean b) {
        return b ? TRUE : FALSE;
    }
}
```

2. **정적 팩터리 메서드는 프로그래머가 찾기 어렵다**
    - public 생성자는 클래스 이름과 동일한 이름을 가지므로, 클래스의 인스턴스를 얻는 전통적인 방법으로 쉽게 찾을 수 있다
    - 반면, 정적 팩터리 메서드는 클래스 이름과 다르기 때문에, 프로그래머가 찾기 어려울 수 있다
    - 따라서, 정적 팩터리 메서드를 제공할 때는 문서화가 중요하다

- 다음은 정적 팩터리 메서드에 흔히 사용되는 이름들이다
```java
public enum Type {
	ADMIN,
	GUEST,
	MEMBER
}

public class User {
	private final String name;
	private final int age;
	private final Type type;
	
	public static User from(String name) { // from: 매개변수 하나로 객체 생성 (ex. 이름 → 기본 GUEST 타입)
		return new User(name, 0, UserType.GUEST);
	}
	
	public static User of(String name, int age) { // of: 여러 매개변수를 받아 객체 생성
		return new User(name, age, UserType.MEMBER);
	}
	
	public static User create(String name) { // create: 일반적인 새 객체 생성
		return new User(name, 0, UserType.MEMBER);
	}
	
	public static User getInstance(String name) { // getInstance: 특정 조건에 따라 캐싱된 인스턴스를 줄 수도 있음 
		return new User(name, 0, UserType.GUEST);
	}
	
	public static User getType(String name, int age, String typeName) { // getType: 타입 이름으로부터 User 생성
		UserType type = UserType.valueOf(typeName.toUpperCase());
		return new User(name, age, type);
	}
	
}
```

### 핵심 정리
- 정적 팩터리 메서드와 public 생성자는 각자의 쓰임새가 있으니 상대적인 장단점을 이해하고 사용하는 것이 좋다.
- 그렇다고 하더라도 정적 팩터리를 사용하는 게 유리한 경우가 더 많으므로 무작정 public 생성자를 제공하던 습관이 있다면 고치자