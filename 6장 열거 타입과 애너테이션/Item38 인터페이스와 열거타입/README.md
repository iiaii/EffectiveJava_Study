# #EffectiveJava/6장_열거타입과애너테이션/38인터페이스와열거타입

## 38. 확장할 수 있는 열거 타입이 필요하면 인터페이스를 사용하라

열거 타입은 몯느 상황에서 타입 안전 열거 패턴보다 우수하지만, 타입 안전 열거 패턴은 확장 할 수 있으나 열거 타입은 그럴 수 없다.
-> 타입 안전 열거 패턴은 열거한 값들을 그대로 가져온 다음 값을 더 추가하여 다른 목적으로 쓸 수 있지만 열거타입은 그럴 수 없다

연산코드와 같은 경우에는 확장 할 수 있는 열거 타입이 어울린다.
```java
public interface Operation {
	double apply(double x, double y);
}

public enum BasicOperation implements Operation {
	PlUS(“+”) {
		public double apply(double x, double y) { return x+y; }
	},
	MINUS(“-”) {
		public double apply(double x, double y) { return x-y; }
	},
	TIMES(“*”) {
		public double apply(double x, double y) { return x*y; }
	},
	DIVIDE(“/”) {
		public double apply(double x, double y) { return x/y; }
	},

	private final String symbol;

	BasicOperation(String symbole) {
		this.symbol = symbol;
	}

	@Override public String toString() {
		return symbol;
	}
}
```

열거 타입인 BasicOperation은 확장할 수 없지만 인터페이스인 Operation은 확장 가능하고, 연산의 타입으로 사용하면 된다. 이렇게 하면 Operation을 구현한 또 다른 열거 타입을 정의해 기본 타입인 BasicOperation을 대체할 수 있다.

```java
// 확장 가능 열거 타입
public enum ExtendedOperation implements Operation {
	EXP(“^”) {
		public double apply(double x, double y) {
			return Math.pow(x, y);
		}
	},
	REMAINDER(“%”) {
		public double apply(double x, double y) {
			return Math.pow(x, y);
		}
	},

	private final String symbol;

	ExtendedOperation(String symbol) {
		this.symbol = symbol;
	}

	@Override
	public String toString() {
		return symbol;
	}
}
```

```java
// ExtendedOperation의 모든 원소를 테스트하도록 수정한 모습
public static void main(String[] args) {
	double x = Double.parseDouble(args[0]);
	double y = Double.parseDouble(args[1]);
	test(ExtendedOperation.class, x, y);
}

private static <T extends Enum<T> & Operation> void test(
	Class<T> opEnumType, double x, double y) {
		for(Operation op : opEnumType.getEnumConstants())
			System.out.printf(“%f %s %f = %f%n”,x, op, y, op.apply(x, y));
}
```

min 메서드는 test 메서드에 ExtendedOperation의 class 리터럴을 넘겨 확장된 연산들이 무엇인지 알려준다. 여기서 class 리터럴은 한정적 타입 토큰(아이템33) 역할을 한다. opEnumType 매개변수의 선언(<T extends Enum<T> & Operation>)이 복잡한데, Class 객체가 열거 타입인 동시에 Operation의 하위 타입이어야 한다는 뜻이다. 열거 타입이어야 원소를 순회할 수 있고, Operation이어야 원소가 뜻하는 연산을 수행 할 수 있기 때문이다. 

```java
// Class 객체 대신 한정적 와일드 카드 타입 넘기는 방법
public static void main(String[] args) {
	double x = Double.parseDouble(args[0]);
	double y = Double.parseDouble(args[1]);
	test(Arrays.asList(ExtendedOperation.values()), x, y);
}

private static void test(Collection<? extends Operation> opSet, double x, double y) {
	for(Operation op : opSet) 
		System.out.printf(“%f %s %f = %f%n”,x, op, y, op.apply(x, y));
}
```


인터페이스를 이용해 확장 가능한 열거 타입을 흉내 내는 방식에도 한가지 문제가 있는데, 열거 타입끼리 구현을 상속할 수 없다는 점이다.디폴트 구현을 이용해 인터페이스에 추가하는 방법이 있지만 구현하는 모두에 들어가야만 한다.


> ::핵심 정리:: 
> 열거 타입 자체는 확장할 수 없지만, 인터페이스와 그 인터페이스를 구현하는 기본 열거 타입을 함께 사용해 같은 효과를 낼 수 있다. 이렇게 하면 클라이언트는 이 인터페이스를 구현해 자신만의 열거 타입(혹은 다른 타입)을 만들 수 있다. 그리고 API가 (기본 열거 타입을 직접 명시하지 않고) 인터페이스 기반으로 작성되었다면 기본 열거 타입의 인스턴스가 쓰이는 모든 곳을 새로 확장한 열거 타입의 인스턴스로 대체해 사용할 수 있다.




