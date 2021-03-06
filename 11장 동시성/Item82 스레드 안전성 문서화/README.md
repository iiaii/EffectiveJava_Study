# #EffectiveJava/11장_동시성/82스레드안전성문서화


## 82. 스레드 안전성 수준을 문서화하라

한 메서드를 여러 스레드가 동시에 호출할 때 그 메서드가 어떻게 동작하느냐는 해당 클래스와 이를 사용하는 클라이언트 사이의 중요한 계약과 같다. API 문서에서 아무런 언급도 없으며 그 클래스 사용자는 동기화를 충분히 하지 못하거나, 지나치게 하는 경우로 모두 오류로 이어질 수 있다. 

API 문서에 synchronized 한정자가 보이는 메서드는 스레드 안전하다고 생각할 수 있지만, 몇가지 면에서 틀렸다. 자바독 기본 옵션에서 생성한 API 문서에는 synchronized 한정자가 포함되지 않는다. **메서드 선언에 synchronized 한정자를 선언할지는 구현 이슈일 뿐 API에 속하지 않는다.** 따라서 이것만으로는 그 메서드가 스레드 안전하다고 믿기 어렵다.
 synchronized 유무로 스레드 안전성을 알수 있다는 주장은 스레드 안전성은 모 아니면 도라는 오해에 뿌리를 둔 것이다. (스레드 안전성에도 수준이 나뉨) **멀티스레드 환경에서도 API를 안전하게 사용하게 하려면 클래스가 지원하는 스레드 안전성 수준을 정확히 명시해야 한다.** 


다음은 스레드 안전성이 높은 순으로 나열한 것이다 (일반적인 경우를 포괄)

- 불변(immutable) : 이 클래스의 인스턴스는 상수와 같아서 외부 동기화도 필요 없다. (String, Long, BigInteger 등)
- 무조건 스레드 안전(unconditionally thread-safe) : 이 클래스의 인스턴스는 수정될 수 있으나, 내부에서 충실히 동기화하여 별도의 외부 동기화 없이 동시에 사용해도 안전하다. (AtomicLong, ConcurrentHashMap이 여기에 속함)
- 조건부 스레드 안전(not thread-safe) : 이 클래스의 인스턴스는 수정될 수 있다. 동시에 사용하려면 각각의(혹은 일련의) 메서드 호출을 클라이언트가 선택한 외부 동기화 메커니즘으로 감싸야 한다. (ArrayList, HashMap 같은 기본 컬렉션이 여기 속함)
- 스레드 적대적(thread-hostile) : 이 클래스는 모든 메서드 호출을 외부 동기화로 감싸더라도 멀티스레드 환경에서 안전하지 않다. 이 수준의 클래스는 일반적으로 정적 데이터를 아무 동기화 없이 수정한다. 이런 클래스를 고의로 만드는 사람은 없겠지만, 동시성을 고려하지 않고 작성하다 보면 우연히 만들어질 수 있다. 스레드 적대적으로 밝혀진 클래스나 메서드는 일반적으로 문제를 고쳐 재배포하거나 사용 자제 API로 지정한다. (아이템78의 generateSerialNumber 메서드에서 내주 동기화를 생략하면 스레드 적대적이게 된다)

-> 이 분류는 (스레드 적대적을 제외하고) [자바 병렬 프로그래밍]의 부록에 나오는 스레드 안전성 애너테이션과 대략 일치한다. 


조건부 스레드 안전한 클래스는 주의해서 문서화해야 한다. 어떤 순서로 호출할 때 외부 동기화가 필요한지, 그리고 그 순서로 호출하려면 어떤 락 혹은 락을 얻어야 하는지 알려줘야 한다. 일반적으로 인스턴스 자체를 락으로 얻지만 예외도 있다. 예를 들어 Collections.synchronizedMap의 API 문서에는 다음과 같이 써 있다.

> synchronizedMap이 반환한 맵의 컬렉션 뷰를 순회하려면 반드시 그 맵을 락으로 사용해 수동으로 동기화하라.
>  
> Map<K, V> m = Collections.synchronizedMap(new HashMap<>());
> Set<K> s = m.keySet(); // 동기화 블록 밖에 있어도 된다.
> 	…
> synchronized(m) { // s가 아닌 m을 사용해 동기화해야 한다!
> 	for(K key : s)
> 		key.f();
> }
>  
> 이대로 따르지 않으면 동작을 예측할 수 없다.

클래스의 스레드 안전성은 보통 클래스의 문서화 주석에 기재하지만, 독특한 특성의 메서드는 해당 메서드의 주석에 기재하자. 열거 타입은 굳이 불변이라고 쓰지 않아도 된다. 반환 타입만으로는 명확히 알 수 없는 정적 팩토리라면 자신이 반환하는 객체의 스레드 안전성을 반드시 문서화해야 한다. 앞서의 Collections.synchronizedMap이 좋은 예다.

클래스가 외부에서 사용할 수 있는 락을 제공하면 클라이언트에서 일련의 메서드 호출을 원자적으로 수행할 수 있다. 하지만 이 유연성은 내부에서 처리하는 고성능 동시성 제어 메커니즘과 혼용할 수 없다. (ConcurrentHashMap 과 같은 동시성 컬렉션과는 함께 사용하지 못함) 또한, 클라이언트가 공개된 락을 오래 쥐고 놓지 않는 서비스 거부 공격을 수행할 수도 있다.
 서비스 거부 공격을 막으려면 synchronized 메서드(이 역시 공개된 락이나 마찬가지) 대신 비공개 락 객체를 사용해야 한다.

```java
// 비공개 락 객체 관용구 - 서비스 거부 공격을 막아준다
private final Object lock = new Object();

public void foo() {
	synchronized(lock) {
		…
	}
}
```

비공개 락 객체는 클래스 바깥에서는 볼 수 없으니 클라이언트가 그 객체의 동기화에 관여할 수 없다. 사실 아이템 15의 조언을 따라 락 객체를 동기화 대상 객체 안으로 캡슐화한 것이다.

> 앞의 코드에서 lock 필드를 final로 선언했다. 이는 우연히라도 락 객체가 교체되는 일을 예방해준다. 락이 교체되면 끔찍한 결과로 이어진다(아이템78). 아이템17 의 조언을 따라 다시 한번 락 필드의 변경 가능성을 최소화한 것이다. 이 예처럼 락 필드는 항상 final로 선언하라. 일반적인 감시 락이든 java.util.concurrent.locks 패키지에서 가져온 락이든 마찬가지다.

비공개 락 객체 관용구는 무조건적 스레드 안전 클래스에서만 사용 가능하다. 조건부 스레드 안전 클래스에서는 특정 호출 순서에 필요한 락이 무엇인지를 클라이언트에게 알려줘야 하므로 이 관용구를 사용할 수 없다.

비공개 락 객체 관용구는 상속용으로 설계한 클래스(아이템19)에 특히 잘 맞는다. 상속용 클래스에서 자신의 인스턴스를 락으로 사용한다면, 하위 클래스는 아주 쉽게, 기반 클래스의 동작을 방해할 수 있다. (그 반대도 마찬가지) 같은 락을 다른 목적으로 사용하게 되어 하위 클래스와 기반 클래스는 서로가 서로를 훼방놓는 상태에 빠지게 된다. 단지 가능성에 그치지 않고 실제로 Thread 클래스에서 나타나곤 하는 문제다.


**::핵심 정리::** 

> 모든 클래스가 자신의 스레드 안전성 정보를 명확히 문서화해야 한다. (정확한 언어로 명확히 설명하거나 스레드  안전성 애너테이션 사용) synchronized 한정자는 문서화와 관련이 없다. 조건부 스레드 안전 클래스는 메서드를 어떤 순서로 호출할때 외부 동기화가 요구되고, 그때 어떤 락을 얻어야 하는지도 알려줘야 한다. 무조건적 스레드 안전 클래스를 작성할 때는 synchrnonized 메서드가 아닌 비공개 락 객체를 사용하자. 이렇게 해야 클라이언트 하위 클래스에서 동기화 메커니즘을 깨뜨리는 걸 예방할 수 있고, 더 정교한 동시성 제어 메커니즘으로 재구현할 여지가 생긴다.



