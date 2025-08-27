## 개념

제네릭은 Set\<E>, Map<K,V> 등의 컬렉션과 ThreadLocal\<V>, AtomicReference\<T>등의 단일 원소 컨테이너에도 흔히 쓰인다. 이 때 매개변수화 되는 대상은 컨테이너 자신이기 때문에, 하나의 컨테이너에 넣을 수 있는 타입의 수가 제한된다.

하지만 더 유연하면서도 동시에 타입안전성을 챙겨야 할 수도 있다.이 때 컨테이너 대신에 키를 매개변수화 한 후, 컨테이너에 값을 넣거나 뺄 때 매개변수화한 키를 제공해 주면 된다. 이를 타입 안전 이종 컨테이너 패턴(type safe heterogeneous container pattern)이라고 한다.

이때 class의 클래스는 제네릭이기 때문에, 각 타입의 Class 객체를 매개변수화 한 키 역할로 사용할 수 있다.

## 예시

원하는 인스턴스를 마음대로 저장하고 불러올 수 있는 아래의 클래스를 보자

```java
public class Favorites {
	private Map<Class<?>, Object> favorites = new HashMap<>();
	
	public <T> void putFavorite(Class<?> type, T instance) {
		favorites.put(Objects.requireNonNull(type), instance);
	}
	
	public <T> T getFavorite(Class type) {
		return type.cast(favorites.get(type));
	};
}

public static void main(String[] args) {
	Favorites f = new Favorites();
	f.put Favorite(String.class, "Java");
	f.putFavorite(Integer.class, Oxcafebabe);
	f.putFavorite(Class.class, Favorites.class);
	
	String favorite String = f.getFavorite(String.class);
	int favoriteinteger = f.getFavorite(Integer.class);
	Class<?> favoriteClass = f.getFavorite(Class.class);
	
	System.out.printf("%s %x %s%n", favoritestring, favoriteinteger, favoriteClass.getName());

}

```
즐겨찾는 String, Integer, Class 인스턴스를 저장, 검색, 출력하고 있다.
- Faborites 인스턴스는 타입 안전하여, 다른 타입의 인스턴스를 반환할 일이 없다.
- 동시에 기존 Map과 달리 수많은 종류의 클래스를 담아낼 수 있다.
- 따라서 해당 클래스는 타입 안전 이종 컨테이너이다.

위의 클래스는 Map이 아닌, Key가 와일드 카드 타입을 담고 있다. 따라서 Class\<Integer>와 Class\<String>이 같은 인스턴스에 담길 수 있게 된다.

putFavorite은 주어진 Class 객체와 인스턴스를 favorites에 추가하면 끝이다. 이 때 키와 값 사이의 타입 링크 정보는 사라지지만 , getFavorite에서 관계를 되살릴 수 있다.

getFavorite에서는 Class의 cast 메서드를 통해 Class 객체가 가리키는 타입으로 동적 형변환한다. cast는 Class객체가 알려주는 타입의 인스턴스인지를 먼저 확인한 다음 반환하거나 예외를 던진다.

```java
public class Class { T cast(Object obj); }
```

cast는 Class객체의 타입 매개변수로 반환하기 때문에, getFavorite에서는 비검사 형변환을 하지 않고 타입 안전하게 사용할 수 있음이 보장된다.

## 제약

위에서 예시로 든 Favorites 클래스에는 두가지 제약이 있다.

1. Class 객체를 제네릭이 아닌 로타입으로 넘기면 타입 안전성이 쉽게 깨진다.
	- 이를 어기지 않기 위해서는 아래와 같이 동적 형변환을 통해 인스턴스의 타입이 명시한 타입과 동일한지 확인하면 된다.
	- 컬렉션에는 이 방식을 적용한 컬렉션 래퍼들이 존재한다.(checkedSet/List/Map등)
```java
public void putFavorite(Class type, T instance) {
	favorites.put(Objects.requireNonNull(type), type.cast(instance));
}
```

2. 실체화 불가 타입에는 사용할 수 없다.
	- 예를 들어 String, String\[]과 달리 List\<String> 등은 사용할 수 없다.
	- 왜냐하면 List\<String>의 Class 객체는 얻어올 수 없기 때문이다.
	- 이는 완벽한 우회 방법은 존재하지 않는다.

#### 슈퍼 타입 토큰

슈퍼 타입 토큰은, 익명 클래스의 제네릭 타입 매개변수는 리플렉션을 통해 매개변수 정보를 얻어올 수 있다는 점을 이용해 만든 타입 토큰이라고 볼 수 있다.
```java
Favorites f = new Favorites();

List pets = Arrays.asList ("개", "고양이", "앵무");
f.putFavorite(new TypeRef<List<String>>(){}, pets);
listofStrings = f.getFavorite(new
TypeRef<List<String>>(){});
```
- 위의 코드에서는 TypeRef를 상속하는 익명의 하위 클래스를 생성한 후 Favorites 객체에 넣는다.



### 한정적 타입 토큰

Favorites 클래스의 타입 토큰은 비한정적이기에, 어떤 Class 객체든 받아들인다. 이를 위해서는 한정적 타입 토큰을 사용할 수 있는데, 한정적 타입 매개변수나 한정적 와일드카드를 사용해 타입을 제한하는 타입 토큰이다.

애너테이션 API는 한정적 타입 토큰을 적극적으로 사용하는데, 아래 예시는 그 중 일부이다.

```java
public <T extends Annotation>
	T getAnnotation(Class<T> annotationType);
```
- 다음은 AnotatedElement 인터페이스에 선언된 메서드이며, 대상 요소에 달린 애너테이션을 읽어오는 역할을 한다.
- 여기서는 애너테이션이 붙은 클래스 객체가 하나의 타입 안전 이종 컨테이너인 것이고, 애너테이션 타입이 키라고 볼 수 있다.


만약 Class\<T> 타입의 객체를 한정적 타입 토큰을 받는 메서드에 넘기려고 할 때, asSubclass를 사용한다면 컴파일 타임에 비검사 형변환 경고 없이 변환이 가능하다.
```java
static Annotation getAnnotation(AnnotatedElement element,
	String annotationTypeName) { 
	
	Class annotationType = null; // 비한정적 타입 토큰
	try {
		annotationType = Class.forName(annotationTypeName);
	} catch (Exception ex) { 
		throw new IllegalArgumentException(ex);
	}
	return element.getAnnotation(
		annotationType.asSubclass(Annotation.class));
}
```
- 맨 아래 줄에서 getAnnotation에 asSubclass를 사용해 비한정적 타입 토큰을 형변환 하는 모습을 볼 수 있다.
- 만약 형변환에 실패하면 ClassCastException을 던진다.

### 정리

> 일반적인 제네릭 타입에서는 하나의 컨테이너가 다룰 수 있는 타입이 제한적이지만, 키를 타입 매개변수로 설정한다면 제한이 없는 타입 안전 이종 컨테이너를 만들 수 있다. 이 컨테이는 Class를 키로 사용하며, 이때 Class 객체를 타입 토큰이라고 한다.