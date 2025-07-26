## item19. 상속을 고려해 설계하고 문서화하라. 그러지 않았다면 상속을 금지하라

### 상속을 고려한 설계와 문서화란 무얼 뜻할까?
- 메서드를 재정의하면 어떤 일이 일어나는 지를 정리한 것
- == 상속용 클래스는 재정의할 수 있는 메서드들을 내부적으로 어떻게 이용하는지 문서로 남겨야함.
1. 내부적인 동작 원리
2. 어떤 순서로 호출하는지
3. 각각의 호출 결과가 이어지는 처리에 어떤 영향을 주는지

### 재정의 가능 이란?
- public과 protected 메서드 중 final이 아닌 모든 메서드

### Implementation Requirements를 이용한 문서화
- 종종 API 문서에서 Implementation Requirements로 시작하는 절을 볼 수 있는데, 그 메서드의 내부 동작 방식을 설명하는 곳이다.
```java
    /**
     * {@inheritDoc}
     *
     * @implSpec
     * This implementation iterates over the collection looking for the
     * specified element.  If it finds the element, it removes the element
     * from the collection using the iterator's remove method.
     *
     * <p>Note that this implementation throws an
     * {@code UnsupportedOperationException} if the iterator returned by this
     * collection's iterator method does not implement the {@code remove}
     * method and this collection contains the specified object.
     *
     * @throws UnsupportedOperationException {@inheritDoc}
     * @throws ClassCastException            {@inheritDoc}
     * @throws NullPointerException          {@inheritDoc}
     */
    public boolean remove(Object o) {
        Iterator<E> it = iterator();
        if (o==null) {
            while (it.hasNext()) {
                if (it.next()==null) {
                    it.remove();
                    return true;
                }
            }
        } else {
            while (it.hasNext()) {
                if (o.equals(it.next())) {
                    it.remove();
                    return true;
                }
            }
        }
        return false;
    }
```
- 위 설명에서는 iterator 메서드를 재정의 하면 remove 메서드의 동작에 영향을 줌을 알 수 있다
- iterator 메서드로 얻는 반복자의 동작이 remove 메서드의 동작에 주는 영향도 정확히 설명했다

### 좋은 API 문서란?
- 클래스를 안전하게 상속할 수 있도록 내부 구현 방식을 설명해야만 한다
- 단 상속이 아니었다면 내부 구현 방식에 대해 기술하지 않아도 됨
- 또한 클래스의 내부 동작 과정 중간에 끼어들 수 있는 훅(hook)을 잘 선별하여 protected 메서드 형태로 공개해야 할 수도 있다.
```java
    /**
     * Removes from this list all of the elements whose index is between
     * {@code fromIndex}, inclusive, and {@code toIndex}, exclusive.
     * Shifts any succeeding elements to the left (reduces their index).
     * This call shortens the list by {@code (toIndex - fromIndex)} elements.
     * (If {@code toIndex==fromIndex}, this operation has no effect.)
     *
     * <p>This method is called by the {@code clear} operation on this list
     * and its subLists.  Overriding this method to take advantage of
     * the internals of the list implementation can <i>substantially</i>
     * improve the performance of the {@code clear} operation on this list
     * and its subLists.
     *
     * @implSpec
     * This implementation gets a list iterator positioned before
     * {@code fromIndex}, and repeatedly calls {@code ListIterator.next}
     * followed by {@code ListIterator.remove} until the entire range has
     * been removed.  <b>Note: if {@code ListIterator.remove} requires linear
     * time, this implementation requires quadratic time.</b>
     *
     * @param fromIndex index of first element to be removed
     * @param toIndex index after last element to be removed
     */
    protected void removeRange(int fromIndex, int toIndex) {
        ListIterator<E> it = listIterator(fromIndex);
        for (int i=0, n=toIndex-fromIndex; i<n; i++) {
            it.next();
            it.remove();
        }
    }
```
- 위 문서에 따르면, removeRange 메서드는 clear 메서드가 실행될 때 호출된다.
- 만약 성능을 개선 하고 싶다면 removeRange 오버라이드하여, 갈아낄 수 있다.
- 즉, 성능 개선 여지를 위해 서브클래스에서 오버라이드 할 수 있도록 protected 형태로 열어둔 형태

### protected 로 노출해야 할지는 어떻게 결정할까?
- 마법은 없다
- 실제 하위 클래스를 만들어 시험해보는 것이 최선이다
- protected 메서드 하나하나가 내부 구현에 해당하므로 그 수는 가능한 적어야 한다
  - 너무 많이 열어두면 부모 클래스가 예상치 못한 방식으로 동작할 수 있기 떄문
- 한편으로는 너무 적게 노출해서 상속의 이점을 없애지 말자
  - 만약 바꿀게 없다면, 애초에 상속 안하고 새로 만드는게 나음
- 상속용으로 설계한 클래스는 배포 전에 반드시 하위 클래스를 만들어 검증해야한다

### 상속을 허용하는 클래스가 지켜야할 몇가지 추가 제약
- 상속용 클래스의 생성자는 직접적으로든 간접적으로든 재정의 가능 메서드를 호출해서는 안 된다.
  - 자식이 생성되기도 전에, 자식의 오버라이드 된 메서드가 호출되기 때문
- clone과 readObject 모두 직접적으로든 간접적으로든 재정의 가능 메서드를 호출해서는 안 된다.
  - 각각 복제하기 전에, 역직렬화 하기 전에 오버라이드 메서드를 호출하게 되므로 예상치 못한 에러를 발생

### 일반적인 구체 클래스는 어떨까?
- 가장 좋은 방법은 상속용으로 설계하지 않은 클래스는 상속을 금지하는 것이다.
1. final로 선언하거나
2. 모든 생성자를 private이나 package-private으로 선언하고 public 정적팩토리메서드를 만들어주는 것

### 핵심 정리
- 클래스 내부에서 스스로를 어떻게 사용하는지 문서로 남기자
- 호율 좋은 하위 클래스를 만들 수 있도록 일부 메서드를 protected로 제공하자
- 확장할 이유가 명확하지 않으면 상속을 금지하자

