## item2. 생성자에 매개변수가 많다면 빌더를 고려하라

- 정적 팩터리와 생성자에는 똑같은 제약이 있다
- **선택적 매개변수**가 많을 때 대응이 어렵다는 것

- 아래 NutritionFacts 클래스는 6개의 매개변수를 가지고 있고, 그 중 4개의 선택적 매개변수를 가지고 있다
- 아래와 같이 생성자를 늘려가는 방식을 **점층적 생성자 패턴**이라 한다
- 점층적 생성자 패턴을 쓸 수는 있지만, 매개 변수의 개수가 많아지면 코드를 작성하거나 읽기가 어렵다
- 매개변수가 더 많아지면 금세 걷잡을 수 없이 복잡해질 것
```java
public class NutritionFacts {
    private final int servingSize; // 필수
    private final int servings; // 필수
    private final int calories; // 선택
    private final int fat; // 선택
    private final int sodium; // 선택
    private final int carbohydrate; // 선택

    public NutritionFacts(int servingSize, int servings) {
        this(servingSize, servings, 0);
    }

    public NutritionFacts(int servingSize, int servings, int calories) {
        this(servingSize, servings, calories, 0);
    }

    public NutritionFacts(int servingSize, int servings, int calories, int fat) {
        this(servingSize, servings, calories, fat, 0);
    }

    public NutritionFacts(int servingSize, int servings, int calories, int fat, int sodium) {
        this(servingSize, servings, calories, fat, sodium, 0);
    }

    public NutritionFacts(int servingSize, int servings, int calories, int fat, int sodium,
                          int carbohydrate) {
        this.servingSize = servingSize;
        this.servings = servings;
        this.calories = calories;
        this.fat = fat;
        this.sodium = sodium;
        this.carbohydrate = carbohydrate;
    }
}
```

- 생성자 대신 활용할 수 있는 방안은 **자바 빈즈** 패턴이다
- 매개변수가 없는 생성자로 객체를 만들고, setter를 호출 해 값을 설정한다
```java
public class NutritionFacts {
    private int servingSize = -1; 
    private int servings = -1;
    private int calories = 0;
    private int fat = 0;
    private int sodium = 0;
    private int carbohydrate = 0;

    public NutritionFacts() {
        // 매개변수가 없는 생성자
    }

    public NutritionFacts setServingSize(int servingSize) {
        this.servingSize = servingSize;
        return this;
    }

    public NutritionFacts setServings(int servings) {
        this.servings = servings;
        return this;
    }

    public NutritionFacts setCalories(int calories) {
        this.calories = calories;
        return this;
    }

    public NutritionFacts setFat(int fat) {
        this.fat = fat;
        return this;
    }

    public NutritionFacts setSodium(int sodium) {
        this.sodium = sodium;
        return this;
    }

    public NutritionFacts setCarbohydrate(int carbohydrate) {
        this.carbohydrate = carbohydrate;
        return this;
    }
}
```

- 자바 빈즈 패턴은 심각한 단점을 지니고 있다
- 객체 하나를 만들기 위해 메서드를 여러개 호출해야하고
- 객체가 완전히 생성되기 전까지는 일관성이 무너진 상태이다
- 런타임 버그를 유발할 수 있다

```java
public class Main {
    public static void main(String[] args) {
        NutritionFacts cocaCola = new NutritionFacts(); // 일관성(Consistency)이 깨진 상태 시작
        cocaCola.setServingSize(240);
        cocaCola.setServings(8);
		cocaCola.setCalories(100);		
		cocaCola.setSodium(35);
		cocaCola.setCarbohydrate(27); // 일관성(Consistency)이 깨진 상태 끝
    }
}
```

- **빌더 패턴**은 점층적 생성자 패턴의 안전성과 자바 빈즈 패턴의 가독성을 겸비한 대안이다
  1. 필수 매개변수만으로 생성자(혹은 정적 팩터리)를 호출해 빌더 객체를 얻음
  2. 빌더 객체에 선택적 매개변수를 설정하는 메서드를 호출 (일종의 세터)
  3. 빌더 객체의 build() 메서드를 호출해 최종 객체를 생성

```java
public class NutritionFacts {
    private final int servingSize; // 필수
    private final int servings; // 필수
    private final int calories; // 선택
    private final int fat; // 선택
    private final int sodium; // 선택
    private final int carbohydrate; // 선택

    public static class Builder { // 빌더 클래스 생성
        // 필수 매개변수
        private final int servingSize;
        private final int servings;

        // 선택 매개변수
        private int calories = 0;
        private int fat = 0;
        private int sodium = 0;
        private int carbohydrate = 0;

        public Builder(int servingSize, int servings) { // 필수 매개변수만으로 생성자(혹은 정적 팩터리)를 호출해 빌더 객체를 얻음
            this.servingSize = servingSize;
            this.servings = servings;
        }

        public Builder calories(int val) { // 빌더 객체에 선택적 매개변수를 설정하는 메서드를 호출 (일종의 세터)
            calories = val;
            return this;
        }

        public Builder fat(int val) {
            fat = val;
            return this;
        }

        public Builder sodium(int val) {
            sodium = val;
            return this;
        }

        public Builder carbohydrate(int val) {
            carbohydrate = val;
            return this;
        }

        public NutritionFacts build() { // 빌더 객체의 build() 메서드를 호출해 최종 객체를 생성
            return new NutritionFacts(this);
        }
    }

    private NutritionFacts(Builder builder) {
        servingSize = builder.servingSize;
        servings = builder.servings;
        calories = builder.calories;
        fat = builder.fat;
        sodium = builder.sodium;
        carbohydrate = builder.carbohydrate;
    }
}

public class Main {
	public static void main(String[] args) {
		NutritionFacts cocaCola = new NutritionFacts.Builder(240, 8) // 필수 매개변수로 빌더 객체 생성
                .calories(100) // 선택 매개변수 설정
                .sodium(35)
                .carbohydrate(27)
                .build(); // 빌더 객체의 build() 메서드를 호출해 최종 객체 생성
	}
}
```

- 빌더 패턴은 계층적으로 설계된 클래스와 함께 쓰기에 좋다
- 해당 내용은 아직 어려울 수 있으니 넘어가도록 하자

### SpringBoot에서 Lombok을 활용한 빌더 패턴
- SpringBoot에서는 아래와 같이 빌더 패턴을 활용한다
- Lombok 라이브러리를 활용해 빌더 패턴을 간편하게 구현할 수 있다
```java
public static User create(String id, String password, String name, String email, String phone, Major major, 
        PasswordEncoder passwordEncoder) {
    validateDept(id, email);
    return User.builder()
        .id(id)
        .password(encodePassword(password, passwordEncoder))
        .name(name)
        .email(email)
        .phone(phone)
        .role(USER)
        .major(major)
        .build();
}
```

### 핵심 정리
- 생성자나 정적 팩터리가 처리해야 할 매개변수가 많다면 빌더 패턴을 선택하는 게 더 낫다
- 매개변수 중 다수가 필수가 아니거나 같은 타입이면 특히 더 그렇다
- 빌더는 점층적 생성자보다 클라이언트 코드를 읽고 쓰기가 훨씬 간결하고, 자바빈즈보다 훨씬 안전하다
