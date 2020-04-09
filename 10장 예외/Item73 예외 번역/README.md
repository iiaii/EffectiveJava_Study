# #EffectiveJava/10장_예외/73예외번역


## 73. 추상화 수준에 맞는 예외를 던지라


수행하려는 일과 관련 없어 보이는 예외가 튀어나오면 당황스러울 것이다. 메서드가 저수준 예외를 처리하지 않고 바깥으로 전파해버릴 때 종종 일어나는 일이다. 이는 프로그래머를 당황시킬 뿐 아니라, 내부 구현 방식을 드러내어 윗 레벨 API를 오염시킨다. (다음 릴리스에서 구현 방식을 바꾸면 다른 예외가 튀어나와 기존 클라이언트 프로그램을 깨지게 할 수도 있음)

이 문제를 피하기 위해서는 **상위 계층에서는 저수준 예외를 잡아 자신의 추상화 수준에 맞는 예외로 바꿔 던져야 한다** -> 예외 번역

```java
// 예외 번역
try {
	… // 저수준 추상화를 이용한다.
} catch(LowerLevelException e) {
	// 추상화 수준에 맞게 번역한다.
	throw new HigherLevelException(…);
}
```


다음은 AbstractSequentialList에서 수행하는 예외 번역의 예다. AbstractSequentialList는 List 인터페이스의 골격 구현(아이템20)이다. 이 예에서 수행한 예외 번역은 List<E> 인터페이스의 get 메서드 명세에 명시된 필수사항임을 기억해두자.

```java
/**
 * 이 리스트 안의 지정한 위치의 원소를 반환한다.
 * @throws IndexOutOfBoundsException index가 범위 밖이라면,
 * 		 즉 ({@code index < 0 || index >= size()})이면 발생한다
 */
public E get(int index) {
	List<E> i = listIterator(index);
	try {
		return i.next();
	} catch(NoSuchElementException e) {
		throw new IndexOutOfBoundsException(“인덱스: “+index);
	}
}
```

예외를 번역할 때, 저수준 예외가 디버깅에 도움이 된다면 예외 연쇄(exception chaining)를 사용하는게 좋다. (예외 연쇄란 문제의 근본원인인 저수준 예외를 고수준 예외에 실어 보내는 방식이다. 별도의 접근자 메서드(Throwable의 getCause메서드)를 통해 필요하면 언제든 저수준 예외를 꺼내 볼 수 있다.

```java
// 예외 연쇄
try {
	… // 저수준 추상화를 이용한다.
} catch(LowerLevelException cause) {
	// 	저수준 예외를 고수준 예외에 실어 보낸다.
	throw new HigherLevelException(cause);
}
```

고수준 예외의 생성자는 (예외 연쇄용으로 설계된) 상위 클래스의 생성자에 이 원인을 건네주어, 최종적으로 Throwable(Throwable) 생성자까지 건네지게 한다.

```java
// 예외 연쇄용 생성자
class HigherLevelException extends Exception {
	HigherLevelException(Throwable cause) {
		super(cause);
	}
}
```

대부분의 표준 예외는 예외 연쇄용 생성자를 갖추고 있다. 
무턱대고 예외를 전파하기보다 예외번역이 우수한 방법이지만 남용하면 안된다. 가능하면 저수준 메서드가 반드시 성공하도록 하여 아래 계층에서는 예외가 발생하지 않도록 하는 것이 최선이다. 때로는 상위 계층 메서드의 매개변수 값을 아래 계층 메서드로 건네기 전에 미리 검사하는 방법으로 이 목적을 달성할 수 있다.

아래 계층에서 예외를 피할 수 없다면, 상위 계층에서 그 예외를 조용히 처리하여 문제를 API 호출자까지 전파하지 않는 방법이 있다. 이 경우 발생한 예외는 java.util.logging 같은 적절한 로깅 기능으로 기록해두면 좋다. (클라이언트 코드와 사용자에게 문제를 전파하지 않으면서도 로그 분석으로 추가 조치가 가능하다)


> **::핵심 정리::** 
> 아래 계층의 예외를 예방하거나 스스로 처리할 수 없고, 그 예외를 상위 계층에 그대로 노출하기 곤란하다면 예외 번역을 사용하라. 이때 예외 연쇄를 이용하면 상위 계층에는 맥락에 어울리는 고수준 예외를 던지면서 근본 원인도 함께 알려주어 오류를 분석하기에 좋다 (아이템75)






