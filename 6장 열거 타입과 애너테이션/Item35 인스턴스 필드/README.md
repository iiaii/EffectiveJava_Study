# #EffectiveJava/6장_열거타입과애너테이션/35인스턴스필드

## 35. ordinal 메서드 대신 인스턴스 필드를 사용하라

대부분의 열거 타입 상수는 자연스럽게 하나의 정수값에 대응된다. 그리고 모든 열거 타입은 해당 상수가 그 열거 타입에서 몇 번째 위치인지를 반환하는 ordinal이라는 메서드를 제공한다.

```java
// ordinal을 잘못 사용한 예 - 따라하지 말 것!
public enum Ensemble {
	SOLO, DUET, TRIO, QUARTET, QUINTET, SEXTET, SEPTET, OCTET, NONET, DECTET;

	public int numberOfMusicians() { return ordianl()+1; }
}
```

동작하지만 유지보수하기 힘들다. 상수 선언 순서를 바꾸는 순간 numberOfMusicians가 오동작하며, 이미 사용중인 정수와 값이 같은 상수는 추가할 방법이 없다. (8중주 상수가 있으므로 복4중주는 추가 불가)

또한 중간의 값을 비워 놓을 수도 없기 때문에 더미 상수를 추가해야만 하는 문제가 발생한다.

```java
public enum Ensemble {
	SOLO(1), DUET(2), TRIO(3), QUARTET(4), QUINTET(5), SEXTET(6), SEPTET(7), OCTET(8), DOUBLE_QUARTET(8), NONET(9), DECTET(10), TRIPLE_QUARTET(12);

	private final int numberOfMusicians;
	Ensemble(int size) { this.numberOfMusicians = size; }
	public int numberOfMusicians() { return numberOfMusicians; }
}
```

Enum의 API문서를 보면 ordinal에 대해 이렇게 쓰여 있다. “대부분의 프로그래머는 이 메서드를 쓸 일이 없다. 이 메서드는 EnumSet과 EnumMap 같이 열거 타입기반 범용 자료구조에 쓸 목적으로 설계되었다”

