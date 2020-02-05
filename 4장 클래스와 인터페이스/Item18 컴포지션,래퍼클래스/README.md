# #EffectiveJava/4장_클래스와인터페이스/18컴포지션

## 18. 상속보다는 컴포지션을 사용하라

상속은 코드를 재사용하는 강력한 수단이지만, 항상 최선은 아니다. 잘못 사용하면 오류를 내기 쉬운 소프트웨어를 만들게 된다. 상위 클래스와 하위 클래스 모두 같은 프로그래머가 통제하는 패키지 안에서라면 상속도 안전한 방법이다. 확장할 목적으로 설계되었고 문서화도 잘 된 클래스(아이템19)도 마찬기지다.

하지만 일반적인 구체 클래스를 패키지 경계를 넘어, 즉 다른 패키지의 구체 클래스를 상속하는 일은 위험하다. (여기서 상속은 클래스가 다른 클래스를 확장하는 구현 상속을 말한다. (클래스가 인터페이스를 구현하거나, 인터페이스가 다른 인터페이스를 확장하는 인터페이스 상속과는 무관하다)


메서드 호출과 달리 상속은 캡슐화를 깨뜨린다. 다르게 말하면, 상위 클래스가 어떻게 구현 되느냐에 따라 하위 클래스의 동작에 이상이 생길 수 있다. 상위 클래스는 릴리스마다 내부 구현이 달라질 수 있으며, 그 여파로 코드 한 줄 건드리지 않은 하위 클래스가 오동작할 수 있다. 이러한 이유로 상위 클래스 설계자가 확장을 충분히 고려하고 무서화도 제대로 해두지 않으면 하위 클래스는 상위 클래스의 변화에 맞춰 수정되어야 한다.

예로, HashSet을 사용하는 프로그램이 있을때 성능을 높이려면 HashSet 생성이후 원소가 몇 개 더해졌는지 알 수 있어야 한다. 그래서 다음과 같이 상속을 했는데

```java
// 잘못 상속된 예
public class InstrumentedHashSet<E> extends HashSet<E> {
	// 추가된 원소의 수
	private int addCount = 0;

	public InstrumentedHashSet() {
	}

	public InstrumentedHashSet(int initCap, float loadFactor) {
		super(initCap, loadFactor);
	}

	@Override
	public boolean add(E e) {
		addCont++;
		return super.add(e);
	}

	@Override 
	public boolean addAll(Collection<? extends E> c) {
		addCount += c.size();
		return super.addAll(e);
	}

	public int gtAddCount() {
		return addCount;
	}
}		
```

이 클래스는 잘 구현된 것 같지만 제대로 동작하지 않는다. 이 클래스의 인스턴스에 addAll 메서드로 원소 3개를 더했다고 했을때 addCount에 3이 추가 되고 addAll이 실행 되는데 상위 addAll은 내부적으로 add를 호출하고 이때 재정의된 add가 호출되어 원소가 더해질 때 마다 추가로 1씩 더해져 addCount 의 값은 6이된다.
-> 내부 구현 방식은 보통 문서화 되지 않고 이후에도 유지될지 알 수 없어서 이런 가정에 기대서 구현 상속한 클래스는 깨지기 쉽다

하위 클래스가 깨지기 쉬운 이유는 더 있는데, 다음 릴리스에서 상위 클래스에 새로운 메서드가 추가 된다고 했을때, 보안 때문에 컬렉션에 추가된 모든 원소가 특정 조건을 만족해야만 하는 프로그램을 생각하면. 그 컬렉션을 상속하여 원소를 추가하는 모든 메서드를 재정의해 필요한 조건을 먼저 검사하게끔 해야한다. (즉, 하위 클래스에서 재정의하지 못한 그 새로운 메서드를 사용해 허용되지 않은 원소가 추가될 수도 있다) 실제로 Hashtable과 Vector를 컬렉션 프레임워크에 포함시키자 이와 관련한 보안 구멍들을 수정해야 하는 사태가 벌어졌다. 

모두 메서드 재정의가 원인이었기 때문에 클래스를 확장하더라도 메서드를 재정의하는 대신 새로운 추가 메서드를 추가하면 괜찮다고 생각 할 수 있다. (하지만 운없게도 하위 클래스에 추가한 메서드와 시그니처가 같고 반환 타입은 다르면 컴파일 되지 않는다. 반환 타입마저 같다면 상위 클래스의 새 메서드를 재정의한 것이 되어 버린다)

기존 클래스를 확장하는 대신, 새로운 클래스를 만들고 private 필드로 기존 클래스의 인스턴스를 참조하게 하면 해결 된다. **기존 클래스가 새로운 클래스의 구성요소로 쓰인다는 뜻에서 이러한 설계를 컴포지션(composition: 구성)이라 한다.** 새 클래스의 인스턴스 메서드들은 기존 클래스의 대응하는 메서드를 호출해 결과를 반환하는데, 이를 전달이라 하며, 새 클래스의 메서드들을 전달 메서드(forwarding method)라 부른다. 새로운 클래스는 기존 클래스의 내부 구현 방식의 영향에서 벗어나며, 기존 클래스에 새로운 메서드가 추가되더라도 전혀 영향 받지 않는다. 

```java
// 래퍼 클래스 - 상속 대신 컴포지션 사용
public class InstrumentedSet<E> extends ForwardingSet<E> {
	private int addCount = 0;

	public InstrumentedSet(Set<E> s) {
		super(s);
	}

	@Override
	public boolean add(E e) {
		addCount++;
		return super.add(e);
	}

	@Override
	public boolean addAll(Collection<? extends E> c) {
		addCount += c.size();
		return super.addAll();
	}

	public int getAddCount() {
		return addCount;
	}
}

// 재사용할 수 있는 전달 클래스
public class ForwardingSet<E> implements Set<E> {
	private final Set<E> s;
	public ForwardingSet(Set<E> s) { this.s = s; }

	public void clear() { s.clear(); }
	public boolean contains(Object i) { return s.contains(o); }
	public boolean isEmpty() { return s.isEmpty(); }
	public int size() { return s.size(); }
	public Iterator<E> iterator() { return s.iterator(); }
	public boolean add(E e) { return s.add(e); }
	public boolean remove(Object o) { return s.remove(o); }
	public boolean containsAll(Collection<?> c) { return s.containsAll(c); }
	public boolean addAll(Collection<? extends E> c) { return s.addAll(c); }
	public boolean removeAll(Collection<?> c) { return s.removeAll(c); }
	public boolean retainAll(Collection<?> c) { return s.retainAll(c); }
	public Object[] toArray() { return s.toArray(); }
	public <T> T[] toArray(T[] a) { return s.toArray(a); }
	@Override 
	public boolean equals(Object o) { return s.equals(o); }
	@Override
	public int hashCode() { return s.hashCode(); }
	@Override
	public String toString() { return s.toString(); }
}
```

구체적으로는 Set 인터페이스를 구현했고, Set 인스턴스를 인수로 받는 생성자를 제공해서 임의의 Set에 계측 기능을 덧씌워 새로운 Set으로 만들게 되었다. (상속 방식은 구체 클래스 각각을 따로 확장해야 하며, 지원하고 싶은 상위 클래스의 생성자 각각에 대응하는 생성자를 별도로 정의해줘야 한다)

```java
// 사용
Set<Instant> times - new InstrumentedSet<>(new TreeSet<>(tmp));
Set<E> s = new InstrumentedSet<>(new HashSet<>(INIT_CAPACITY));

static void walk(Set<Dog> dogs) {
	InstrumentedSet<Dog> iDogs = new InstrumentedSet<>(dogs);
	// 이 메서드에서는 dogs대신 iDogs를 사용
}
```

다른 Set 인스턴스를 감싸고 있다는 뜻에서 InstrumentedSet 같은 클래스를 래퍼 클래스라 하고, 다른 Set에 계측 기능을 덧씌운다는 뜻에서 데코레이터 패턴이라고 한다. 컴포지션과 전달의 조합은 넓은 의미로 위임이라고 부른다. (엄밀히 따지면 래퍼 객체가 내부 객체에 자기 자신의 참조를 넘기는 경우만 위임에 해당)

래퍼 클래스는 단점이 거의 없다. 다만, 래퍼 클래스가 콜백 프레임워크와는 어울리지 않는 다는 것을 주의해야 한다. 콜백 프레임워크에서는 자기 자신의 참조를 다른 객체에 넘겨서 다음 호출때 사용하도록 한다. 내부 객체는 자신을 감싸고 있는 래퍼의 존재를 모르니 대신 자신(this)의 참조를 넘기고, 콜백 때는 래퍼가 아닌 내부 객체를 호출하게 된다. (SELF문제)

전달 메서드가 성능에 주는 영향이나 래퍼 객체가 메모리 사용량에 주는 영향을 걱정하는 사람도 있지만, 실전에서는 둘다 별다른 영향이 없다고 밝혀졌다. 전달 메서드를 인터페이스 당 하나씩만 만들어두면 원하는 기능을 덧씌우는 클래스를 쉽게 구현할 수 있다. (구아바는 모든 컬렉션 인터페이스용 전달 메서드를 전부 구현해 뒀다)


상속은 반드시 하위 클래스가 상위 클래스의 진짜 하위 타입인 상황에서만 쓰여야 한다. 클래스 B가 클래스 A와 is-a 관계일 때만 클래스를 상속해야 한다. 클래스 A를 상속하는 클래스 B를 작성할 때 정말 B가 A인가? 를 생각해 보아야 한다. 아니라면 A를 private 인스턴스로 두고, A와는 다른 API를 제공해야 하는 상황이 대다수이다. A는 B의 필수 구성요소가 아니라 구현하는 방법 중 하나일 뿐이다.

자바 플랫폼 라이브러리에서도 위반한 클래스가 있는데, 스택은 멕터가 아니므로 Stack은 Vector를 확장해서는 안 됐다. 마찬가지로 속성 목록도 해시테이블이 아니므로 Properties도 Hashtable을 확장해서는 안 됐다. (컴포지션을 사용했다면 좋았을 것)


컴포지션을 써야 할 상황에서 상속을 하는 건 내부 구현을 불필요하게 노출하는 꼴이다. 그 결과 API가 내부 구현에 묶이고 그 클래스의 성능도 영원히 제한된다. 더 심각한 것은 클라이언트가 노출된 내부에 직접 접근 할 수 있다는 것이다. (상위클래스를 직접 수정하여 하위 클래스의 불변식을 해칠 수 있음)

컴포지션 대신 상속을 사용하기로 결정하기 전에 마지막으로 자문해야할 질문은 “확장하려는 클래스의 API에 아무런 결함이 없는가? 결함이 있다면, 이 결함이 현 클래스의 API까지 전파돼도 괜찮은가?” 이다. 컴포지션으로는 이런 결함을 숨기는 새로운 API를 설계할 수 있지만, 상속은 상위 클래스의 API를 그 결함까지도 그대로 승계한다.

> ::핵심 정리:: 
> 상속은 강력하지만 캡슐화를 해친다는 문제가 있다. 상속은 상위 클래스와 하위 클래스가 순수한 is-a 관계일 때만 써야 한다. is-a 관계일 때도 안심할 수만은 없는게, 하위 클래스의 패키지가 상위클래스와 다르고, 상위 클래스가 확장을 고려해 설계되지 않았다면 여전히 문제가 될 수 있다. 상속의 취약점을 피하려면 상속 대신 컴포지션과 전달을 사용하자. 특히 래퍼 클래스로 구현할 적당한 인터페이스가 있다면 래퍼클래스를 사용하는 것이 하위 클래스보다 견고하고 강력하다.




