# #EffectiveJava/5장_제네릭/28배열보다리스트

## 28. 배열보다는 리스트를 사용하라

배열과 제네릭 타입에는 중요한 차이 2가지가 있다.

첫 번째 배열은 공변이다. Sub가 Super의 하위 타입이라면 Sub[]는 Super[]의 하위 타입이 된다. (공변 : 함께 변함)
반면 제네릭은 불공변이다. 서로 다른 타입 Type1, Type2가 있을때 List<Type1>은 List<Type2>의 하위 타입도 아니고 상위 타입도 아니다.

```java
// 런타임 실패 - 문법상 허용
Object[] objectArray = new Long[1];
objectArray[0] = “타입이 달라 넣을 수 없다.”; // ArrayStoreException

// 컴파일 되지 않는다 - 문법상 오류
List<Object> ol = new ArrayList<Long>(); // 호환되지 않는 타입
ol.add(“타입이 달라 넣을 수 없다.”);
```

어느 쪽이든 실행되지 않지만, 배열은 런타임에 리스트는 컴파일에 알 수 있다. (컴파일 시에 알아채는 것이 좋음)


두 번째 주요 차이는 배열은 실체화 된다. (배열은 런타임에도 자신이 담기로 한 원소의 타입을 인지하고 확인한다) 그래서 ArrayStoreException이 발생한다. 반면 제네릭은 타입정보가 런타입에는 소거된다. 원소 타입을 컴파일타임에만 검사하며 런타임에는 알 수 조차 없다. (소거는 제네릭이 지원되기 전의 레거시 코드와 제네릭을 함께 사용할 수 있도록 해주는 메커니즘으로 자바5가 제네릭으로 순조롭게 전환될 수 있도록 해줬다.

-> 배열과 제네릭은 잘 어우러지지 못하는데, 배열은 제네릭타입, 매개변수화 타입, 타입 매개변수로 사용할 수 없다. (new List<E>[], new List<String>[], new E[] 는 컴파일에 제네릭 배열 생성 오류를 일으킨다)


제네릭 배열을 못 만드는 이유는 타입 안전하지 않기 때문이다. 이를 허용하면 컴파일러가 자동생성한 형변환 코드에서 런타임에 ClassCastException이 발생할 수 있다. 런타입에 ClassCastException이 발생하는 일을 막아주겠다는 제네릭 타입 시스템의 취지에 어긋난다.

```java
// 제네릭 배열 생성을 허용하지 않는 이유 - 컴파일되지 않는다
List<String>[] stringLists = new List<String>[1];//1-허용된다가정
List<Integer> intList = List.of(42); // 2 - 원소하나인 List<>
Object[] objects = stringLists; // 3 - 배열은 공변이므로 문제 X
objects[0] = intList; // 4 - 2에서 생성한 인스턴스 0에 저장
String s = stringLists[0].get(0); // 5
```

런타임에는 List<Integer> 인스턴스의 타입은 단순히 List가 되고, List<Integer>[] 인스턴스의 타입은 List[]가 된다. 따라서 4에서도 ArrayStoreException을 일으키지 않는다. stringLists 배열에는 List<Integer> 인스턴스가 저장돼 있다. 그리고 이 배열의 처음 리스트에서 첫 원소를 꺼내려 한다. 컴파일러는 꺼낸 원소를 자동으로 String으로 형변환 하는데, 이 원소는 Integer 이므로 런타임에 ClassCastException이 발생한다. 이런 일을 방지하려면 (제네릭 배열이 생성되지 않도록 (1)에서 컴파일 오류를 내야 한다.

E, List<E>, List<String> 같은 타입을 실체화 불가 타입이라 한다. 실체화되지 않아서 런타임에는 컴파일 타임보다 타입 정보를 적게 가지는 타입이다. 소거 메커니즘 때문에 매개변수화 타입 가운데 실체화 될 수 있는 타입은 List<?> 와 Map<?,?> 같은 비한정적 와일드 카드 타입이다.(아이템26) (거의 안쓰임)

제네릭 컬렉션에서는 자신의 원소 타입을 담은 배열을 반환하는게 보통은 불가능하다.(아이템33) 또한 제네릭 타입과 가변인수 메서드(아이템53)를 함게 쓰면 해석하기 어려운 경고 메시지를 받게 된다. 가변 인수를 호출할 때마다 가변인수 매개변수를 담을 배열이 하나 만들어지는데, 이때 그 배열의 원소가 실체화 불가 타입이라면 경고가 발생한다. (@safeVarags 애너테이션으로 대처(아이템32))


배열로 형변환할 때 제네릭 배열 생성 오류나 비검사 형변환 경고가 뜨는 경우 대부분은 배열인 E[] 대신 컬렉션인 List<E>를 사용하면 해결된다. 코드가 조금 복잡해지고 성능이 살짝 나빠질 수도 있지만, 그 대신 타입 안전성과 상호운용성은 좋아진다.

생성자에서 컬렉션을 받는 Chooser 클래스를 예로 보명 이 클래스는 컬렉션 안의 원소중 하나를 무작위로 선택해 반환하는 choose 메서드를 제공한다. 

```java
// Chooser - 제네릭 적용해야함
public class Chooser {
	private final Object[] choiceArray;

	public Chooser(Collection choices) {
		choiceArray = choices.toArray();
	}

	public Object choose() {
		Random rnd = ThreadLocalRandom.current();
		return choiceArray[rnd.nextInt(choiceArray.length)];
	}
}
```

이 클래스를 사용하려면 choose 메서드를 호출할 때마다 반환된 Object를 원하는 타입으로 형변환해야 한다. 혹시나 타입이 다른 원소가 들어 있었다면 런타임에 형변환 오류가 날 것이다. 

```java
// Chooser 를 제네릭으로 만드는 1차시도 - 컴파일 되지 않음
public class Chooser<T> {
	private final T[] choiceArray;

	public Chooser(Collection<T> choices) {
		choicArray = choices.toArray();
	}

	// choose 메서드 그대로
}

// 컴파일 에러 메시지
Chooser.java:9: error: incompatible types: Object[] cannot be converted to T[]
		choiceArray = choices.toArray();

	where T is a type-variable:
		T extends Object declared in class Chooser

// Object 배열을 T배열로 형변환하면 된다
// choiceArray = (T[]) choices.toArray();

// 고치면은 경고가 뜬다
Chooser.java:9: warning: [unchecked] unchecked cast
		choiceArray = (T[]) choices.toArray();

	required: T[], found Object[]
	where T is a type-variable:
T extends Object declared in class Chooser
```

T가 무슨 타입인지 알 수 없으니 컴파일러는 이 형변환이 런타임에도 안전한지 보장할 수 없다는 메시지가 나온다. (**제네릭에서는 원소의 타입 정보가 소거되어 런타임에는 무슨 타입인지 알 수 없음을 기억하자!**)

컴파일러가 안전을 보장하지 못할 뿐 동작은 한다. 안전하다고 확신하면 주석을 남기고 애너테이션을 달아 경고를 숨겨도 되지만 경고의 원인을 제거하는 것이 가장 좋다(아이템27)

```java
// 배열대신 리스트를 사용하면 모두 해결된다
// 리스트 기반 Chooser - 타입 안전성 확보!
public class Chooser<T> {
	private final List<T> choiceList;
	
	public Chooser(Collection<T> choices) {
		choiceList = new ArraysList<>(choices);
	}

	public T choose() {
		Random rnd = ThreadLocalRandom.current();
		return choiceList.get(rnd.nextInt(choiceList.size()));
	}
}
```
-> 코드가 길어지고, 조금 느리지만 런타임에 ClassCastException을 만날 일이 없다

> ::핵심 정리:: 
> 배열과 제네릭에는 매우 다른 타입 규칙이 적용된다. 배열은 공변이고 실체화되는 반면, 제네릭은 불공변이고 타입 정보가 소거된다. 그 결과 배열은 런타임에는 타입 안전하지만 컴파일타임에는 그렇지 않다. 제네릭은 반대다. 그래서 둘을 섞어 쓰기란 쉽지 않다. 둘을 섞어 쓰다가 컴파일 오류나 경고를 만나면, 가장 먼저 배열을 리스트로 대체하는 방법을 적용하는 것이 좋다.

