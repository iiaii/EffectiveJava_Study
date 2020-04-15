# #EffectiveJava/12장_직렬화/89readResolve


## 89. 인스턴스 수를 통제해야 한다면 readResolve 보다는 열거 타입을 사용하라


아이템3에서 싱글턴 패턴을 설명하며 다음 예를 보여주었다. 이 클래스는 바깥에서 생성자를 호출하지 못하게 막는 방식으로 인스턴스가 오직 하나만 만들어짐을 보장했다.

```java
public class Elvis {
	public static final Elvis INSTANCE = new Elvis();
	private Elvis() { … }

	public void leaveTheBuilding() { … }
}
```

아이템3에서 언급했듯 이 클래스는 그 선언에 implements Serializable을 추가하는 순간 더이상 싱글턴이 아니게 된다. 기본 직렬화를 쓰지 않더라도(아이템87) 그리고 명시적인 readObject를 제공하더라도(아이템88) 소용없다. 어떤 readObject를 사용하든 이 클래스가 초기화될 때 만들어진 인스턴스와는 별개인 인스턴스를 반환하게 된다.

readResolve 기능을 이용하면 readObject가 만들어낸 인스턴스를 다른 것으로 대체할 수 있다. 역직렬화한 객체의 클래스가 readResolve 메서드를 적절히 정의해뒀다면, 역직렬화 후 새로 생성된 객체를 인수로 이 메서드가 호출되고, 이 메서드가 반환한 객체 참조가 새로 생성된 객체를 대신해 반환된다. 대부분의 경우 이때 새로 생성된 객체의 참조는 유지하지 않으므로 바로 가비지 컬렉션 대상이 된다.


```java
// 인스턴스 통제를 위한 readResolve - 개선의 여지가 있다!
private Object readResolve() {
	// 진짜 Elvis를 반환하고, 가짜 Elvis는 가비지 컬렉터에 맡긴다.
	return INSTANCE;
}
```

이 메서드는 역직렬화한 객체는 무시하고 클래스 초기화 때 만들어진 Elvis 인스턴스를 반환한다. 따라서 Elvis 인스턴스의 직렬화 형태는 아무런 실 데이터를 가질 이유가 없으니 모든 인스턴스 필드를 transient로 선언해야 한다. **readResolve를 인스턴스 통제 목적으로 사용한다면 객체 참조 타입 인스턴스 필드는 모두 transient로 선언해야 한다.** 그렇지 않으면 아이템88과 같은 방식으로 역직렬화된 객체의 참조를 공격할 여지가 남는다.

```java
// 잘못된 싱글턴 - transient가 아닌 참조 필드를 가지고 있다!
public class Elvis implements Serializable {
	public static final Elvis INSTANCE = new Elvis();
	private Elvis() {}

	private String[] favoriteSons =
		{ “Hound Dog”, “Heartbreak Hotel” };
	public void printFavorites() {
		System.out.println(Arrays.toString(favoriteSongs));
	}

	private Objet readObject() {
		return INSTANCE;
	}
}
```


```java
// 도둑 클래스
public class ElvisStealer implements Serializable {
	static Elvis impersonator;
	private Elvis payload;

	private Object readResolve() {
		// resolve되기 전의 Elvis 인스턴스의 참조를 저장한다.
		impersonator = payload;
		
		// favoriteSongs 필드에 맞는 타입의 객체를 반환한다.
		return new String[]{“A Fool Such as I”};
	}
	private static final long serialVersionUID = 0;
}
```

```java
// 직렬화의 허점을 이용해 싱글턴 객체 2개 생성 (deserialize메서드 생략)
public class ElvisImpersonator {
	// 진짜 Elvis 인스턴스로는 만들어질 수 없는 바이트 스트림!
	private static final byte[] serializedForm = { … };

	public static void main(String[] args) {
		// ElvisStealer.impersonator를 초기화한 다음,
		// 진짜 Elvis(즉, Elvis.INSTANCE)를 반환한다.
		Elvis elvis = (Elvis) deserialize(serializedForm);
		Elvis impersonator = ElvisStealer.impersonator;

		elvis.printFavorites();
		impersonator.printFavorites();
	}
}

// 결과
// [Hound Dog, Heartbreak Hotel]
// [A Fool Such as I]
```

favoriteSongs 필드를 transient로 선언하여 이 문제를 고칠 수 있지만 Elvis를 원소하나짜리 열거 타입으로 바꾸는 편이 더 나은 선택이다. (아이템3)
ElvisStealer 공격 처럼 readResolve 메서드를 사용해 순간적으로 만들어진 역직렬화된 인스턴스에 접근하지 못하게 하는 방법은 깨지기 쉽고 신경을 많이 써야한다.

직렬화 가능한 인스턴스 통제 클래스를 열거 타입을 이용해 구현하면 선언 한 상수 외의 다른 객체는 존재하지 않음을 자바가 보장해 준다. (공격자가 AccessibleObject.setAccessible 같은 특권 메서드를 악용하면 모든 방어가 무력화됨)


```java
// 열거 타입 싱글턴 - 전통적인 싱글턴보다 우수하다
public enum Elvis {
	INSTANCE;
	private String[] favoriteSongs =
		{ “Hound Dog”, “Heartbreak Hotel” };
	public void printFavorites() {
		System.out.println(Arrays.toString(favoriteSongs));
	}
}
```


인스턴스 통제를 위해 readResolve를 사용하는 것이 완전히 쓸모없는 것은 아니다. 직렬화 가능 인스턴스 통제 클래스르 작성해야 하는데, 컴파일타임에는 어떤 인스턴스들이 있는지 알 수 없는 상황이라면 열거 타입으로 표현하는 것이 불가능하기 때문이다.

**readResolve 메서드의 접근성은 매우 중요하다.** final 클래스에서라면 readResolve 메서드는 private이어야 한다. final이 아닌 클래스에서는 다음의 몇가지를 주의해서 고려해야 한다. private으로 선언하면 하위 클래스에서 사용할 수 없다. package-private으로 선언하면 같은 패키지에 속한 하위 클래스에서만 사용할 수 있다. protected나 public으로 선언하면 이를 재정의하지 않은 모든 하위 클래스에서 사용할 수 있다. protected나 public이면서 하위 클래스에서 재정의하지 않았다면, 하위 클래스의 인스턴스를 역직렬화하면 상위 클래스의 인스턴스를 생성하여 ClassCastException을 일으킬 수 있다.


> **::핵심 정리::** 
> 불변식을 지키기 위해 인스턴스를 통제해야 한다면 가능한 열거 타임을 사용하자. 여의치 않은 상황에서 직렬화와 인스턴스 통제가 모두 필요하다면 readResolve 메서드를 작성해 넣어야 하고, 그 클래스에서 모든 참조타입 인스턴스 필드를 transient로 선언해야한다.


