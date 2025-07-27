# item25 : 톱 레벨 클래스는 한 파일에 하나만 담으라

### 하나의 소스 파일에 톱레벨 클래스를 여러개 선언하는 것
- 큰 문제는 없다.
- 하지만 아무런 득도 없다.
- 심각한 리스크를 감수해야 한다.

main클래스
```java
public class Main {
	public static void main(String[] args) {
		System.out.println(Utensil.NAME + Dessert.NAME);
	}
}
```
Utensil과 Dessert 클래스가 Utensil.java라는 한 파일에 정의 되어있다고 해보자
```java
// 파일명 : Utensil.java

class Utensil {
	static final String NAME = "pan";
}

class Dessert {
	static final String NAME = "cake";
}
```

-> Main 실행 시 pancake 출력

똑같은 두 클래스를 담은 Dessert.java라는 파일을 만들었다고 해보자
```java
// 파일명 : Dessert.java

class Utensil {
	static final String NAME = "pot";
}

class Dessert {
	static final String NAME = "pie";
}
```

- `javac Main.java Dessert.java` 명령으로 컴파일 => 컴파일 오류가 나서 Utensil과 Dessert 클래스를 중복 정의 했다고 알려줄 것임
- `javac Main.java` 혹은 `javac Main.java Utensil.java` 명령으로 컴파일 => Dessert.java 파일을 작성하기 전처럼 `pancake`를 출력함
- 하지만, javac Dessert.java Main.java 명령으로 컴파일하면 `potpie`를 출력함. 

컴파일러에 어느 소스 파일을 먼저 건네느냐에 따라 동작이 달라지므로 반드시 잡아야 한다.
-  동작이 달라지는 이유 : 자바 컴파일러는 ‘의존성 기반 컴파일’을 수행한다.
  - (지정된 클래스와 그 클래스가 참조하는 파일들만 컴파일함)

## 해결책
- 정말 간단하다. 톱레벨 클래스들을 서로 다른 소스파일로 분리하면 된다.

### 굳이 여러 톱레벨 클래스를 한 파일에 담고 싶다면..
- 정적 멤버 클래스 사용을 고려
- 위의 코드를 정적 멤버 클래스로 바꿔본 예시이다.
```java
public class Test {
    public static void main(String[] args){
        System.out.println(Utensil.NAME + Dessert.NAME);
    }
    
    private static class Utensil {
        static final String NAME = "pan";
    }
    
    private static class Dessert {
        static final String NAME = "cake";
    }
}
```

# 핵심 정리 및 요약
### 소스 파일 하나에는 반드시 톱레벨 클래스를 하나만 담자.
- 그래야 컴파일 순서에 따라 동작이 달라지는 일이 없을테니!