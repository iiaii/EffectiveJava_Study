# #EffectiveJava/8장_메서드/49매개변수유효성검사

## 49. 매개변수가 유효한지 검사하라

메서드와 생성자 대부분은 입력 매개변수의 값이 특정 조건을 만족하기를 바란다. (예를 들면 인덱스 값은 음수면 안되고, 객체 참조는 null이 아니어야 하는 식) 이런 제약은 반드시 문서화해야 하며 메서드 몸체가 시작되기 전에 검사해야 한다. 
-> 오류는 가능한 한 빨리 (발생한 곳에서) 잡아야 한다는 일반 원칙의 한 사례임

메서드 몸체가 실행되기 전에 매개변수를 확인하면 잘못된 값이 넘어왔을때 즉각적이고 깔끔한 방식으로 예외를 던질 수 있다. 
매개변수 검사를 제대로 하지 못하면 메서드가 수행되는 중간에 모호한 예외를 던지며 실패할 수 있다. (-> 더 나쁜 상황은 잘못된 결과를 반환하는것 -> 더 더 나쁜상황은 어떤 객체를 이상한 상태로 만들어 놓아서 미래의 알 수 없는 시점에 이 메서드와는 관련없는 오류를 내는 경우, 즉 실패 원자성을 어기는 결과를 낳음)


public 과 protected 메서드는 매개변수 값이 잘못됐을 때 던지는 예외를 문서화해야 한다. (@throws 자바독 태그를 사용) 보통은 IllegalArgumentException, NullPointerException 중 하나가 될 것이다. 매개변수의 제약을 문서화하면 그 제약을 어겼을 때 발생하는 예외도 함께 기술해야 한다. 이런 간단한 방법으로 API 사용자가 제약을 지킬 가능성을 크게 높일 수 있다.

```java
/**
 * (현재 값 mod m) 값을 반환한다. 이 메서드는
 * 항상 음이 아닌 BigInteger를 반환한다는 점에서 remainder 메서드와 다르다
 * 
 * @param m 계수 (양수여야 한다.)
 * @return 현재 값 mod m
 * @throws ArithmeticException m이 0보다 작거나 같으면 발생한다.
 */
public BigInteger mod(BigInteger m) {
	if (m.signum() <= 0)
		throw new ArithmeticException(“계수(m)는 양수여야 함 ”+m);
  … // 계산 수행
}
```

이 메서드는 m이 null이면 m.signum() 호출 때 NullPointerException을 던진다. 이 것은 BigInteger 클래스 수준에서 기술했기 때문에 여기서는 설명이 없다. 


자바 7에 추가된 java.util.Objects.requireNonNull 메서드는 유연하고 사용하기도 편해서 더 이상 null 검사를 수동으로 하지 않아도 된다. 원하는 예외 메시지도 지정 가능하고 입력을 그대로 반환하기때문에 값을 사용하는 동시에 null 검사를 수행할 수 있다.

```java
// 자바의 null 검사 기능 사용하기
this.strategy = Objects.requireNonNull(strategy, “전략”);
```

반환값은 그냥 무시하고 필요한곳 어디서든 순수한 null 검사 목적으로 사용해도 된다. 

자바 9에서는 Objects에 범위 검사 기능도 더해졌다. checkFromInsidexSize, checkFromToIndex, checkIndex 라는 메서드인데 null 검사 메서드만큼 유연하지 않다. 예외 메시지를 지정할 수 없고, 리스트와 배열 전용으로 설계됐다. 또한 닫힌 범위는 다루지 못한다. (양 끝단 값을 포함) 

공개되지 않은 메서드라면 패키지 제작자인 개발자가 메서드가 호출되는 상황을 통제할 수 있다. 따라서 오직 유효한 값만이 메서드에 넘겨지리라는 것을 보증할 수 있도록 해야 한다. (public이 아닌 메서드라면 단언문(assert)을 사용해 매개변수 유효성을 검증할 수 있다)

```java
// 재귀 정렬용 private 도우미 함수
private static void sort(long a[], int offset, int length) {
	assert a != null;
	assert offset >= 0 && offset <= a.length;
	assert length >= 0 && length <= a.length - offset;
  … // 계산 수행 
}
```

이 단언문들은 자신이 단언한 조건이 무조건 참이라고 선언한다. 이 메서드가 포함되니 패키지를 클라이언트가 어떤 식으로 하든 상관없다. 단언문은 몇 가지 면에서 일반적인 유효성 검사와 다르다. 
1. 실패하면 AssertionError를 던진다. 
2. 런타임에 아무런 효과도, 아무런 성능 저하도 없다 (단, java를 실행할 때 명렬줄에서 -ea 혹은 --enableassertions 플래그를 설정하면 런타임에 영향을 줌)

메서드가 직접 사용하지는 않으나 나중에 쓰기 위해 저장하는 매개변수는 특히 더 신경 써서 검사해야 한다. (디버깅하기 매우 어려워 질 수 있음)

생성자는 [ 나중에 쓰려고 저장하는 매개변수의 유효성을 검사하라 ]는 원칙의 특수한 사례다. 생성자 매개변수의 유효성 검사는 클래스 불변식을 어기는 객체가 만들어지지 않게 하는데 꼭 필요하다. 
 메서드 몸체 실행 전에 매개변수 유효성을 검사해야 한다는 규칙에도 예외는 있다. 유효성 검사 비용이 지나치게 높거나 실용적이지 않을 때, 혹은 계산과정에서 암묵적으로 검사가 수행될 때다. (예를 들어 Collections.sort(List) 처럼 객체 리스트를 정렬하는 메서드를 생각해보면 리스트 안의 객체들은 모두 상호 비교될 수 있어야 하며, 정렬 과정에서 이 비교가 이뤄진다. 만약 상호 비교될 수 없는 타입은 ClassCastException을 던지는데 모두 객체를 확인하는 것은 실익이 없다. 하지만 실패 원자성을 어기는 것에 주의해야 한다)


때로는 계산 과정에서 필요한 유효성 검사가 이뤄지지만 실패했을 때 잘못된 예외를 던지기도 한다. 계산 중 잘못된 매개변수 값을 사용해 발생한 예외와 API 문서에서 던지기로 한 예외가 다를 수 있다는 뜻이다. 이런 경우에는 아이템 API 문서에서 던지기로 한 예외가 다를 수 있다.

이번 아이템을 매개변수에 제약을 두는게 좋다고 해석해서는 안된다. 오히려 **메서드는 최대한 범용적으로 설계해야 한다.** 메서드가 건네 받은 값으로 무언가 제대로 된 일을 할 수 있다면 매개변수 제약은 적을수록 좋다. 하지만 구현하려는 개념 자체가 특정한 제약을 내재한 경우도 드물지 않다.


::핵심 정리:: 

> 메서드나 생성자를 작성할 때면 그 매개변수들에 어떤 제약이 있을지 생각해야 한다. 그 제약들을 문서화하고 메서드 코드 시작 부분에서 명시적으로 검사해야 한다. 이런 습관을 기르면 유효성 검사가 실제 오류를 걸러냄으로서 보상 받게 된다.


