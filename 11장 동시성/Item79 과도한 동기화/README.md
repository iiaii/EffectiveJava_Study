# #EffectiveJava/11장_동시성/79과도한동기화


## 79. 과도한 동기화는 피하라


아이템78 과 반대로 과도한 동기화는 성능을 떨어뜨리고, 교착상태에 빠뜨리고, 예측할 수 없는 동작을 낳기도 한다.

##### 정확성

**응답 불가와 안전 실패를 피하려면 동기화 메서드나 동기화 블록 안에서는 제어를 절대로 클라이언트에 양도하면 안 된다.** 
 예를 들어 동기화된 영역 안에서는 재정의할 수 있는 메서드는 호출하면 안되며, 클라이언트가 넘겨준 함수 객체(아이템24)를 호출해서도 안된다. 동기화된 영역을 포함한 클래스 관점에서는 이런 메서드는 무슨 일을 할지 모르며 통제할 수도 없다. 외계인 메서드가 하는 일에 따라 동기화된 영역은 예외를 일으키거나, 교착상태에 빠지거나, 데이터를 훼손할 수도 있다.

 다음은 어떤 집합을 감싼 래퍼클래스고 이 클래스의 클라이언트는 집합에 원소가 추가되면 알림을 받을 수 있다. (관찰자 패턴, 핵심만 보여주고자 원소가 제거될 때 알려주는 기능은 생략, 아이템18에서 사용한 ForwaidingSet을 재사용) 

```java
public class ObservableSet<E> extends ForwardingSet<E> {
	public ObservableSet(Set<E> set) { super(set); }

	private final List<SetObserver<E>> observers
			= new ArrayList<>();

	public void addObserver(SetObserver<E> observer) {
		synchronized(observers) {
			observers.add(observer);
		}
	}
	
	public boolean removeObserver(SetObserver<E> observer) {
		synchronized(observers) {
			return observers.remove(observer);
		}
	}

	// 주목!
	public void notifyElementAdded(E element) {
		synchronized(observers) {
			return oblservers.remove(observer);
		}
	}

	@Override public boolean add(E element) {
		boolean added = super.add(element);
		if(added)
			notify(ElementAdded(element);
		return added;
	}

	@Override public boolean addAll(Collection<? extends E> c) {
		boolean result = false;
		for(E element : c)
			result |= add(element); // notifyElement를 호출
		return result;
	}
}
```

관찰자들은 addObserver와 removeObserver 메서드를 호출해 구독을 신청하거나 해지한다. 두 경우 모두 다음 콜백 인터페이스의 인스턴스를 메서드에 건넨다.

```java
// 구조적으로 BiConsumer<ObservableSet<E>,E>와 같음
// 보다 직관적이고 다중 콜백을 지원하기 위함
@FunctionalInterface public interface SetObserver<E> {
	// ObservableSet에 원소가 더해지면 호출된다.
	void added(ObservableSet<E> set, E element);
}
```




```java
// 다음 프로그램은 0~99까지만 출력
public static void main(String[] args) {
	ObservableSet<Integer> set =
			new ObservableSet<>(new HashSet<>());

	set.addObserver((s,e) -> System.out.println(e));

	for(int i=0 ; i<100 ; i++)
		set.add(i);
}


// 평상시에는 집합에 추가된 정숫값을 출력하다가, 그 값이 23이면 자신을 제거
set.addObserver(new SetObserver<>() {
	public void added(ObservableSet<Integer> s, Integer e) {
		System.out.println(e);
		if(e == 23)
			s.removeObserver(this);
	}
});
```

> 람다를 사용한 이전 코드와 달리 익명클래스를 사용함. s.removeObserver 메서드에 함수 객체 자신을 넘겨야 하기 때문. 람다는 자신을 참조할 수단이 없다(아이템42)

위 프로그램은 23까지 출력하고 관찰자 자신을 구독해지한 다음 종료하는 것이 아니라 23까지 출력한 뒤 ConcurrentModificationException 을 던진다. 관찰자의 added메서드 호출이 일어난 시점이 notifyElementAdded가 관찰자들의 리스트를 순회하는 도중이기 때문이다. added 메서드는 ObservableSet의 removeObserver 메서드를 호출하고, 이 메서드는 다시 observers.remove 메서드를 호출한다. (여기서 문제가 발생) 리스트에서 원소를 제거하려 하는데, 마침 지금은 이 리스트를 순회하는 도중이다. 허용되지 않은 동작이다. notifyElementAdded 메서드에서 수행하는 순회는 동기화 블록 안에 있으므로 동시 수정이 일어나지 않도록 보장하지만, 정작 자신이 콜백을 거쳐 되돌아와 수정하는 것 까지 막지는 못한다.


```java
// 구독해지를 하는 관찰자를 실행자 서비스를 이용해 다른 스레드에게 부탁
// 쓸데없이 백그라운드 스레드를 사용하는 관찰자
set.addObserver(new SetObserver<>() {
	public void added(ObservableSet<Integer> s, Integer e) {
		System.out.println(e);
		if(e == 23) {
			ExcutorService exec =
				Excutors.newSingleThreadExecutor();
			try {
				exec.submit(() -> s.removeObserver(this)).get();
			} catch(ExecturionException | InterrupedException ex) {
				throw new AssertionError(ex);
			} finally {
				exec.shutdown();
			}
		}
	}
});
```

> 이 프로그램은 catch 구문 하나에서 두가지 예외를 잡고 있다. 다중 캐치라고도 하는 이 기능은 자바7부터 지원. (똑같이 처리해야 하는 예외가 여러개일 때 프로그램의 크기를 줄이고 코드 가독성을 크게 개선)

위 프로그램은 예외 없이 교착상태에 빠지게 된다. 백그라운드 스레드가 s.removeObserver를 호출하면 관찰자를 잠그려 시도하지만 락을 얻을 수 없다. (메인 스레드가 락을 쥐고 있기 때문, 그와 동시에 메인 스레드는 백그라운드 스레드가 관찰자를 제거하기만을 기다림) 


자바 언어의 락은 재진입을 허용하므로 교착상태에 빠지지 않는다. 예외를 발생시킨 첫 번째 예에서라면 외계인 메서드를 호출하는 스레드는 이미 락을 쥐고 있으므로 다음번 락 획득도 성공한다. (그 락이 보호하는 데이터에 대해 개념적으로 관련이 없는 다른 작업이 진행중이어도) -> 문제가 될 수 있음

문제의 주 원인을 락이 제 구실을 하지 못했기 때문이다. 재진입 가능 락은 객체 지향 멀티스레드 프로그램을 쉽게 구현할 수 있도록 해주지만, 응답 불가(교착상태)가 될 상황을 안전 실패(데이터 훼손)로 변모시킬 수도 있다.


외계인 메서드 호출을 동기화 블록 바깥으로 옮기면 된다. notifyElementAdded 메서드에서라면 관찰자 리스트를 복사해 쓰면 락 없이도 안전하게 순회할 수 있다. 이 방식을 적용하면 앞서의 두 예제에서 예외 발생과 교착상태 증상이 사라진다.

```java
// 외계인 메서드를 동기화 블록 바깥으로 옮겼다.
private void notifyElementAdded(E element) {
	List<SetObserver<E>> snapshot = null;
	synchronized(observers) {
		snapshot = new ArrayList<>(observers);
	}
	for(SetObserver<E> observer : snapshot)
		observer.added(this, element);
}
```

더 나은 방법은 자바의 동시성 컬렉션 라이브러리의  CopyOnWriteArrayList 가 정확히 이 목적으로 특별히 설계되었다. 이름이 말해주듯 ArrayList를 구현한 클래스로, 내부를 변경하는 작업은 항상 깨끗한 복사본을 만들어 수행하도록 구현했다. 내부 배열은 절대 수정되지 않으니 순회할 때 락이 필요 없어 매우 빠르다. (다른 용도로 사용한다면 CopyOnWriteArrayList 는 많이 느리지만, 수정할 일이 드물고 순회만 빈번히 일어나는 관찰자 리스트 용도로는 최적이다)


ObservableSet을 CopyOnWriteArrayList 를 사용해 다시 구현하면 메서드들은 다음처럼 바꾸면 된다(add와 addAll 메서드는 수정할게 없다)

```java
// CopyOnWriteArrayList를 사용해 스레드 안전하고 관찰 가능한 집합
private final List<SetObserver<E>> observers = 
		new CopyOnWirteArrayList<>();

public void addObserver(SetObserver<E> observer) {
	observers.add(observer);
}

public boolean removeObserver(SetObserver<E> observer) {
	return observers.remove(observer);
}

private void notifyElementAdded(E element) {
	for(SetObserver<E> observer : observers)
		observer.added(this, element);
}
```

동기화 영역 바깥에서 호출되는 외계인 메서드를 열린 호출이라 한다. 외계인 메서드는 얼마나 오래 실행될지 알 수 없는데, 동기화 영역 안에서 호출된다면 그동안 다른 스레드는 보호된 자원을 사용하지 못하고 대기해야만 한다. 따라서 열린 호출은 실패 방지 효과 외에도 동시성 효율을 크게 개선해준다.

기본 규칙은 동기화 영역에서는 가능한 한 일을 적게하는 것이다. (락을 얻고 공유데이터를 검사하고 필요하면 수정하고 락을 놓는다) 오래걸리는 작업이라면 아이템78 지침을 어기지 않으면서 동기화 영역 바깥으로 옮기는 방법을 찾아보자.


##### 성능

동기화 비용이 빠르게 낮아져 왔지만 과도한 동기화를 피하는 것이 중요하다. 과도한 동기화가 초래하는 진짜 비용은 락을 얻는데 드는 CPU시간이 아닌, 경쟁하느라 낭비하는 시간(병렬로 실행할 기회를 잃고, 모든 코어가 메모리를 일관되게 보기 위한 지연시간)이다. 가상머신의 코드 최적화를 제한한다는 점도 동기화의 숨은 비용이다.


가변클래스를 작성할때 다음 2가지중 하나를 따르자

1. 동기화를 전혀하지 말고, 그 클래스를 동시에 사용해야 하는 클래스가 외부에서 알아서 동기화하게 하자.
2. 동기화를 내부에서 수행해 스레드 안전한 클래스로 만들자(아이템82) (단, 클라이언트가 외부에서 객체 전체에 락을 거는 것보다 동시성을 월등히 개선할 수 있을때만 선택)


StringBuffer 인스턴스는 거의 단일 스레드에서 쓰였음에도 내부적으로 동기화를 수행했다. (뒤늦게 StringBuilder가 등장한 이유, java.util.Random -> java.util.concurrent.ThreadLocalRandom (동기화 안하는 버전)) 선택하기 어려우면 스레드 안전하지 않다고 명기하면 된다.


클래스를 내부에서 동기화하기로 했으면 락 분할, 락 스트라이핑, 비차단 동시성 제어(nonblocking concurrency control) 등 다양한 기법을 동원해 동시성을 높여줄 수 있다.

여러 스레드가 호출할 가능성이 있는 메서드가 정적 필드를 수정한다면 그 필드를 사용하기 전에 반드시 동기화 해야 한다(비결정적 행동도 용인하는 클래스는 상관 없음)


> **::핵심 정리::** 
> 교착상태와 데이터 훼손을 피하려면 동기화 영역 안에서 외계인 메서드를 절대 호출하지 말자. **일반화해 이야기하면 동기화 영역 안에서의 작업은 최소한으로 줄이자**. 가변 클래스를 설계할 때는 스스로 동기화해야 할지 고민하자. 멀티코어 세상인 지금은 과도한 동기화를 피하는게 어느 때보다 중요하다 합당한 이유가 있을때 내부에서 동기화하고, 동기화했는지 여부를 문서에 명확히 밝히자(아이템82)



