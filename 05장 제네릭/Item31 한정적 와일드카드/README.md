# #EffectiveJava/5장_제네릭/31한정적와일드카드

## 31. 한정적 와일드카드를 사용해 API 유연성을 높이라

매개변수화 타입은 불공변이다.(아이템28) 
-> 서로 다른 타입 Type1과 Type2가 있을 때 List<Type1>은 List<Type2>의 하위 타입도 상위 타입도 아님

예를 들어 List<Object>에는 어떤 객체든 넣을 수 있지만, List<String>에는 문자열만 넣을 수 있다. List<String>은 List<Object>가 하는 일을 제대로 수행하지 못하므로 하위 타입이 될 수 없다. (리스코프 치환 원칙 어긋남, 아이템10)


때로는 불공변 방식보다 유연한 무언가가 필요하다.

```java
// Stack 의 public API 추림
public class Stack<E> {
	public Stack();
	public void push(E e);
	public E pop();
	public boolean isEmpty();
}

// 여기에서 일련의 원소를 스택에 넣는 메서드를 추가
// 와일드카드 타입을 사용하지 않은 pushAll메서드 - 결함이 있다!
public void pushAll(Iterable<E> src) {
	for(E e : src)
		push(e);
}
```

위 메서드는 컴파일은 되지만, 완벽히 작동하지 않는다. Iterable src의 원소 타입이 스택의 원소타입과 일치하면 잘 작동하지만, 하위 혹은 상위 타입 관계에서는 다음과 같은 오류가 발생한다.

```java
// Number의 하위 타입 Integer
Stack<Number> numberStack = new Stack<>();
Iterable<Integer> integers = …;
numberStack.pushAll(integer);

// 에러 발생
StackTest.java:7: error: incompatible types: Iterable<Integer> cannot be converted to Iterable<Number>
		numberStack.pushAll(integers);
```

자바는 이런 상황을 위한 한정정 와일드카드 타입이라는 특별한 매개변수화 타입을 지원한다. pushAll의 입력 매개변수 타입은 E의 Iterable이 아니라 E의 하위타입의 Iterable이어야 한다. -> Iterable<? extends E> (하위 타입, 자기자신 포함)

이전 메서드를 수정하면
```java
// E 생산자(producer) 매개변수에 와일드카드 타입 적용
public void pushAll(Iterable<? extends E> src) {
	for(E e : src)
		push(e);
}
```


마찬가지로 popAll 메서드도 결함이 있는데
```java
// 와일드카드 타입을 사용하지 않은 popAll메서드 - 결함이 있다!
public void popAll(Collection<E> dst) {
	while(!isEmpty())
		dst.add(pop());
}

// 컴파일하면 문제 발생
Stack<Number> numberStack = new Stack<>();
Collection<Object> objects = …;
numberStack.popAll(objects);
```

popAll의 입력 매개변수의 타입이 E의 Collection이 아니라 E의 상위타입의 Collection이어야 한다. -> Collection<? super E>

```java
// E 소비자(consumer) 매개변수에 와일드카드 타입 적용
public void popAll(Collection<? super E> dst) {
	while(!isEmpty())
		dst.add(pop());
}
```

유연성을 극대화하려면 원소의 생산자나 소비자용 입력 매개변수에 와일드카드 타입을 사용해야 한다. 한편 입력 매개변수가 생산자와 소비자 역할을 동시에 한다면 와일카드 타입을 써도 좋을 게 없다. (타입을 정확히 지정해야 하는 상황으로 쓰면 안됨)

> 펙스(PECS) : producer-extends, consumer-super (이때 사용)

즉, 매객변수화 타입 T가 생산자라면 <? extends T>를 사용하고, 소비자라면 <? super T>를 사용해야 한다. 

생산자 - 입력 매개변수로부터 이 컬렉션으로 원소를 옮겨 담는다
소비자 - 컬렉션 인스턴스의 원소를 입력 매개변수로 옮겨 담는다

```java
// T 생산자 매개변수에 와일드카드 타입 적용
public Chooser(Collections<? extends T> choices)

public static <E> Set<E> union(Set<? extends E> s1, Set<? extends E> s2)

// 사용
Set<Integer> integers = Set.of(1, 3, 5);
Set<Double> doubles = Set.of(2.0, 4.0, 6.0);
Set<Number> numbers = union(integers, doubles);
```

> 반환타입에는 한정적 와일드카드 타입을 사용하면 안된다. 유연성을 높이는 것이 아니라 클라이언트 코드에서도 와일드카드 타입을 써야하기 때문.

제대로만 사용한다면 클래스 사용자는 와일드카드 타입이 쓰였다는 사실을 의식하지 못한다. 클래스 사용자가 와일드카드 타입을 신경 써야 한다면 그 API에 무슨 문제가 있을 가능성이 크다.

자바7은 타입 추론 능력이 가능하지 못해서 에러가 나지만 명시해주면 해결 가능하다.
`Set<Number> numbers = Union.<Number>union(integers, doubles);`

> 매개변수와 인수의 차이, 매개변수는 메서드 선언에 정의한 변수이고, 인수는 메서드 호출 시 넘기는 실젯값이다. 
> `void add(int value) { … } / add(10)`
> 이 코드에서 value는 매개변수이고 10은 인수이다
> 이 정의를 제네릭까지 확장하면
> `class Set<T> { … } / Set<Integer> = …;`
> T는 타입 매개변수이고, Integer는 타입인수가 된다.


public static <E extends Comparable<E>> E max(List<E> list)
-> 
public static <E extends Comparable<? super E>> E max(List<? extends E> list)

Compareable은 언제나 소비자이므로 일반적으로 Comparable<E> 보다는 Comparable<? super E>를 사용하는 것이 좋다 (Comparator도 마찬가지)


비한정적 와일드카드 (List<?>꼴)에서
메서드 선언에 타입 매개변수가 한번만 나오면 와일드 카드로 대체하는 것이 좋지만 List<?>에는 null 외에는 어떤 값도 넣을 수 없다. 

```java
// 에러발생 - 방금 꺼낸 원소를 리스트에 다시 넣을 수 없음
public static void swap(List<?> list, int i, int j) {
	list.set(i, list.set(j, list.get(i));
}

// private 도우미 메서드로 해결
public static void swap(List<?> list, int i, int j) {
	swapHelper(list, i, j);
}

// 와일드카드 타입을 실제 타입으로 바꿔주는 private 도우미 메서드
private static <E> void swapHelper(List<E> list, int i, int j) {
	list.set(i, list.set(j, list.get(i)));
}
```

도우미 메서드 시그니처는 public API로 쓰기에 너무 복잡해서 
`public static <E> void swap(List<E> list, int i, int j);`
과 같은 형태로 쓰인다


::핵심 정리:: 

> 조금 복잡하더라도 와일드카드 타입을 적용하면 API가 유연해진다. 널리 쓰일 라이브러리를 작성한다면 반드시 와일드카드 타입을 적절히 사용해야 한다. PECS 공식을 기억하자. 생산자는 extends를 소비자는 super를 사용한다. Comparable, Comparator는 모두 소비자라는 사실도 기억하자.


