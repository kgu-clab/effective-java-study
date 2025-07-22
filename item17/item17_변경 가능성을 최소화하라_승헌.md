# 아이템 17: 변경 가능성을 최소화하라

## 불변 클래스란?

> 생성 후 내부 상태가 절대 바뀌지 않는 클래스

- 예시: `String`, `Integer`, `BigInteger`, `BigDecimal`
- 설계, 디버깅, 테스트가 간편하고 스레드 안전

---

##  왜 써야 할까?

- **버그 감소**: 상태 변경 X
- **스레드 안전**: 동기화 없이 사용 가능
- **보안에 강함**: 외부 영향 차단
- **공유 가능**: 객체를 안전하게 재사용 가능

---

##  불변 클래스를 만드는 5가지 규칙

1. **변경자 메서드 제거**  
   → setter 등 상태 변경 메서드 제공 X

2. **상속 금지**  
   → `final` 클래스 또는 생성자 `private` 처리

3. **모든 필드 `private final`로 선언**

4. **가변 객체를 참조할 경우 방어적 복사 사용**  
   → 외부에서 내부 객체를 수정 못 하게 막기

5. **내부 상태를 외부에 절대 노출하지 않기**

---
## 클래스를 불변으로 만드는 방법
- 객체의 상태를 변경하는 메서드(변경자)를 제공하지 않는다.

- 클래스를 확장할 수 없도록 한다. (상속이 불가능하게 만든다.)

- 모든 생성자를 private 혹은 package-private으로 만들고 정적 팩터리를 제공한다.


- 모든 필드를 final로 선언한다.

- 모든 필드를 private으로 선언한다.

- public final은 추후 API 리팩토링의 유연성을 저해할 수 있기 때문에 주의해야 한다.

- 자신 외에는 내부의 가변 컴포넌트에 접근할 수 없도록 한다.
```java
static final class Complex {
    private final double re;
    private final double im;

    public Complex(double re, double im) {
        this.re = re;
        this.im = im;
    }

    public double realPart() {
        return re;
    }

    public double imaginaryPart() {
        return im;
    }

    public Complex plus(Complex c) {
        return new Complex(re + c.re, im + c.im);
    }

    public Complex minus(Complex c) {
        return new Complex(re - c.re, im - c.im);
    }

    public Complex times(Complex c) {
        return new Complex(re * c.re - im * c.im,
                           re * c.im + im * c.re);
    }

    public Complex dividedBy(Complex c) {
        double tmp = c.re * c.re + c.im * c.im;
        return new Complex((re * c.re + im * c.im) / tmp,
                           (im * c.re - re * c.im) / tmp);
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        Complex complex = (Complex) o;
        return Double.compare(complex.re, re) == 0 && Double.compare(complex.im, im) == 0;
    }

    @Override
    public int hashCode() {
        return Objects.hash(re, im);
    }

    @Override
    public String toString() {
        return "(" + re + " + " + im + "i)";
    }
}
```
위 예는 복소수를 클래스로 구현한 것이다.
접근자 메서드와 사칙연산 메서드가 있다.
사칙연산 메서드는 피연산자인 클래스의 멤버를 건들지 않고, 매번 새로운 메서드를 반환한다.
그래서 메서드 이름으로 add 같은 동사 메서드 대신 plus와 같은 전치사를 사용하였다.
이를 함수형 프로그래밍이라 한다.
이러한 방식을 이용하면, 코드에서 불변이 되는 영역이 늘어나는 장점을 누릴 수 있다.



### 불변이 되면 얻어지는 장점

불변객체는 기본적으로 스레드 안전하여 따로 동기화가 필요 없다.
불변객체는 안심하고 공유할 수 있다.

불변의 장점을 활용하려면 클래스 내부에 많은 상수들을 제공하여 사용하는 것도 좋다.

---
## 불변 클래스 및 불변 객체의 특징

불변 클래스는 자주 사용되는 인스턴스를 캐싱하여 같은 인스턴스를 중복 생성하지 않게 해주는 정적 팩터리를 제공할 수 있다.

정적 팩터리를 통해 캐싱을 추후에 쉽게 덧붙일 수 있다.


불변 클래스는 clone() 메서드나 복사 생성자를 제공하지 않는 것이 좋다.

String의 경우 이전의 잘못된 설계로 복사 생성자를 제공하고 있지만 사용하지 않는 것이 좋다.


불변 객체는 자유롭게 공유도 가능할 뿐더러 내부 데이터 공유도 가능하다.


---
## 불변 클래스의 단점

값이 다르면 반드시 독립된 객체로 만들어야 한다.
보통 약간의 메모리 낭비가 있다.
값이 다를 때마다 새로운 객체를 만들어야 한다.


값의 가짓수가 많다면, 이들을 모두 만드는데 큰 비용을 치러야 한다.
ex) 100만 자리수를 가지고 있는 비트에서 앞의 1자리만 다른 경우, 낭비가 심하다.


## 불변 클래스의 단점 해소

다단계 연산들을 예측하여 기본 기능으로 제공할 수 있다.
클라이언트가 원하는 복잡한 연산을 미리 예측하여 제공하는 것이다.
불변인 String 클래스의 경우 가변 동반 클래스인 StringBuilder 클래스가 있다.

---

# 핵심 정리

1) 게터(getter)가 있다해서 무조건 세터(setter)를 만들진 말자.
2) 클래스는 꼭 필요한 경우가 아니라면 불변이어야 한다.
3) 불변 클래스는 장점이 많으며 단점이라곤 특정 상황에서의 잠재적 성능 저하뿐이다.
4) PhoneNumber와 Complex 같은 단순한 값 객체는 항상 불변으로 만들자.
5) String과 BigInteger처럼 무거운 값 객체도 불변으로 만들 수 있는지 고심해야 한다.
6) 모든 클래스를 불변으로 만들 순 없다. 불변으로 만들 수 없는 클래스라도 변경할 수 있는 부분을 최소한으로 줄이자.
7) 생성자는 불변식 설정이 모두 완료된, 초기화가 완벽히 끝난 상태의 객체를 생성해야 한다. 확실한 이유가 없다면 생성자와 정적 팩터리 외에는 그 어떤 초기화 메서드도 public으로 제공해선 안된다.