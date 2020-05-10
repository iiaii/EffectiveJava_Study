# #EffectiveJava/8장_메서드/55옵셔널


## 55. 옵셔널 반환은 신중히 하라

자바 8 전에는 메서드가 특정 조건에서 값을 반환할 수 없을 때 취할 수 있는 선택지가 두가지 있었다. 예외를 던지거나 (반환타입이 객체 참조라면) null을 반환하는 것이다. 두 방법은 모두 허점이 있다. 
-> 예외는 진짜 예외적인 상황에서만 사용해야 함(아이템69)
-> 별도의 null 처리 코드를 추가해야 한다.


자바 8로 올라가면서 또 하나의 선택지가 [ Optional<T> ] 인데, null 이 아닌 T타입 참조를 하나 담거나 혹은 아무것도 담지 않을 수 있다. (아무것도 담지 않은 옵셔널은 ‘비었다’고 말함. 반대로 어떤 값을 담은 옵셔널은 ‘비지 않았다’고 함)

옵셔널은 원소를 최대 1개 가질 수 있는 불변 컬렉션이다. (Optional<T>가 
Collection<T>를 구현하지는 않았지만, 원칙적으로 그렇다는 말임)

보통은 T를 반환해야 하지만 특정 조건에서는 아무것도 반환하지 않아야 할때 T 대신 Optional<T>를 반환하도록 선언하면 된다. 그러면 유효한 반환값이 없을 때는 빈 결과를 반환하는 메서드가 만들어진다. 
-> 옵셔널을 반환하는 메서드는 예외를 던지는 메서드보다 유연하고 사용하기 쉬우며, null을 반환하는 메서드보다 오류 가능성이 작다.

```java
// 컬렉션에서 최댓값을 구한다 (컬렉션이 비었으면 예외를 던짐)
public static <E extends Comparable<E>> E max(Collection<E> c) {
	if(c.isEmpty())
		throw new IllegalArgumentException(“빈 컬렉션”);
	
	E result = null;
	for(E e : c)
		if(result == null || e.compareTo(result) > 0)
			result = Objects.requireNonNull(e);
	return result;
}
```
-> 빈 컬렉션이 오면 IllegalArgumentException을 던진다.

```java
// 컬렉션에서 최댓값을 구해 Optional<E>로 반환
public static <E extends Comparable<E>> Optional<E> max(Collection<E> c) {
	if(c.isEmpty())
		reuturn Optional.empty();

	E result = null;
	for(E e : c)
		if(result == null || e.compareTo(result) > 0)
			result = Objects.requireNonNull(e);
	return Optional.of(result);
}
```

적절한 정적 팩토리를 사용해 옵셔널을 생성해주면 된다 (위 코드에서는 empty()와 of(value)로 생성했다. of()에 null을 넣으면 NullPointerException을 던짐)
null 값도 허용하는 옵셔널을 만들려면 Optional.ofNullable(value)를 사용하면 된다. **옵셔널을 반환하는 메서드에서는 절대 null을 반환하면 안된다**

스트림의 종단 연산 중 상당수가 옵셔널을 반환한다. 앞의 max 메서드를 스트림 버전으로 다시 작성한다면 Stream의 max 연산이 우리에게 필요한 옵셔널을 생성해줄 것이다. (비교자를 명시적으로 전달해야 하지만)

```java
// 컬렉션에서 최댓값을 구해 Optional<E>로 반환한다 - 스트림 버전
public static <E extends Comparable<E>> Optional<E> max(Collection<E> c) {
	return c.stream().max(Comparator.naturalOrder());
}
```

그렇다면 null을 반환하거나 예외를 던지는 대신 옵셔널 반환을 선택해야 하는 기준에서 옵셔널은 검사 예외와 취지가 비슷하다 (아이템71). 즉 반환 값이 없을 수도 있음을 API 사용자에게 명확히 알려준다. 비검사 예외를 던지거나 null을 반환한다면 API 사용자가 그 사실을 인지하지 못해 끔찍한 결과로 이어질 수 있다. 하지만 검사 예외를 던지면 클라이언트에서는 반드시 이에 대처하는 코드를 작성해넣어야 한다.

비슷하게, 메서드가 옵셔널을 반환한다면 클라이언트는 값을 받지 못했을 때 취할 행동을 선택해야 한다. 

```java
// 옵셔널 활용 1 - 기본값 정해둘 수 있다.
String lastWordInLexicon = max(words).orElse(“단어 없음…”);
```

또는 상황에 맞는 예외를 던질 수 있다 (팩토리를 건네는데 예외가 발생하지 않으면 생성비용이 들지 않는다)

```java
// 옵셔널 활용 2 - 원하는 예외를 던질 수 있다.
Toy myToy = max(toys).orElseThrow(TemperTantrumException::new);
```

옵셔널에 항상 값이 채워져 있다고 확신하면 그냥 곧바로 값을 꺼내 사용하는 선택지도 있다. (잘못 판단한 경우 NoSuchElementException발생)

```java
// 옵셔널 활용 3 - 항상 값이 채워져 있다고 가정
Element lastNobleGas = max(Elements.NOBLE_GASES).get();
```



기본 값을 설정하는 비용이 아주 커서 부담이 될 때는 Supplier<T>를 인수로 받는 orElseGet을 사용하면, 값이 처음 필요할 때 Supplier<T>를 사용해 생성하므로 초기 설정 비용을 낮출 수 있다.

filter, map, flatMap, ifPresent를 사용하면 문제를 해결 가능한 경우가 많다.
isPresent 메서드는 안전 밸브 역할의 메서드로 원하는 모든 작업을 수행할 수 있지만, 신중히 사용해야 한다. (isPresent 코드 대부분은 앞의 메서드로 대체 가능하고 그게 더 명확한 코드임)

```java
Optional<ProcessHandle> parentProcess = ph.parent();
System.out.println(“부모 PID: “+(parentProcess.isPresent() ? String.valueOf(parentProcess.get().pid()) : “N/A”));

// 이를 Optional의 map을 사용하여 다듬으면
System.out.println(“부모 PID: “+ ph.parent().map(h -> String.valueOf(h.pid())).orElse(“N/A”));

// 스트림을 사용하면
StreamOfOptionals
	.filter(Optional::isPresent)
	.map(Optional::get)
// 값이 있다면 그 값을 스트림에 매핑한다 라는 의미

// 자바 9에서는 Optional에 stream() 메서드가 추가되었다. 
// 이를 Stream flatMap 메서드와 조합하면
steramOfOptionals
	.flatMap(Optional::stream)
```

반환값으로 옵셔널을 사용한다고 해서 무조건 득이 되는 것이 아니다. 
**컬렉션, 스트림, 배열, 옵셔널 같은 컨테이너 타입은 옵셔널로 감싸면 안된다**
빈 Optional<List<T>> 가 아닌 List<T>를 반환하는 게 좋다. (빈 컨테이너를 반환하면 클라이언트에 옵셔널 처리 코드를 넣지 않아도 됨)

기본적으로 결과가 없을 수 있으며, 클라이언트가 이 상황을 특별하게 처리해야 한다면 Optional<T>를 반환한다. (그래도 엄연히 초기화해야하는 객체이고 값을 꺼내려면 메서드를 호출해야 하기 때문에 성능이 중요한 상황에서는 맞지 않을 수  있다)

박싱된 기본 타입을 담는 옵셔널은 기본 타입 자체보다 무거울 수밖에 없다. (값을 두겹이나 감싸기 때문) 그래서 자바 API에 OptionalInt, OptionalLong, OptionalDouble이 있다. (이런 경우 박싱된 기본타입을 담는 옵셔널을 반환하지 말자)

옵셔널을 맵의 값으로 사용하면 절대 안된다. 만약 그러면 맵 안의 키가 없다는 사실을 나타내는 방법이 두가지가 된다. 하나는 키 자체가 없는 경우, 하나는 키가 속이 빈 옵셔널인 경우이다. -> 옵셔널을 컬렉션의 키, 값, 원소나 배열의 원소로 사용하는게 적절한 상황은 거의 없다!

마지막으로 옵셔널을 인스턴스 필드에 저장해두는 경우는 대부분 필수 필드를 갖는 클래스와 이를 확장해 선택적 필드를 추가한 하위 클래스를 따로 만들어야 함을 암시한다. (적절한 경우도 있다. 기본타입이라 값이 없을을 나타낼 방법이 마땅치 않을 경우)


**::핵심 정리::** 

> 값을 반환하지 못할 가능성이 있고, 호출할 때마다 반환값이 없을 가능성을 염두에 둬야하는 메서드라면 옵셔널을 반환해야 할 상황일 수 있다. 하지만 옵셔널 반환에는 성능 저하가 뒤따르니, 성능에 민감한 메서드라면 null을 반환하거나 예외를 던지는 편이 나을 수 있다. 그리고 옵셔널을 반환값 이외의 용도로 쓰는 경우는 매우 드물다.


