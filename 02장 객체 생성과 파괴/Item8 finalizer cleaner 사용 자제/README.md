# #EffectiveJava/2장_객체생성과파괴/8finalizer와cleaner사용을피하라

## 8. finalizer와 cleaner 사용을 피하라

자바는 2가지 객체 소멸자를 제공한다. **finalizer는 예측할 수 없고, 상황에 따라 위험할 수 있어 일반적으로 불필요하다.** 오동작, 낮은 성능, 이식성 문제의 원인이 되기도 한다. deprecate API로 지정되고 cleaner가 그 대안으로 나왔지만 **cleaner는 finalizer보다는 덜 위험하지만, 여전히 예측할 수 없고 느리고 일반적으로 불필요하다**

finalizer와 cleaner는 즉시 수행된다는 보장이 없다. 객체에 접근할 수 없게 된 후 finalizer나 cleaner가 실행되기까지 얼마나 걸릴지 알 수 없다. **즉, finalizer와 cleaner로는 제때 실행되어야 하는 작업은 절대 할 수 없다.

상태를 영구적으로 수정하는 작업에서는 절대 finalizer나 cleaner에 의존해서는 안된다.

심각한 성능문제도 동반한다.

finalizer를 사용한 클래스는 finalizer 공격에 노출되어 심각한 보안 문제를 일으킬 수도 있다. 

객체 생성을 막으려면 생성자에서 예외를 던지는 것만으로 충분하지만, finalizer가 있다면 아니다.

final이 아닌 클래스를 finalizer공격으로부터 방어하려면 아무 일도 하지 않는 finalize 메서드를 만들고 final로 선언한다.

AutoCloseable을 구현해주고, 클라이언트에서 인스턴스를 다 쓰고 나면 close()메서드를 호출하면 된다. (일반적으로 예외가 발생해도 제대로 종료되도록 try-with-resources를 사용해야 함)

```java
public class Room implements AutoCloseable {
	private static final Cleaner cleaner = Cleaner.create();

	//	청소가 필요한 자원. 절대 Room을 참조해서는 안된다!
	private static class State implements Runnable {
		int numJunkPiles; // 방 안의 쓰레기 수

	State(int numJunkPiles) {
		this.numJunkPiles = numJunkPiles;
	}

	// close 메서드나 cleaner가 호출한다.
	@Override
	public void run() {
		System.out.println(“방 청소”);
		numJunkPiles = 0;
	}

	// 방의 상태. cleanable과 공유한다
	private final State state;

	// cleanable 객체. 수거 대상이 되면 방을 청소한다.
	private final Cleaner.Cleanable cleanable;

	public Room(int numJunkPiles) {
		state = new State(numJunkPiles);
		cleanable = cleaner.register(this, state);
	}

	@Override
	public void close() {
		cleanable.clean();
	}
}
```

System.exit을 호출할 때의 cleaner동작은 구현하기 나름이다. 청소가 이뤄질지는 보장하지 않는다.


::핵심 정리::

> cleaner는 안전망 역할이나 중요하지 않은 네이티브 자우너 회수용으로만 사용하자. 물론 이런 경우라도 불확실성과 성능 저하에 주의해야 한다.


