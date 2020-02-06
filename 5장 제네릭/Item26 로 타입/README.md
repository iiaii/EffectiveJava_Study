# #EffectiveJava/5장_제네릭/26로타입

제네릭은 자바5 부터 사용가능. 제네릭 이전에는 컬렉션에서 객체를 꺼낼 때마다 형변환을 해야 했다. (제네릭을 사용하면 컬렉션이 담을 수 있는 타입을 컴파일러에 알려주게 되어 알아서 형변환 코드를 추가한다) 엉뚱한 타입의 객체를 넣으려는 시도를 컴파일 과정에서 차단하여 더 안전하고 명확한 프로그램을 만들어 준다. 하지만 코드가 복잡해지는 원인이 되기도 한다.


## 26. 로 타입은 사용하지 말자 (raw type)

클래스와 인터페이스 선언에 타입 매개변수가 쓰이면, 제네릭 클래스 혹은 제네릭 인터페이스라하고 이를 통틀어 제네릭 타입이라 한다. (List 인터페이스는 원소의 타입을 나타내는 타입 매개변수 E를 받는다 이 인터페이스의 완전한 이름은 List<E>지만 List라고도 쓴다) 

각각의 제네릭 타입은 일련의 매개변수화 타입을 정의한다. 먼저 클래스 이름이 나오고 이어서 꺽쇠괄호 안에 실제 타입 매개변수들을 나열한다. (List<String>은 원소의 타입이 String인 리스트를 뜻하는 매개변수화 타입이다. 여기서 String이 정규타입 매개변수 E에 해당하는 실제 타입 매개변수다)

제네릭 타입을 하나 정의하면 그에 딸린 로 타입(raw type)도 함께 정의된다. 로 타입이란 제네릭 타입에서 타입 매개변수를 전혀 사용하지 않을 때를 말한다. (List<E>의 로 타입은 List. 로 타입은 선언에서 제네릭 타입 정보가 전부 지워진 것 처럼 동작하는데, 제네릭이 도래하기 전 코드와 호환되도록 하기 위한 궁여지책임)

```java
// 컬렉션의 로 타입 - 따라 하지 말 것! (자바9에서도 동작은 함)
private final Collection stamps = …;

// 실수로 동전을 넣는다
stamps.add(new Coin(…)); // “unchecked call” 경고를 내뱉는다
```

이 코드를 사용하면 실수로 다른 객체가 들어가도 아무 오류 없이 컴파일 되고 실행된다. (컴파일러가 모호한 경고 메시지를 보여주긴 함) 컬렉션에서 Coin을 꺼내기 전에는 오류를 알아채지 못한다.

```java
// 반복자의 로 타입 - 따라 하지 말 것!
for(Iterator i = stamps.iterator(); i.hasNext(); ) {
	Stamp stamp = (Stamp) i.next(); // ClassCastException을 던짐
	stamp.cancel();
}
```

(책에서 계속 이야기하듯) 오류는 가능한 한 발생 즉시, 이상적으로는 컴파일할 때 발견하는 것이 좋다. 이 예에서는 오류가 발생하고 한참 뒤인 런타임에야 알아챌 수 있는데, 이렇게 되면 런타임에 문제를 겪는 코드와 원인을 제공한 코드가 물리적으로 상당히 떨어져 있을 가능성이 커진다. ClassCastException이 발생하면 stamps에 동전을 넣은 지점을 찾기 위해 코드 전체를 훑어봐야 할 수도 있다. “// Stamp 인스턴스만 취급한다” 는 주석은 컴파일러가 이해하지 못하므로 별 도움이 되지 못한다.
-> 제네릭을 활용하면 이 정보가 주성이 아닌 타입 선언 자체에 녹아든다.

```java
// 매개변수화된 컬렉션 타입 - 타입 안정성 확보!
private final Collection<Stamp> stamps = …;
```

이렇게 선언하면 컴파일러는 stamps에는 Stamp의 인스턴스만 넣어야 함을 컴파일러가 인지하게 된다. 따라서 아무런 경고 없이 컴파일된다면 의도대로 동작할 것임을 보장한다. (물론 컴파일러 경고를 숨기지 않았어야 함(아이템27)) 
-> 이제는 stamps 에 엉뚱한 타입의 인스턴스를 넣으려 하면 컴파일 오류가 발생하며 무엇이 잘못됐는지 정확히 알려준다

```
Test.java:9: error: incompatible types: Coin cannot be converted to Stamp
	stamps.add(new Coin());
```

컴파일러는 컬렉션에서 원소를 꺼내는 모든 곳에 보이지 않는 형변환을 추가하여 절대 실패하지 않음을 보장한다. (이번에도 컴파일러 경고가 나지 않았고 경고를 숨기지도 않았다고 가정했다) -> 현업에서 이와 같은 어이없는 실수가 일어나곤 함

앞에서 이야기 했듯 로 타입(타입 매개변수가 없는 제네릭 타입)을 쓰는 걸 언어 차원에서 막아 놓지는 않았지만 절대로 써서는 안 된다. 로 타입을 쓰면 제네릭이 안겨주는 안정성과 표현력을 모두 잃게 된다. (로 타입은 호환성 때문에 있는 것임)

제네릭 없이 짠 코드와도 맞물려 돌아가게 해야만 했기 때문에 로 타입을 사용하는 메서드에 매개변수화 타입의 인스턴스를 넘겨도 동작해야 했다. 이 마이그레이션 호환성을 위해 로 타입을 지원하고 제네릭 구현에는 소거(아이템28) 방식을 사용했다.


List 같은 로 타입은 사용해서 안되지만, List<Object> 처럼 임의 객체를 허용하는 매개변수화 타입은 괜찮다. List는 제네릭 타입에서 완전히 발을 뺀 것이고, List<Object>는 모든 타입을 허용한다는 의사를 컴파일러에 명확히 전달한 것이다. 매개변수로 List를 받는 메서드에 List<String>을 넘길 수 있지만, List<Object>를 받는 메서드에는 넘길 수 없다. (List<String>은 로 타입 List의 하위 타입이지만, List<Object>의 하위 타입은 아니다(아이템28). 그 결과 List<Object> 같은 매개변수화 타입을 사용할 때와 달리 List 같은 로 타입을 사용하면 타입 안전성을 잃게 된다)

```java
// 런타임에 실패한다. - unsafeAdd 메서드가 로 타입(List)을 사용
public static void main(String[] args) {
	List<String> strings = new ArrayList<>();

	unsafeAdd(strings, Integer.valueOf(42));
	String s = strings.get(0); // 컴파일러가 자동으로 형변환 코드 추가
}

private static void unsafeAdd(List list, Object o) {
	list.add(o);
}

// 경고 발생
Test.java:10: warning: [unchecked] unchecked call to add(E) as a member of the raw type List
	list.add(0);
```

이 프로그램을 이대로 실행하면 strings.get(0)의 결과를 형변환하려 할 때 ClassCastException 을 던진다. Integer를 String으로 변환하려 시도한 것이다. 이 형변환은 컴파일러가 자동으로 만들어준 것이라 보통은 실패하지 않는다. 하지만 이 경우엔 컴파일러의 경고를 무시하여 그 대가를 치른 것이다.
  
List를 매개변수화 타입인 List<Object>로 바꾸고 컴파일하면 컴파일 조차 되지 않는다

```
Test.java:5: error: incompatible types: List<String> cannot be converted to List<Object>
	unsafeAdd(strings, Integer.valueOf(42));
```

로 타입은 위 경우를 허용한다고 해서 쓰고 싶을 수 있지만 다음 경우를 보면

```java
// 잘못된 예 - 모르는 타입의 원소도 받는 로 타입을 사용
static int numElementsInCommon(Set s1, Set s2) {
	int result = 0;
	for(Object o1 : s1) {
		if(s2.contains(o1))
			result++;
	}
	return result;
}
```

위 메서드는 동작은 하지만 로 타입을 사용해 안전하지 않다. 따라서 비한정적 와일드카드 타입을 대신 사용하는게 좋다. 제네릭 타입을 쓰고 싶지만 매개변수가 무엇인지 신경쓰고 싶지 않을 때는 물음표를 사용하자. (Set<E>의 비한정적 와일드카드 타입은 Set<?>이다. 어떤 타입이라도 담을 수 있는 가장 범용적인 매개변수화 Set타입이다)

```java
// 비한정적 와일드카드 타입 사용 - 타입 안전하며 유연하다
static int numElementsInCommon(Set<?> s1, Set<?> s2) { … }
```

Set<?> 과 Set 의 차이는 무엇일까?
특징을 간단히 말하면 와일드카드 타입은 안전하고 로 타입은 안전하지 않다. 로 타입 컬렉션에는 아무 원소나 넣을 수 있어서 불변식을 훼손하기 쉽지만, Collection<?> 에는 null 외에는 어떤 원소도 넣을 수 없다. 다른 원소를 넣으려 하면 컴파일할 때 다음의 오류 메시지를 보게 된다.

```
WildCard.java:13: error: incompatible types: String cannot be converted to CAP#1
		c.add(“verboten”);
	where CAP#1 is a fresh type-variable:
		CAP#1 extends Object from capture of ?
```

로 타입과는 다르게 컬렉션의 타입 불변식을 훼손하지 못하게 막았다. null을 제외한 어떠한 원소도 Collection<?> 에 넣지 못하게 했으며 컬렉션에서 꺼낼 수 있는 객체의 타입도 전혀 알 수 없게 했다. (이러한 제약이 싫다면 제네릭 메서드(아이템30)나 한정적 와일드카드 타입(아이템31)을 사용하면 된다)


로 타입을 쓰지 말라는 규칙도 예외가 몇개 있다. class 리터럴에는 로 타입을 써야 한다. 자바 명세는 class 리터럴에 매개변수화 타입을 사용하지 못하게 했다(배열과 기본 타입은 허용한다. 예를 들어 List.class, String[].class, int.class 는 허용하고 List<String>.class와 List<?>.class는 허용하지 않는다)

두 번째 예외는 instanceof 연산자와 관련이 있다. 런타임에는 제네릭 타입 정보가 지워지므로 instanceof 연산자는 비한정적 와일드카드 타입 이외의 매개변수화 타입에는 적용할 수 없다. 그리고 로 타입이든 비한정적 와일드카드 타입이는 instanceof는 똑같이 동작한다. 비한정적 와일드카드 타입이든 instanceof는 완전히 똑같이 동작한다. 비한정적 와일드 카드 타입의 꺽쇠괄호와 물음표는 아무런 역할 없이 코드만 지저분하게 만드므로 로 타입을 쓰는 편이 깔끔하다. 

```java
// 제네릭 타입에 instanceof를 사용하는 예
if(o instanceof Set) {	// 로 타입
	Set<?> s = (Set<?>) o; // 와일드 카드 타입
	…
}

// o의 타입이 Set임을 확인한 다음 와일드카드 타입인 Set<?>로 형변환해야 한다
```


> ::핵심 정리:: 
> 로 타입을 사용하면 런타입에 예외가 일어날 수 있으니 사용하면 안 된다. 로 타입은 제네릭이 도입되기 이전 코드와의 호환성을 위해 제공될 뿐이다. Set<Object>는 어떤 타입의 객체도 저장할 수 있는 매개변수화 타입이고, Set<?>는 모든 종의 타입 객체만 저장할 수 있는 와일드카드 타입이다. 로 타입인 Set은 제네릭 타입 시스템에 속하지 않는다. Set<Object>와 Set<?>은 안전하지만, 로 타입인 Set은 안전하지 않다.




