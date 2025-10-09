```java
private final List<Cheese> chessesInStock = ...;

/**
 * (Qreturn 매장 안의 모든 치즈 목록을 반환한다.
 * 단, 재고가 하나도 없다면 null을 반환한다.
 */
public List<Cheese> getCheeses() {
	return cheesesInStock.isEmpty() ? null
		: new ArrayList<>(cheesesInStock);
}
```

- 만약 위의 코드처럼 null을 반환한다면, 받는 쪽에서 이 null을 처리하는 코드를 따로 만들어 주어야 한다
```java
List<Cheese> cheeses = shop.getCheeses();
if (cheeses != null && cheeses.contains(Cheese.STILTON))
System.out.println("좋았어, 바로 그거야.");
```

- 이처럼 방어하는 코드를 넣어두지 않으면 오류가 발생할 수 있다.
- 반대로 반환하는 쪽에서도 특수 처리를 해야 하기 때문에 코드는 더 복잡해진다.

### 빈 컬렉션

빈 컨테이너를 null대신 반환하면 성능이 저하 된다는 주장은 두 가지로 반박이 가능하다.
1. 빈 컨테이너 할당은 딱히 신경 쓸 정도의 성능 저하가 일어나지 않는다.
2. 빈 컬렉션과 배열은 굳이 새로 할당하지 않고도 반환할 수 있다.

다음 코드는 빈 컬렉션을 반환하는 전형적인 코드이다.
```java
public List getCheeses() {
	return new ArrayList<>(cheesesInStock);
}
```

만약 빈 컬렉션 할당이 확연히 성능 저하를 일으킨다고 한다면, 빈 불변 컬렉션을 반환하자.

Collections에는 emptyList,emptyMap,emptySet 등이 존재한다.
```java
public List getCheeses() {
	return cheesesInStock.isEmpty() ? Collections.emptyList()
		: new ArrayListo(cheesesInStock);
}

```

다만 정말로 성능 저하가 발생함을 확인한 후, 성능이 개선됨을 확인했을 경우에 사용하자.


### 빈 배열

- 배열을 반환할 때도 절대 null 말고 길이가 0인 배열을 반환하자.
- 아래 코드는 길이가 0일수도 있는 배열을 반환하는 방법이다.
```java
public Cheese[] getCheeses() {
	return cheesesInStock.toArray(new Cheese[0]);
}
```
- 여기서 toArray에 집어넣은 배열은 반환 타입을 알려주는 역할을 한다.
- 만약 집어넣는 배열을 새로 생성하는 것이 성능 저하 같다면 아래와 같이 하면 된다.
```java
private static final Cheese[] EMPTY_CHEESE_ARRAY = new Cheese[0];

public Cheese[] getCheeses() {
	return cheesesInStock.toArray(EMPTY_CHEESE_ARRAY);
}
```
- 미리 final 배열을 생성해두고, 필요할 때마다 동일한 배열을 집어넣어주면 된다.
- 만약 0인 경우에만 EMPTY_CHEESE_ARRAY를 그대로 반환한다.

> [! ]
> toArray에 집어 넣은 배열 a가 충분히 크다면 그 안에 원소를 집어넣고, 그렇지 않다면 새로 생성한다. 위의 코드에서는 원소가 0일 때만 EMPTY_CHEESE_ARRAY를 그대로 반환할 것이다.

- 다만 오히려 성능이 떨어진다는 말도 있는 만큼, 성능 개선이 목적이라면 추천하지 않는다

