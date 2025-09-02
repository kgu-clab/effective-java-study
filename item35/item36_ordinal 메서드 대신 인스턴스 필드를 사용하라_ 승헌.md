# ordinal 메서드 대신 인스턴스 필드를 사용하라

```java
public enum Ensemble {
SOLO, DUET, TRIO, QUARTET, QUINTET,
SEXTET, SEPTET, OCTET, NONET, DECTET;
public int numberOfMusicians() { return ordinal() + 1; }
}
```

먼저 코드를 보면 Enum형식의 solo, duet, trio, quartet 등등 이 적혀있고 0부터 시작이므로 numberOfMusicians에 의해 1씩 더해줘 각각의 숫자가 의마하는 바를 갖추고 있습니다.

## 여기서 문제점 !

현재는 10까지 밖에 선언이 안되어있지만 83이 필요하다.
그러면 numberOfMusicians를 활용해 값을 매기는 거기 때문에 11번째에 83을 의미하는 것을 끼워 넣으면

83이 아닌 12를 의미하게 된다!

Double_Trio를 넣고싶은데…

Double_Trio = 3 x 2 = 6을 의미하지만 sextet이 살아있고 그걸 한칸씩 밀게되면 뒤에는 모두 의미하는 바와 다른 값들이 들어가게 된다.

---

```java
public enum Ensemble {
SOLO(1), DUET(2), TRI0(3), QUARTET(4), QUINTET(5),
SEXTET(6), SEPTET(7), 0CTET(8), D0UBLE_QUARTET(8),
N0NET(9), DECTET(10), TRIPLE_QUARTET(12);
private final int numberOfMusicians;
Ensemble(int size) { this.numberOfMusicians = size; }
public int numberOfMusicians() { return numberOfMusicians; }
}
```

위에 코드처럼 인스턴스 필드로 값을 저장하면

즉 ordinal 대신, 생성자 매개변수로 직접 값을 저장한다.

Ensemble(int size)

너가 선언할 용어(실제 값) 이 되는거다.

그러면 순서에 상관을 쓰지 않아도 되고, 멀리 있는 값을 따와도 문제 없고, 또한 중복인 값이 들어가도 끄떡없다.

결론

## "순서에 의존하지 말고, 필요한 값을 직접 필드로 들고 있어라."
### 그래야 enum 수정, 확장해도 안전하고 유지보수가 쉬움.
