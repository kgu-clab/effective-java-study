# item 34

### int 상수 대신 열거 타입을 사용하라

- 열거 타입
  :일정 개수의 상수 값을 정의한 다음, 그 외의 값은 허용하지 않는 타입

- 자바에서 열거 타입을 지원하기 전
  :정수 상수를 한 묶음 선언하는 ‘정수 열거 패턴’을 사용
  :예제 코드

    ```java
    public static final int APPLE_FUJI = 0;
    public static final int APPLE_PIPPIN = 1;
    public static final int APPLE_GRANNY_SMITH = 2;
    
    public static final int ORANGE_NAVEL = 0;
    public static final int ORANGE_TEMPLE= 1;
    public static final int ORANGE_BLOOD = 2;
    ```

  →정수 열거 패턴의 단점
  1. 타입 안전을 보장할 방법이 없음

  2. 표현력이 좋지 않음
       →오렌지와 사과를 동등 연산자(==)로 비교하더라도 컴파일러는 경고 메시지를 출력하지 않음
  3. 정수 열거 패턴을 위한 별도 이름 공간을 지원하지 않기 때문에 접두어를 써서 이름 충돌을 방지할 수밖에 없음
  4. 정수 열거 패턴을 사용한 프로그램은 깨지기 쉬움
       →상수를 나열한 것뿐이라 컴파일 시 그 값이 클라이언트 파일에 그대로 새겨짐. 따라서 상수의 값이 바뀌면 클라이언트도 반드시 다시 컴파일 해야 함

- 추가 단점⇒정수 상수는 문자열로 출력하기가 까다롭다.
  이유: 디버거 입장에서는 단지 숫자로만 보임→그렇다고 정수 열거 그룹에 속한 모든 상수를 순회하는 것도 쉽지 않음
  해결책 1) 문자열 열거 패턴
  →정수 대신 문자열 상수를 사용함
  →장점: 상수의 의미를 출력할 수 있음
  →단점1: 문자열 상수의 이름 대신 문자열 값을 그대로 하드코딩하게 만듦
  →단점2: 하드코딩한 문자열에 오타가 있어도 컴파일러는 확인할 방법이 없음→런타임 버그가 발생
  →단점3: 문자열 비교에 따른 성능 저하

→결론: 사용하지 말 것

해결책 2) 열거 타입

- 열거타입

```java
public enum Apple { FUJI, PIPPIN, GRANNY_SMITH }
public enum Orange { NAVEL, TEMPLE, BLOOD }
```

→특징 1: 열거 타입 자체는 클래스
→특징 2: 상수 하나당 자신의 인스턴스를 하나씩 만들어 public static final 필드로 공개함

→특징 3: 열거 타입은 밖에서 접근할 수 있는 생성자를 제공하지 않으므로 사실상 final

(따라서 열거 타입 선언으로 만들어진 인스턴스들은 딱 하나씩만 존재함)
(인스턴스가 1개뿐인 타입인 싱글턴은 원소가 하나뿐인 열거타입이라 할 수 있고, 거꾸로 말하면 열거 타입은 싱글턴을 일반화한 형태라고 볼 수 있음)

→장점 1: 컴파일 타임 타입 안전성을 제공함
(다른 타입의 값을 넘기게 될 시 컴파일 오류 발생)
→장점 2: 열거타입에는 각자의 이름 공간이 있어서 이름이 같은 상수도 공존가능
(새로운 상수를 추가하거나 순서를 바꿔도 다시 컴파일하지 않아도 됨)
→장점 3: 열거 타입의 toString 메서드는 출력하기에 적합한 문자열을 내어줌

→장점 4: 열거 타입에는 임의의 메서드나 필드를 추가할 수 있고 임의의 인터페이스를 구현할 수도 있음

- 장점 4→열거 타입에 메서드나 필드를 추가하는 경우
  예시 상황 1: Apple과 Orange 각 과일의 색을 알려주거나 과일 이미지를 반환하는 메서드를 추가할 때 열거 타입에는 어떤 메서드도 추가할 수 있음
  예시 상황 2: 태양계의 여덟 행성에 대한 질량과 반지름을 이용해 표면 중력 계산하는 것과 같은 추상 개념 또한 표현할 수 있음

    ```java
    public enum Planet {
    MERCURY(3.302e+23,2.439e6),
    VENUS (4.869e+24,6.052e6),
    EARTH (5.975e+24, 6.378e6),
    MARS (6.419e+23,3.393e6),
    JUPITER(1.899e+27, 7.149e7),
    SATURN (5.685e+26,6.027e),
    URANUS (8.683e+25,2.556e7),
    NEPTUNE(1.024e+26,2.477e7);
    // 질량（단위: 킬로그램）
    // 반지름（단위: 미터）
    private final double mass;
    private final double radius;
    private final double surfaceGravity; // 표면중력(단위: m / s주2)
    // 중력상수(단위: 73 / kg sA2)
    private static final double G = 6.67300E-11;
    // 생성자
    Planet(double mass, double radius) {
    this.mass = mass;
    this.radius = radius;
    surfaceGravity = G * mass / (radius * radius);
    }
    public double mass() { return mass; }
    public double radius() { return radius; }
    public double surfaceGravity() { return surfaceGravity; }
    public double surfaceWeight(double mass) {
    return mass * surfaceGravity; // F = ma
    }
    }
    ```

  →열거 타입 상수 각각을 특정 데이터와 연결지으려면 생성자에서 데이터를 받아 인스터스 필드에 저장하면 됨

  →열거 타입은 자신 안에 정의된 상수들의 값을 배열에 담아 반환하는 정적 메서드인 values를 제공하는데, 여러 타입 값의 toString 메서드는 상수 이름을 문자열로 반환하므로 println이나 printf로 출력하기 적합

- 열거 타입에서 상수를 하나 제거하게 되면?
  case1) 제거한 상수를 참조하지 않는 클라이언트→아무 영향이 없음
  case2) 제거된 상수를 참조하는 클라이언트
  →클라이언트 프로그램을 다시 컴파일하면⇒’상수 없음’을 소스단계에서 잡게되어 컴파일 오류가 발생하게 됨
  →클라이언트 프로그램을 다시 컴파일 하지않으면⇒바뀐 enum과 참조가 어긋나서 런타임에 예외가 발생하게 됨

- 열거 타입을 선언한 클래스 혹은 그 패키지에서만 유용한 기능은 private이나 package-private 메서드로 구현하라
  →일반 클래스와 마찬가지로 그 기능을 클라이언트에 노출해야 할 합당한 이유가 없다면 private이나 package-private으로 선언하라

- 상수마다 동적이 달라져야 하는 상황에서 열거 타입
  →예시 상황: 사칙 연산에서 연산 종류를 열거 타입으로 선언+ 실제 연산까지 열거 타입 상수가 수행
  코드1) switch문 사용

    ```java
    public enum Operation {
    PLUS, MINUS, TIMES, DIVIDE;
    // 상수가 뜻하는 연산을 수행한다.
    public double apply(double x, double y) {
    switch(this) {
    case PLUS: return x + y;
    case MINUS: return x - y;
    case TIMES: return x * y;
    case DIVIDE: return x / y;
    }
    throw new AssertionError("알 수 없는 연산: " + this);
    }
    }
    ```

  →문제점1) 마지막의 throw 코드를 생략하게 될 시 컴파일조차 되지 않음
  →문제점2) 깨지기 쉬운 코드(새로운 상수를 추가하면 case문 추가해야함, 하지 않을 시 연산 수행하려 할 때 런타임 오류를 내며 프로그램 종료됨)
    
  코드2) 열거 타입을 활용해 상수별 메서드 구현
  : 열거 타입에 apply라는 추상 메서드를 선언하여 각 상수에서 자신에 맞게 재정의하는 방법

    ```java
    public enum Operation {
    PLUS {public doubleapply(double x, double y){return x + y;}},
    MINUS {public doubleapply(double x, double y){return x - y;}},
    TIMES {public doubleapply(double x, double y){return x * y;}},
    DIVIDE{public doubleapply(double x, double y){return x / y;}};
    public abstract double apply(double x, double y);
    }
    ```

  →장점1) apply 메서드가 상수 선언 바로 옆에 붙어 있어 새로운 상수를 추가할 때 apply도 재정의해야 한다는 사실을 잊지 않을 수 있음

  →장점2) apply가 추상 메서드이므로 재정의하지 않았다면 **컴파일 오류**로 알려줌


- 상수별 메서드 구현을 상수별 데이터와 결합
  →예시 상황: Operation의 toString을 재정의해 해당 연산을 뜻하는 기호를 반환하도록 함

    ```java
    public enum Operation {
    PLUS("+") {
    public double apply(double x, double y) { return x + y; }
    },
    MINUS("-") {
    public double apply(double x, double y) { return x - y; }
    },
    TIMES("*") {
    public double apply(double x, double y) { return x * y; }
    },
    DIVIDE("/") {
    public double apply(double x, double y) { return x / y; }
    };
    private final String symbol;
    Operation(String symbol) { this.symbol = symbol; }
    @Override public String toString() { return symbol; }
    public abstract double apply(double x, double y);
    }
    ```

    ```java
    public static void main(String[] args) {
    double x = Double.parseDouble(args[0]);
    double y = Double.parseDouble(args[1]);
    for (Operation op : Operation.values())
    System.out.printf("%f %s %f = %f%n", x,op, y, op.apply(x,y));
    }
    ```

  →실행 결과( 명령줄 인수가 2와 4만 주어도 해당 결과를 얻을 수 있음)

  2.000000 + 4.000000=6.000000
  2.000000 - 4.000000=-2.000000
  2.000000 * 4.000000=8.000000
  2.000000 / 4.000000=0.500000

  →해당 코드를 사용해 연산식을 출력하게 될 시, 계산식  출력을 편하게 만들 수 있음

- 열거 타입의 toString 메서드를 재정의할 때, fromString 메서드도 함께 제공된다는 점 고려
  →fromString이란? toString이 반환하는 문자열을 해당 열거 타입 상수로 변환해줌
  →예제 코드: 모든 열거 타입에서 사용할 수 있도록 구현한 fromString

    ```java
    private static final Map<String, Operation> stringToEnum =
     Stream.Of(values()).collect (
        toMap(Object::toString, e->e));
    // 지정한 문자열에 해당하는 Operation을 （존재한다면） 반환한다.
    public static Optional<Operation> fromString(String symbol){
    return Optional.ofNullable(stringToEnum.get(symbol));
    }
    ```


- 상수별 메서드 구현에는 열거 타입 상수끼리 코드를 공유하기 어렵다.
  →예시 상황:
  급여명세서에서 쓸 요일을 표현하는 열거 타입에서는 직원의 기본 임금과 그날 일한 시간에 따른 일당을 계산해주는 메서드가 있음→주중에 오버타임이 발생하게 될 시 잔업 수당이 주말에는 무조건 주어짐
  →예시 코드1: switch 문 사용

    ```java
    enum PayrollDay {
    MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY,
    SATURDAY, SUNDAY;
    private static final int MINS_PER_SHIFT = 8 * 60;
    int pay(int minutesWorked, int payRate) {
    int basePay = minutesWorked * payRate;
    int overtimePay;
    switch(this) {
    case SATURDAY: case SUNDAY: // 주말
    overtimePay = basePay / 2;
    break;
    default: // 주중
    overtimePay = minutesWorked <= MINS_PER_SHIFT ?
    0 : (minutesWorked - MINS_PER_SHIFT) * payRate / 2;
    }
    return basePay + overtimePay;
    }
    }
    ```

  →간결하기는 하지만, 휴가와 같은 새로운 값을 열거 타입에 추가하게 될 시 case문을 잊지 말고 사용해야 함
    
  예시 코드2) 새로운 상수를 추가할 때 잔업수당 ‘전략’을 선택하도록 하는 것
  : 잔업수당 계산을 private 중첩 열거 타입으로 옮기고 payrollDay 열거 타입의 생성자에서 이 중 적당한 것을 선택하도록 함→ PayrollDay 열거 타입은 잔업수당 계산을 그 전략 열거 타입에 위임하게 되어 switch문이나 상수별 메서드 구현 없이 가능하게 됨

    ```java
    enum PayrollDay {
    MONDAY(WEEKDAY), TUESDAY(WEEKDAY), WEDNESDAY(WEEKDAY),
    THURSDAY(WEEKDAY), FRIDAY(WEEKDAY),
    SATURDAY(WEEKEND), SUNDAY(WEEKEND);
    private final PayType payType;
    PayrollDay(PayType payType) { this.payType = payType; }
    int pay(int minutesWorked, int payRate) {
    return payType.pay(minutesWorked, payRate);
    }
    // 전략 열거 타입
    enum PayType {
    WEEKDAY {
    int overtimePay(int minsWorked, int payRate) {
    return minsWorked <= MINS_PER_SHIFT ? 0 :
    (minsWorked - MINS_PER_SHIFT) * payRate / 2;
    }
    },
    WEEKEND {
    int overtimePay(int minsWorked, int payRate) {
    return minsWorked * payRate / 2;
    }
    };
    abstract int overtimePay(int mins, int payRate);
    private static final int MINS_PER_SHIFT = 8 * 60;
    int pay(int minsWorked, int payRate) {
    int basePay = minsWorked * payRate;
    return basePay + overtimePay(minsWorked, payRate);
    }
    }
    }
    ```


→결론: switch문은 열거 타입의 상수별 동작을 구현하는데 적합하지 않음
→Q)그렇다면 switch문은 열거 타입에서는 필요가 없을까?
→A) 기존 열거 타입에 상수별 동작을 혼합해 넣을 때는 switch문이 좋은 선택이 될 수 있음
ex)Operation 열거 타입이 있을 때, 각 연산의 반대 연산을 반환하는 메서드가 필요할 시

```java
public static Operation inverse(Operation op) {
switch(op) {
case PLUS: return Operation.MINUS;
case MINUS: return Operation.PLUS;
case TIMES: return Operation.DIVIDE;
case DIVIDE: return Operation.TIMES;
default: throw new AssertionError("알 수 없는 연산: " + op);
}
}
```

→switch문을 이용해 원래 열거 타입에 없는 기능을 수행하도록 할 수 있음

- 열거 타입에 대한 최종 정리
  사용 경우: 필요한 원소를 컴파일 타임에 다 알 수 있는 상수 집합이라면 항상 열거 타입을 사용하자
  ex) 태양계 행성, 한 주의 요일, 체스 말
  Q)그렇담 열거 타입에 정의된 상수 개수는 고정적이어야 하는가?
  A)Nope, 고정 불변일 필요가 없음, 열거 타입은 나중에 상수가 추가돼도 바이너리 수준에서 호환되도록 설계됨