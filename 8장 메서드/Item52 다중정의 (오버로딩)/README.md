# #EffectiveJava/8장_메서드/52다중정의(오버로딩)


## 52. 다중정의는 신중히 사용하라


```java
// 컬렉션 분류기 - 오류! 이 프로그램의 출력은 ?
public class CollectionClassifier {
	public static String classify(Set<?> s) {\
		return “집합”;
	}

	public static String classify(List<?> let) {
		return “리스트”;
	}

	public static String classify(Collection<?> c) {
		return “그 외”;
	}

	public static void main(String[] args) {
		Collection<?>[] collections = {
			new HashSet<String>(),
			new ArrayList<BigInteger>(),
			new HashMap<String, String>().values()
		};

		for(Collection<?> c : collections)
			Systems.out.println(classify(c));
	}
}

// 출력 결과 :
// 그 외
// 그 외
// 그 외
```

-> classify 중 어느 메서드를 호출할지가 컴파일 타임에 정해지기 때문, for 문 안의 c는 항상 Collection<?> 타입이다. 런타임에는 타입이 매번 달라지지만 호출할 메서드를 선택하는 데 영향을 주지 못하고 컴파일타임의 매개변수 타입을 기준으로 항상 세번째 메서드만 호출하게 된다.

직관과 어긋나는 이유는 **재정의(오버라이딩)한 메서드는 동적으로 선택되고, 다중정의(오버로딩)한 메서드는 정적으로 선택되기 때문이다.** 메서드를 재정의 했다면 해당 객체의 런타임 타입이 어떤 메서드를 호출할지의 기준이 된다. 메서드를 재정의 한다음 하위 클래스의 인스턴스에서 그 메서드를 호출하면 재정의한 메서드가 실행 된다. (컴파일 타임에 그 인스턴스 타입이 무엇이었냐는 상관없다)


```java
// 재정의된 메서드 호출 메커니즘 - 이 프로그램의 출력은 ?
class Wine {
	String name() { return “포도주”; }
}

class SparklingWine extends Wine {
	@Override String name() { return “발포성 포도주”; }
}

class Champagne extends SparklingWine {
	@Override String name() { return “샴페인”; }
}

public class Overriding {
	public static void main(String[] args) {
		List<Wine> wineList = List.of(
			new Wine(), new SparklingWine(), new Champagne());

		for(Wine wine : wineList)
			System.out.println(wine.name());
	}
}

// 출력 결과 :
// 포도주
// 발포성 포도주
// 샴페인
```

-> for 문에서 컴파일타임 타입이 Wine인 것과 무관하게 가장 하위에서 정의한 재정의 메서드가 실행 된다. 한편 **다중정의(오버로딩)된 메서드 사이에서는 객체의 런타임 타입은 전혀 중요하지 않고, 선택은 컴파일타임에, 오직 매개변수의 컴파일타임 타입에 의해 이루어 진다.**

재정의 (오버라이딩) 메서드 -> 런타임 타임에 결정 (동적)
다중정의 (오버로딩) 메서드 -> 컴파일 타임에 결정 (정적)


앞의 CollectionClassifier 예를 수정한다면 다음과 같다

```java
public static String classify(Collection<?> c) {
	return c instanceof Set ? “집합” :
		   c instanceof List ? “리스트” : “그 외”;
}
```

이러한 다중정의의 예외적인 동작처럼 헷갈릴 수 있는 코드는 작성하지 않는게 좋다. (특히 공개 API의 경우) -> 다중정의가 혼동을 일으키는 상황을 피해야 함

안전하고 보수적으로 가려면 매개변수가 같은 다중정의는 만들지 말자. 가변인수를 사용하는 메서드라면 다중정의를 아예 하지 말아야 한다(아이템53). 이 규칙만 잘 따르면 어떤 다중정의 메서드가 호출될지 헷갈릴 일은 없다. (다중정의가 아닌 다른 이름으로 짓는 해결책도 있다)

`writeBoolean(boolean) / writeInt(int) / writeLong(long)` 같이 구성하면 read 메서드의 이름과 짝을 맞추기 좋다. (readBoolean(), readInt() 등) ObjectInputStream 클래스의 read 메서드는 이렇게 되어 있다.


생성자는 이름을 다르게 지을 수 없으니 두번째 생성자 부터는 무조건 다중정의(오버로딩)가 된다. 하지만 정적 팩터리라는 대안을 활용할 수 있는 경우가 많다(아이템1). 또한 생성자는 재정의할 수 없으니 다중정의와 재정의가 혼용될 걱정은 안해도 된다. 그래도 여러 생성자가 같은 수의 매개변수를 받아야하는 경우에는 다음과 같이 해결한다.

매개변수 수가 같은 다중정의 메서드가 많더라도, 그중 어느 것이 주어진 매개변수 집합을 처리할지가 명확히 구분된다면 헷갈릴 일은 없다. 즉 매개변수 중 하나 이상이 근본적으로 다르다면 헷갈릴 일이 없다. (두 타입이 (null이 아닌) 값을 서로 어느쪽으로든 형변환할 수 없다) 

이 조건을 충족하면 어느 다중정의 메서드를 호출할지가 매개변수들의 런타임 타입만으로 결정된다. 예를 들어 ArrayList 의 int, Collection을 받는 생성자는 어느 것이 호출될지 헷갈릴 일이 없다.


```java
// 자바 오토박싱
public class SetList {
	public static void main(String[] args) {
		Set<Integer> set = new TreeSet<>();
		List<Integer> list = new ArrayList<>();
	
		for(int i=-3 ; i<3 ; i++) {
			set.add(i);
			list.add(i);
		}

		for(int i=0 ; i<3 ; i++) {
			set.remove(i);
			list.remove(i);
		}
		System.out.println(set+” “+list);
	}
}

// 출력 결과 :
// [-3, -2, -1] [-2, 0, 2]

// 예상은 [-3, -2, -1] [-3, -2, -1]
```

위의 경우 set.remove(i)의 시그니처는 remove(Object)인데 다중정의된 다른 메서드가 없으므로 기대한대로 동작한다.

하지만 list.remove(i)는 다중정의된 remove(int index)를 선택해서 [-3, -2, -1, 0, 1, 2]에서 0번째 제거, [-2, -1, 0, 1, 2]에서 1번째 제거 ... -> [-2, 0, 2] 가 된다.

위 문제를 해결하려면
```java
for(int i=0 ; i<3 ; i++) {
	set.remove(i);
	list.remove((Integer)i); // remove(Integer.valueOf(i));
}
```


이 예가 혼란스러운 이유는 List<E> 인터페이스가 remove(Object)와 remove(int)를 다중정의했기 때문이다. 이러한 예가 또 있는데

```java
// 1번. Thread의 생성자 호출
new Thread(System.out::println).start();

// 2번. ExecutorService의 submit 메서드 호출
ExecutorService exec = Executors.newCachedThreadPool();
exec.submit(System.out::println);
```

여기서 2번만 컴파일 오류가 난다. 넘겨진 인수는 모두 System.out::println으로 같고 양쪽 모두 Runnable을 받는 형제 메서드를 다중정의하고 있지만 submit 다중정의 메서드 중에 Callable<T>를 받는 메서드도 있는데 이와 혼동한다 (println이 void를 반환하지만 참조된 println 과 호출한 submit 양쪽 다 다중정의되어 부정확한 메서드 참조이며 이와 같은 인수 표현식은 목표타입이 선택되기 전에 그 의미가 정해지지 않는다)
-> 함수형 인터페이스라도 인수 위치가 같으면 혼란이 생긴다. 따라서 메서드를 다중정의할 때, 서로 다른 함수형 인터페이스라도 같은 위치의 인수로 받아서는 안된다. (서로 다른 함수형 인터페이스라도 서로 근복적으로 다르지 않다는 뜻)

Object 외의 클래스 타입과 배열 타입은 근본적으로 다르다. Serializable 과 Cloneable 외의 인터페이스 타입과 배열 타입도 근본적으로 다르다. 한편, String 과 Throwable 처럼 상위/하위 관계가 아닌 두 클래스는 관련없다고 한다.
이 외에도 어떤 방향으로도 형변활할 수 없는 타입 쌍이 있지만 앞 문단에서 나열한 간단한 예보다 복잡해지면 대부분 프로그래머는 어떤 다중정의 메서드가 선택될지를 구분하기 어려워 한다.


이번 아이템의 지침을 어기고 싶은 경우가 있는데, String은 자바 4부터 contentEquals(StringBuffer) 메서드를 가지고 있었는데, 자바 5에서 StringBuffer, StringBuilder, String, CharBuffer 등의 비슷한 부류의 타이을 위한 공통 인터페이스로 CharSequence가 등장하였고, 자연스럽게 String에도 CharSequence를 받은 contentEquals가 다중정의되었다. 
-> 이번 지침을 어기는 모습인데, 기능이 같기 때문에 크게 문제가 되지는 않는다.

```java
// 인수를 포워드하여 두 메서드가 동일한 일을 하도록 보장
public boolean contentEquals(StringBuffer sb) {
	return contentEquals((CharSequence) sb);
}
```

이 외에도 Sting 클래스의 valueOf(char[]) 과 valueOf(Object)는 같은 객체를 건네더라도 전혀 다른 일을 수행한다. 이렇게 해야 할 이유가 없었음에도, 혼란을 불러올 수 있는 잘못된 사례로 남게 되었다.

> **::핵심 정리::**
> 프로그래밍 언어가 다중정의를 허용한다고 해서 다중정의를 꼭 활용하란 뜻은 아니다. 일반적으로 매개변수 수가 같을 때는 다중정의를 피하는게 좋다. 특히 생성자라면 이 조언을 따르기 불가능 할 수 있느데, 헷갈릴 만한 매개변수는 형변환하여 정확한 다중정의 메서드가 선택되도록 해야 한다. 이것이 불가능하면, 기존 클래스를 수정해 새로운 인터페이스를 구현해야 할 때는 같은 객체를 입력받는 다중정의 메서드들이 모두 동일하게 동작하도록 만들어야 한다. (안그러면 굉장히 혼란스러울 수 있음)



