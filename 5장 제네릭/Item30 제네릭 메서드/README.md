# #EffectiveJava/5장_제네릭/30제네릭메서드


## 30. 이왕이면 제네릭 메서드로 만들라


클래스와 마찬가지로, 메서드도 제네릭으로 만들 수 있다. 매개변수화 타입을 받는 정적 유틸리티 메서드는 보통 제네릭이다. Collections의 알고리즘 메서드 (binarySearch, sort 등)는 모두 제네릭이다.

제네릭 메서드 작성법은 제네릭 타입 작성법과 비슷하다. 

```java
// 로 타입 사용 - 수용 불가! (아이템26)
public static Set union(Set s1, Set s2) {
	Set result = new HashSet(s1);
	result.addAll(s2);
	return result;
}

// 컴파일은 되지만 경고 2개 발생
Union.java:5: warning: [unchecked] unchecked call to
HashSet(Collection<? extends E>) as a member of raw type HashSet
		Set result = new HashSet(s1);

Union.java:6: warning: [unchecked] unchecked call to addAll(Collection<? extends E>) as a member of raw type Set
	result.addAll(s2);
```

경고를 없애려면 이 메서드를 타입 안전하게 만들어야 한다.

메서드 선언에서의 세집합의 원소타입을 타입 매개변수로 명시하고, 메서드 안에서도 이 타입 매개변수만 사용하게 수정하면 된다. (타입 매개변수들을 선언하는) 타입 매개변수 목록은 메서드의 제한자와 반환 타입 사이에 온다. 

다음 코드에서 타입 매개변수 목록은 <E>이고 반환타입은 Set<E>이다. 타입 매개 변수의 명명 규칙은 제네릭 메서드나 제네릭 타입이나 같다.(아이템29,68)

```java
// 제네릭 메서드
public static <E> Set<E> union(Set<E> s1, Set<E> s2) {
	Set<E> result = new HashSet<>(s1);
	result.addAll(s2);
	return result;
}

public static void main(String[] args) {
	Set<String> guys = Set.of(“톰”, “딕”, “해리”);
	Set<String> stooges = Set.of(“래리”, “모에”, “컬리”);
	Set<String> aflCio = union(guys, stooges);
	System.out.println(aflCio);
}

// 결과
// [모에, 톰, 해리, 래리, 컬리, 딕]
```

불변 객체를 여러 타입으로 활용할 수 있게 만들어야 할 때가 있다. 제네릭은 런타임에 타입 정보가 소거(아이템28)되므로 하나의 객체를 어떤 타입으로든 매개변수화 할 수 있다. 하지만 이렇게 하려면 요청한 타입 매개변수에 맞게 매번 그 객체의 타입을 바꿔주는 정적 팩토리를 만들어야 한다. 
이 패턴을 제네릭 싱글턴 팩토리라 하며, Collections.reverseOrder 같은 함수 객체나 Collections.emptySet 같은 컬렉션용으로 사용한다.


항등함수를 담은 클래스를 만든다고 할때 자바 라이브러리의 Function.identity를 사용하면 되지만, 직접 작성하면 

```java
// 제네릭 싱글턴 팩토리 패턴
private static UnaryOperator<Object> IDENTITY_FN = (t) -> t;

@SuppressWarnings(“unchecked”)
public static <T> UnaryOperator<T> identity Function() {
	return (UnaryOperator<T>) IDENTITY_FN;
}
```

IDENTITY_FN 을 UnaryOperator<T> 로 형변환하면 비검사 형변환 경고가 발생한다. T가 어떤 타입이든 UnaryOperator<Object> 는 UnaryOperator<T>가 아니기 때문이다. 하짐나 항등함수란 입력 값을 수정 없이 그대로 반환하는 특별한 함수이므로, T가 어떤 타입이든 UnaryOperator<T>를 사용해도 타입 안전하다.


```java
// 제네릭 싱글턴을 사용하는 예
public static void main(String[] args) {
	String[] strings = {“삼베”,”대마”,”나일론”};
	UnaryOperator<String> sameString = identityFunction();
	for(String s : strings)
		System.out.println(sameString.apply(s));

	Number[] numbers = {1,2.0,3L};
	UnaryOperator<Number> sameNumber = identityFunction();
	for(Number n : numbers)
		System.out.println(sameNumber.apply(n));
}
```

자기 자신이 들어간 표현식을 사용하여 타입 매개변수의 허용 범위를 한정할 수 있다. 바로 재귀적 타입 한정이라는 개념이다. 재귀적 타입 한정은 주로 타입의 자연적 순서를 정하는 Comparable 인터페이스(아이템14)와 함께 쓰인다. 

```java
public interface Comparable<T> {
	int compareTo(T o);
}
```

여기서 타입 매개변수 T는 Comparable<T>를 구현한 타입이 비교할 수 있는 원소의 타입을 정의한다. 실제로 거의 모든 타입은 자신과 같은 타입의 원소와만 비교할 수 있다. String은 Comparable<String>을 구현하고 Integer는 Comparable<Integer>를 구현하는 식이다.

Comparable을 구현한 원소의 컬렉션을 입력받는 메서드들은 주로 그 원소들을 정렬, 검색 하는 식으로 사용된다. 이 기능을 수행하렴녀 컬렉션에 담긴 모든 원소가 상호 비교가 가능해야 한다.

```java
// 재귀적 타입한정을 이용해 상호비교 할 수 있음
public static <E extends Comparable<E>> E max(Collection<E> c);
```

타입 한정인 <E extends Comparable<E>> 는 모든 타입E는 자신과 비교할 수 있다 라고 읽을 수 있다. 상호 비교 가능하다는 뜻을 아주 정확하게 표현했다고 할 수 있다.

```java
// 컬렉션에서 최대값을 반환 - 재귀적 타입 한정 사용
public static <E extends Comparable<E>> E max(Collection<E> c) {
	if(c.isEmpty())
		throw new IllegalArgumentException(“컬렉션이 비어 있습니다.”);

	E result = null;
	for(E e : c)
		if(result == null || e.compareTo(result)>0)
			result = Objects.requireNonNull(e);

	return result;
}
```

> 이 메서드에 빈 컬렉션을 건네면 IllegalArgumentException을 던지는데 Optional<E>를 반환하도록 고치는 편이 나을 것이다 (아이템 55)

재귀적 타입 한정은 훨씬 복잡해질 가능성이 있기는 하지만 잘 안 일어 난다. 


> ::핵심 정리:: 
> 제네릭 타입과 마찬가지로, 클라이언트에서 입력 매개변수와 반환값을 명시적으로 형변환 해야 하는 메서드 보다 제네릭 메서드가 더 안전하며 사용하기도 쉽다. 타입과 마찬가지로, 메서드도 형변환 없이 사용할 수 있는 편이 좋으며, 많은 경우 제네릭 메서드가 되어야 한다. (형변환을 해야하는 기존 메서드 -> 제네릭 메서드) 기존 클라이언트는 그대로 둔 채 새로운 사용자의 삶을 훨씬 편하게 만들어 줄 것이다.


