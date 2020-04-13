# #EffectiveJava/11장_동시성/78가변데이터동기화


## 78. 공유 중인 가변 데이터는 동기화해 사용하라

synchronized 키워드는 해당 메서드나 블록을 한번에 한 스레드씩 수행하도록 보장한다. 많은 프로그래머가 동기화를 배타적 실행, 즉 한 스레드가 변경하는 중이라서 상태가 일관되지 않은 순간의 객체를 다른 스레드가 보지 못하게 막는 용도로만 생각한다.

한 객체가 일관된 상태를 가지고 생성되고, 이 객체에 접근하는 메서드는 그 객체에 락을 건다. 락을 건 메서드는 객체의 상태를 확인하고 필요하면 수정한다. 객체를 하나의 일관된 상태에서 다른 일관된 상태로 변화시킨다. 동기화를 제대로 사용하면 어떤 메서드도 이 객체의 상태가 일관되지 않은 순간을 볼 수 없을 것이다.

맞는 설명이지만, 동기화에는 중요한 기능이 하나 더 있다. 동기화 없이는 한 스레드가 만든 변화를 다른 스레드에서 확인하지 못할 수 있다. 동기화는 일관성이 깨진 상태를 볼 수 없게 하는 것은 물론, 동기화된 메서드나 블록에 들어간 스레드 같은 락의 보호하에 수행된 모든 이전 수정의 최종 결과를 보게 해준다. 

언어 명세상 long과 double은 변수를 읽고 쓰는 동작이 원자적이다. 여러 스레드가 같은 변수를 동기화 없이 수정하는 중이라도, 항상 어떤 스레드가 정상적으로 저장한 값을 온전히 읽어옴을 보장한다.

여기서 “성능을 높이려면 원자적 데이터를 읽고 쓸 때는 동기화하지 말아야겠다”고 생각하기 쉬운데, 아주 위험하다. 자바 언어 명세는 스레드가 필드를 읽을 때 항상 수정이 완전히 반영된 값을 얻는다고 보장하지만, 한 스레드가 저장한 값이 다른 스레드에게 보이는가는 보장하지 않는다. **동기화는 배타적 실행뿐 아니라 스레드 사이의 안정적인 통신에 꼭 필요하다.**(한 스레드가 만든 변화가 다른 스레드에게 언제 어떻게 보이는지를 규정한 자바의 메모리 모델 때문이다)

공유중인 가변 데이터를 비록 원자적으로 읽고 쓸 수 있을지라도 동기화에 실패하면 처참한 결과로 이어질 수 있다. 다른 스레드를 멈추는 작업을 생각해 보면, Thread.stop 메서드는 안전하지 않아 deprecate API로 지정되었다. 그러니 **Thread.stop 은 사용하지 말자!**


 다른 스레드를 멈추는 올바른 방법은

1. 스레드는 자신의 boolean 필드를 폴링하면서 그 값이 true가 되면 멈춘다. 

이 필드를 false로 초기화해놓고, 다른 스레드에서 이 스레드를 멈추고자 할 때 true로 변경하는 식이다. boolean 필드를 읽고 쓰는 작업은 원자적이라 어떤 프로그래머는 이런 필드에 접근할 때 동기화를 제거하기도 한다.

```java
// 잘못된 코드 - 이 프로그램은 얼마나 오래 실행될까?
public class StopThread {
	private static boolean stopRequested;

	public static void main(String[] args) 
			throws InterruptedException {
		Thread backgroundThread = new Thread(() -> {
			int i = 0;
			while(!stopRequested)
				i++;
		});
		backgroundThread.start();

		TimeUnit.SECONDS.sleep(1);
		stopRequested = true;
	}
}
```
-> 메인 스레드가 1초후 stopRequested를 true로 설정하면 backgroundThread는 반복문을 빠져나올 것처럼 보이지만, 영원히 수행된다.

원인은 동기화에 있다. 동기화하지 않으면 메인 스레드가 수정한 값을 백그라운드 스레드가 언제쯤에나 보게 될지 보증할 수 없다. 동기화가 빠지면 가상머신이 다음과 같은 최적화를 수행할 수도 있는 것이다.

```java
// 원래 코드
while(!stopRequested)
	i++;

// 최적화 코드
if (!stopRequested)
	while(true)
		i++;
```

OpenJDK 서버 VM이 실제로 적용하는 호이스팅(끌어올리기hoisting) 최적화 기법이다. 이 결과 프로그램은 응답 불가 상태가 되어 더 이상 진전이 없다. stopRequested 필드를 동기화해 접근하면 문제가 해결된다

```java
// 적절히 동기화해 스레드가 1초후 정상 종료한다.
public class StopThread {
	private static boolean stopRequested;

	private static synchronized void requestStop() {
		stopRequested = true;
	}

	private static synchronized boolean stopRequested() {
		return stopRequested;
	}

	public static void main(String[] args)
			throws InterruptedException {
		Thread backgroundThread = new Thread(() -> {
			int i = 0;
			while(!stopRequested())
				i++;
		});
		backgroundThread.start();

		TimeUnint.SECONDS.sleep(1);
		requestStop();
	}
}
```

쓰기 메서드(requestStop)와 읽기 메서드(stopRequested) 모두를 동기화되야지만 동작을 보장한다. (둘중 하나만 동기화해도 동작하는 듯 보이지만 속아서는 안된다)
(사실 이 두 메서드는 단순화해서 동기화 없이도 원자적으로 동작한다. 앞서 이야기했듯 동기화는 배타적 수행과 스레드 간 두 통신이라는 두가지 기능을 수행하는데, 통신 목적으로만 사용되었다)


```java
// volatile 필드를 사용해 스레드가 정상 종료한다.
public class StopThread {
	private static volatile boolean stopRequested;

	public static void main(String[] args)
			throws InterruptedException {
		Thread backgroundThread = new Thread(() -> {
			int i = 0;
			while(!stopRequested)
				i++;
		});
		backgroundThread.start();

		TimeUnit.SECONDS.sleep(1);
		stopRequested = true;
	}
}
```

하지만 volatile은 주의해서 사용해야 한다.

```java
// 잘못된 코드 - 동기화가 필요하다!
private static volatile int nextSerialNumber = 0;

public static int generateSerialNumber() {
	return nextSerialNumber++;
}
```
-> 매번 고유한 값을 반환할 의도로 만들어졌다.(값이 2^32을 넘어가지 않으면) 이 메서드의 상태는 nextSerialNumber라는 단 하나의 필드로 결정되는데, 원자적으로 접근할 수 있고 어떤 값이든 허용한다. 따라서 굳이 동기화하지 않더라도 불변식을 보호할 수 있어 보인다. 하지만 이 역시 동기화 없이는 올바로 동작하지 않는다.

문제는 증가 연산자(++)다. 이 연산자는 코드상으로는 하나지만 실제로는 nextSerialNumber 필드에 두번 접근한다. 먼저 값을 읽고, 그런 다음 1 증가한 값을 저장하는 것이다. 만약 두 번째 스레드가 이 두 접근 사이를 비집고 들어와 값을 읽으면 첫 번째 스레드와 똑같은 값을 돌려받게 된다. (프로그램이 잘못된 결과를 계산해내는 이런 오류를 안전실패(safety failure)라고 한다)

generateSerialNumber 메서드에 synchronized 한정자를 붙이면 문제가 해결된다. 동시에 호출해도 서로 간섭하지 않으면 이전 호출이 변경한 값을 읽게 된다. 메서드에 synchronized를 붙이면 nextSerialNumber 필드에서는 volatile을 제거해야 한다. 이 메서드를 더 견고하게 하려면 int 대신 long을 사용하거나 nextSerialNumber가 최댓값에 도달하면 예외를 던지게 하자.


아이템59의 조언을 따라 java.util.concurrent.atomic 패키지의 AtomicLong을 사용해보자. 이 패키지에는 락이 없이도(lock-free) 스레드 안전한 프로그래밍을 지원하는 클래스들이 담겨 있다. volatile은 동기화의 두 효과 중 통신 쪽만 지원하지만 이 패키지는 원자성(배타적 실행)까지 지원한다. 우리가 generateSerialNumber에 원하는 바로 그 기능이다. 성능도 동기화 버전보다 우수!!

```java
// java.util.concurrent.atomic을 이용한 락-프리 동기화
private static final AtomicLong nextSerialNum = new AtomicLong();

public static long generateSerialNumber() {
	return nextSerialNum.getAndIncrement();
}
```


이번 아이템에서 언급한 문제들을 피하는 가장 좋은 방법은 가변 데이터를 공유하지 않는 것이다. 불변 데이터(아이템17)만 공유하거나 아무것도 공유하지 말자. 다시 말해 가변 데이터는 단일 스레드에서만 쓰도록 하자. 또한, 사용하는 프레임워크와 라이브러리를 깊이 이해하는 것도 중요하다. 이런 외부 코드가 인지하지 못한 스레드를 수행하는 복병으로 작용하는 경우도 있다.

한 스레드가 데이터를 다 수정한 후 다른 스레드에 공유할 때는 해당 객체에서 공유하는 부분만 동기화해도 된다. 그러면 그 객체를 다시 수정할 일이 생기기 전까지 다른 스레드들은 동기화 없이 자유롭게 값을 읽어갈 수 있다. 이런 객체를 사실상 불변이라고하고 다른 스레드에 이런 객체를 건네는 행위를 안전 발행이라 한다. 
-> 객체를 안전하게 발행하는 방법은 클래스 초기화 과정에서 객체를 정적 필드, volatile 필드, final 필드, 보통의 락, 동시성 컬렉션 을 통해 접근하는 필드에 저장해도 된다. 


> **::핵심 정리::** 
> 여러 스레드가 가변 데이터를 공유한다면 그 데이터를 읽고 쓰는 동작은 반드시 동기화해야 한다. 동기화하지 않으면 한 스레드가 수행한 변경을 다른 스레드가 보지 못할 수도 있다. 공유되는 가변 데이터를 동기화하는데 실패하면 응답 불가 상태에 빠지거나 안전 실패로 이어질 수 있다. 이는 디버깅 난이도가 가장 높은 문제에 속한다. (간헐적이거나 특정 타이밍에만 발생하기도하고, VM에 따라 현상이 달라지기도 함) 배타적 실행은 필요없고 스레드끼리의 통신만 필요하다면 volatile 한정자만으로 동기화할 수 있다. (올바로 사용하기 까다롭기는 함)







