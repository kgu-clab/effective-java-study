## item46. 스트림에서는 부작용 없는 함수를 사용하라

- 스트림은 함수형 프로그래밍에 기초한 패러다임이앋
- 스트림이 제공하는 표현력, 속도, 병렬성을 얻으려면 이 패러다임까지 함께 받아들여야 한다.
- 패러다임의 핵심은 일련의 변환이 순수한 함수여야 한다
  - 다른 가변 상태를 참조하지 않고
  - 함수 스스로도 다른 상태를 변경하지 않는다
```java
words.forEach(word -> {
	freq.merge(word.toLowerCase(), 1L, Long::sum);
    })
```
```java
freq = words.collect(
    groupingBy(String::toLowerCase, counting()));
```
- 첫 번째 코드는 외부 상태를 수정하는 람다를 실행한다
  - 연산 결과를 보여주는 일 이상을 하는 것이므로 나쁜 코드이다
- 반면 두 번째 코드는 부작용 없는 함수를 사용한다
- forEach는 종단 연산 중 가장 기능이 적고 덜 스트림 다운 코드이다
  - 따라서 forEach는 스트림 계산 결과를 보고할 때만 사용하고 계산할 때는 쓰지 말자

### 수집기
- 수집기를 사용하면 스트림의 원소를 손쉽게 컬렉션으로 모을 수 있다
- 수집기는 총 3가지로 toList(), toSet(), toCollection()이 있다

```java
List<String> topTen = freq.keySet().stream()
    .sorted(comparing(freq::get).reversed())
    .limit(10)
    .collect(toList());
```
- 가장 흔한 단어가 위로 오도록 역순으로 뽑아 정렬하고 리스트에 담는다
- Collectors의 멤버를 정적 임포트하여 쓰면 스트림 파이프라인 가독성이 좋아진다.

```java
private static final Map<String, Operation> stringToEnum = 
    Stream.of(values()).collect(
        toMap(Object::toString, e -> e));
```
- 스트림 원소를 키에 매핑하는 함수와 값에 매핑하는 함수를 인수로 받는다
- 만약 같은 키를 사용하면 파이프라인이 IllegalStateException을 던진다
- 더 복잡한 형태의 `toMap`이나 `groupingBy`는 충돌을 고려한 병합 함수까지 제공하기도 한

```java
Map<Artist, Album> topHits = albums.collect(
	toMap(Album::artist, a->a, maxBy(comparing(Album::sales))));
```
- 비교자로 maxBy라는 정적 팩터리 메서드를 사용했다
- 복잡해 보이지만 매끄럽게 읽힌다
- "앨범 스트림을 맵으로 바꾸는데, 이 맵은 각 음악가와 그 음악가의 베스트 앨범을 짝지은 것이다"

```java
words.collect(groupingBy(word -> alphabetize(word)));
```
- groupingBy는 원소들을 카테고리별로 모아놓은 맵을 담은 수집기를 반환한다
- 여기에서 alphabetize는 분류 함수로 스트림의 요소를 어떤 기준에 따라 묶을지 결정하게 한다

- 외에도 `partitioningBy`, `mapping`, `filtering,` `flatMapping`, `joining` 등 다양한 수집기가 있다

### 핵심 정리
- 스트림 객체에 건네지는 모든 함수는 부작용이 없어야 한다
- forEach는 계산에는 이용하지 말고 수행한 결과를 보고할 때만 이용하자
- 스트림을 올바로 사용하려면 수집기를 잘 알아야 한다
- 가장 중요한 수집기는 toList(), toSet(), toMap(), groupingBy(), joining()이다