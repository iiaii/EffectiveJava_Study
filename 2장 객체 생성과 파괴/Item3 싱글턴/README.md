# #EffectiveJava/2장_객체생성과파괴/3싱글턴

## 3. private 생성자나 열거 타입으로 싱글턴임을 보증하라

싱글턴이란 인스턴스를 오직 하나만 생성할 수 있는 클래스를 말한다.
싱글턴의 전형적인 예로는 함수와 같은 무상태 객체나 설계상 유일해야 하는 시스템 컴포넌트를 들 수 있다. 

클래스를 싱글턴으로 만들면 이를 사용하는 클라이언트를 테스트하기가 어려워질 수 있다. (타입을 인터페이스로 정의한 다음 그 인터페이스를 구현해서 만든 싱글턴이 아니라면 싱글턴 인스턴스를 가짜(mock) 구현으로 대체할 수 없기 때문이다)

싱글턴을 만드는 방식은 보통 2가지 이다. 두방식 모두 생성자는 private으로 감춰두고 유일한 인스턴스에 접근할 수 있는 수단으로 public static 멤버를 하나 마련해둔다.

1. public static final 필드 방식의 싱글턴
```java
public class Elvis {
	public static final Elvis INSTANCE = new Elvis();
	private Elvis() { … }

	public void leaveTheBuilding() { … }
}
```

private 생성자는 public static final 필드인 Elvis.INSTANCE를 초기화할 때 딱 한번만 호출된다. public이나 protected 생성자가 없으므로 Elvis클래스가 초기화될 때 만들어진 인스턴스가 전체 시스템에서 하나뿐임이 보장된다. 클라이언트는 손 쓸 방법이 없다. 예외는 단 한가지, 권한이 있는 클라이언트는 리플렉션 API인 AccessibleObject.setAccessible을 사용해 private 생성자를 호출할 수 있다. 이러한 공격을 방어하려면 생성자를 수정하여 두번째 객체가 생성되려 할때 예외를 던지게 하면 된다.

### 장점

public 필드 방식의 큰 장점은 해당 클래스가 싱글턴임이 API에 명백히 드러난다. public static 필드가 final이니 절대로 다른 객체를 참조할 수 없다. 두번째 장점은 간결함이다.

2. 정적 팩토리 메서드 방식의 싱글턴

```java
public class Elvis {
	private static final Elvis INSTANCE = new Elvis();
	private Elvis() { … }
	public static Elvis getInstance() { return INSTANCE; }

	public void leaveTheBuilding() { … }
}
```

Elvis.getInstance는 항상 같은 객체의 참조를 반환하므로 제2의 Elvis 인스턴스는 결코 만들어 지지 않는 다(리플렉션을 통한 예외는 같이 적용됨)

### 장점

(마음이 바뀌면) API를 바꾸지 않고도 싱글턴이 아니게 변경할 수 있다는 장점이 있다. 유일한 인스턴스를 반환하던 팩터리 메서드가 (예컨대) 호출하는 스레드별로 다른 인스턴스를 넘겨주게 할 수 있다. 
두번째 장점은 원한다면 정적 팩토리를 제네릭 싱글턴 팩토리로 만들수 있다는 점이다.
세번째 장점은 정적 팩토리의 메서드 참조를 공급자(supplier)로 사용할 수 있다는 점이다. 가령 Elvis::getInstance를 Supplier<Elvis>로 사용하는 식이다. 이러한 장점들이 굳이 필요하지 않다면 public 필드 방식이 좋다

->
둘중 하나의 방식으로 만든 싱글턴 클래스를 직렬화하려면(12장 참조) 단순히 Serializable을 구현한다고 선언하는 것만으로는 부족하다. 모든 인스턴스 필드를 일시적이라고 선언하고 readResolve 메서드를 제공해야 한다. 이렇게 하지 않으면 직렬화된 인스턴스를 역직렬화할 때 마다 새로운 인스턴스가 만들어진다. (코드2의 예에서 가짜 Elvis가 탄생한다는 뜻) 
(가짜 Elvis 탄생을 예방하고 싶다면 Elvis 클래스에 다음의 readResolve 메서드를 추가)
```java
// 싱글턴임을 보장해주는 readResolve 메서드
private Object readResolve() {
	// 진짜 Elvis를 반환하고 가짜 Elvis는 가비지 컬렉터에 맡긴다
	return INSTANCE;
}
```

3. 열거타입 방식의 싱글턴

```java
public enum Elvis {
	INSTANCE;
	public void leaveTheBuilding() { … }
}
```

public 필드 방식과 비슷하지만, 더 간결하고 , 추가 노력없이 직렬화할 수 있고, 심지어 아주 복잡한 직렬화 상황이나 리플렉션 공격에서도 제2의 인스턴스가 생기는 일을 완벽히 막아준다. 

대부분 상황에서는 원소가 하나뿐인 열거 타입이 싱글턴을 만드는 가장 좋은 방법이다.

단, 만들려는 싱글턴이 Enum외의 클래스를 상속해야 한다면 이 방법은 사용할 수 없다 (열거 타입이 다른 인터페이스를 구현하도록 선언할 수는 있다)
