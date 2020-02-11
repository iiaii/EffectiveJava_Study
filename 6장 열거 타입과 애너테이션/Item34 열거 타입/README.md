# #EffectiveJava/6장_열거타입과애너테이션/34열거타입

자바에는 특수한 목적의 참조 타입이 2가지 있다. 하나의 클래스의 일종인 열거타입(enum), 다른 하나는 인터페이스의 일종인 애너테이션이다. 


## 34. int 상수 대신 열거 타입을 사용하라

열거타입은 일정 개수의 상수 값을 정의한 다음, 그 외의 값은 허용하지 않는 타입이다. 사계절, 태양계의 행성, 카드게임의 카드 종류 등..
이전에는 정수 상수를 한 묶음 선언해서 사용하곤 했다.

```java
// 정수 열거 패턴 - 상당히 취약!
public static final int APPLE_FUJI = 0;
public static final int APPLE_PIPPIN = 1;
public static final int APPLE_GRANNY_SMITH = 2;

public static final int ORANGE_NAVEL = 0;
public static final int ORANGE_TEMPLE = 1;
public static final int ORANGE_BLOOD = 2;

// 향긋한 오렌지 향의 사과 소스!
int i = (APPLE_FUJI-ORANGE_TEMPLE)/APPLE_PIPPIN;
```

정수 열거 패턴 기법에는 단점이 많은데, 타입 안전을 보장할 방법이 없고, 표현력도 좋지 않다. 오렌지를 건네야 할 메서드에 사과를 보내고 동등 연산자(==)로 비교하더라도 컴파일러는 아무런 경고 메시지를 출력하지 않는다.

##### 정수 열거 패턴의 단점
1. APPLE_  ORANGE_ 와 같은 접두어를 사용해 이름 충돌을 방지해야만 한다.
2. 평범한 상수를 나열한 것 뿐이라 컴파일하면 그 값이 클라이언트 파일에 그대로 새겨진다. 상수의 값이 바뀌면 클라이언트도 반드시 다시 컴파일 해야만 잘 작동한다.
3. 정수 상수는 문자열로 출력하기가 다소 까다롭다. (그 값을 출력하거나 디버거로 살펴보면 숫자로만 보여서 도움이 되지 않는다)
4. 정수 열거 그룹에 속한 모든 상수를 순회하는 방법이나 상수 개수를 알 수 없다

-> 대안으로 상수의 의미를 알수 있는 문자열 열거 패턴도 있지만, 문자열 상수의 이름 대신 문자열 값을 그대로 하드코딩하게 만든다. (오타가 있어도 컴파일러는 확인 할 수 없고 런타임 버그 발생, 문자열 비교로 인한 성능 저하)

그래서 나온 대안이 **열거 타입**

```java
// 가장 단순한 열거 타입
public enum Apple { FUJI, PIPIN, GRANNY_SMITH }
public enum Orange { NAVEL, TEMPLE, BLOOD }
```

열거 타입 자체는 클래스이며, 상수 하나당 자신의 인스턴스를 하나씩 만들어 public static final 필드로 공개한다. 열거 타입은 밖에서 접근할 수 있는 생성자를 제공하지 않으므로 사실상 final이다. 따라서 클라이언트가 인스턴스를 직접 생성하거나 확장할 수 없으니 열거 타입 선언으로 만들어진 인스턴스들은 딱 하나씩만 존재함이 보장된다. (열거 타입은 인스턴스 통제된다. 싱글턴은 원소가 하나뿐인 열거타입이라 할 수 있고, 열거타입은 싱글턴을 일반화한 형태라고 볼 수 있다)

열거 타입은 컴파일 타임 타입 안전성을 제공한다. Apple 열거 타입을 매개변수로 받는 메서드를 선언했다면, 건네받은 참조는 Apple의 3가지 값 중 하나임이 확실하다. (다른 타입의 값을 넘기려 하면 컴파일 오류가 난다. 타입이 다른 열거 타입 변수에 할당하려 하거나 다른 타입의 값 끼리 연산자로 비교하려는 것이기 때문)


열거 타입에는 임의의 메서드나 필드를 추가할 수 있고 임의의 인터페이스를 구현하게 할 수도 있다. Object 메서드들을 높은 품질로 구현해놨고, Comparable(아이템14)과 Serializable(12장)을 구현했으며, 그 직렬화 형태도 웬만큼 변형을 가해도 문제없이 동작하게끔 구현해놨다.

```java
// 데이터와 메서드를 갖는 열거타입
public enum Planet {
	MERCURY(3.302e+23,2.439e6),
	VENUS(4.869e+24,6.052e6),
	…

	private final double mass; 		// 질량
	private final double radius;	// 반지름
	private final double surfaceGravity; // 표면증력

	// 중력상수
	private static final double G = 6.67300E-11;

	// 생성자
	Planet(double mass, double radius) {
		this.mass = mass;
		this.radius = radius;
		surfaceGravity = G * mass / (radius * radius);
	}

	public double mass()	{ return mass; }
	public double radius() { return radius; }
	public double surfaceGravity() { return surfaceGravity; }

	public double surfaceWeight(double mass) {
		return mass * surfaceGravity;
	}
}
```

열거타입 상수 각각을 특정 데이터와 연결지으려면 생성자에서 데이터를 받아 인스턴스 필드에 저장하면 된다. (열거 타입은 근본적으로 불변이라 모든 필드는 final이어야 한다. public으로 선언해도 되지만, private으로 두고 별도의 public 접근자메서드를 두는 것이 낫다)

```java
// 8개 행성의 무게를 출력
public class WeightTable {
	public static void main(String[] args) {
		double earthWeight = Double.parseDouble(args[0]);
		double mass = earthWeight / Planet.EARTH.surfaceGravity();
		for(Planet p : Planet.values())
			System.out.printf(“%s에서의 무게는 %f이다.%n”,p,p.surfaceWeight(mass));
```

열거 타입은 자신 안에 정의된 상수들의 값을 배열에 담아 반환하는 정적 메서드 values를 제공한다. (값들은 선언된 순서로 저장됨) 
각 열거 타입 값의 toString 메서드는 상수 이름을 문자열로 반환하므로 println과 printf로 출력하기에 좋다. (재정의 가능)

열거타입을 선언한 클래스 혹은 그 패키지에서만 유용한 기능은 private이나 package-private 메서드로 구현한다. (클라이언트에 노출해야 할 합당한 이유가 없다면 private, package-private으로 선언)
널리 쓰이는 열거타입은 톱레벨 클래스로 만들고, 특정 톱레벨 클래스에서만 쓰인다면 해당 클래스의 멤버 클래스(아이템24)로 만든다. 
예를 들어 소수 자릿수의 반올림 모드를 뜻하는 열거타입인 java.math.RoundingMode는 BigDecimal을 사용한다. 반올림 모드는 관련없는 영역에서도 유용하기 때문에 RoundingMode를 톱레벨로 올렸다.


```java
// 값에 따라 분기하는 열거 타입 - 이대로 만족?
public enum Operation {
	PLUS, MINUS, TIMES,DIVIDE;

	// 상수가 뜻하는 연산을 수행한다.
	public double apply(double x, double y) {
		switch(this) {
			case PLUS: return x+y;
			case MINUS: return x-y;
			case TIMES: return x*y;
			case DIVIDE: return x/y;
		}
		throw new AssertionError(“알 수 없는 연산: “+this);
	}
}
```

동작은 하지만 마지막의 throw문은 실제로 도달할 일이 없지만 기술적으로 도달할 수 있기 때문에 생략하면 컴파일 조차 되지 않는다. 또한 깨지기 쉬운 코드이다. 새로운 상수가 추가되면 case 문도 추가해야 한다. (깜빡하면 “알 수 없는 연산”이 되어버림)

```java
// 상수별 메서드 구현을 활용한 열거 타입
public enum Operation {
	PLUS{public double apply(double x, double y){return x+y;}},
	MINUS{public double apply(double x, double y){return x-y;}},
	TIMES{public double apply(double x, double y){return x*y;}},
	DIVIDE{public double apply(double x, double y){return x/y;}};
	
	public abstract double apply(double x, double y);
}
```
-> apply 메서드가 상수 선언 바로 옆에 붙어 재정의하는 것을 상기시키고 추상 메서드로서 재정의하지 않았다면 컴파일 오류로 알려준다.

```java
// 상수별 클래스 몸체와 데이터를 사용한 열거 타입
public enum Operation {
	PLUS(“+”) {
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
	}

	private final String symbol;

	Operation(String symbol) { this.symbol = symbol; }
	
	@Override public String toString() { return symbol; }
	public abstract double apply(double x, double y);
}

// 사용
public static void main(String[] args) {	
	double x = Double.parseDouble(args[0]);
	double y = Double.parseDouble(args[1]);

	for(Operation op : Operation.values())
		System.out.printf(“%f %s %f = %f%n”,x,op,y,op.apply(x,y));
}
```

열거 타입에는 상수 이름을 입력받아 그 이름에 해당하는 상수를 반환해주는 valueOf(String) 메서드가 자동 생성된다. 
toString 메서드를 재정의하려거든, toString이 반환하는 문자열을 해당 열거 타입 상수로 변환해주는 fromString 메서드도 함께 제공하는 걸 고려해보자. 

```java
// 열거 타입용 fromString 메서드 구현

private static final Map<String, Operation> stringToEnum = Stream.of(values()).collect(toMap(Object::toString, e -> e));

// 지정한 문자열에 해당하는 Operation을 (존재한다면) 반환
public static Optional<Operation> fromString(String symbol) {
	return Optional.ofNullable(stringToEnum.get(symbol));
}
```

Operation 상수가 stringToEnum 맵에 추가되는 시점은 열거 타입 상수 생성 후 정적 필드가 초기화될 때다. (앞의 코드는 values 메서드가 반환하는 배열 대신 스트림을 사용했다)

fromString이 Optional<Operation>을 반환하는데, 이는 주어진 문자열이 가리키는 연산이 존재하지 않을 수 있음을 클라이언트에 알리고, 그 상황을 클라이언트에서 대처함


한편, 상수별 메서드 구현에는 열거 타입 상수끼리 코드를 공유하기 어렵다는 단점이 있다. 
```java
// 값에 따라 분기하여 코드를 공유하는 열거 타입 - 좋은 방법?
enum PayrollDay {	
	MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY, SATURDAY, SUNDAY;

	private static final int MINS_PER_SHIFT = 8*60;

	int pay(int minutesWorked, int payRate) {
		int basePay = minutesWorked * payRate;

		int overtimePay;
		switch(this) {
			case SATURDAY: case SUNDAY: // 주말
				overtimePay = basePay/2;
				break;
			default: // 주중
				overtimePay = minutesWorked <= MINS_PER_SHIFT ? 0 : (minutesWorked - MINS_PER_SHIFT) * payRate / 2;
		}
		return basePay + overtimePay;
	}
}
```

휴가와 같은 새로운 값을 추가하면 case문을 잊지 말고 추가해야함

```java
// 전략 열거 타입 패턴
enum PayrollDay {
	MONDAY(WEEKDAY), TUESDAY(WEEKDAY), WEDNESDAY(WEEKDAY), THURSDAY(WEEKDAY), FRIDAY(WEEKDAY), SATURDAY(WEEKEND), SUNDAY(WEEKEND);

	private final PayType payType;

	PayrollDay(PayType payType) { this.payType = payType; }
	
	int pay(int minutesWorked, int payRate) {
		return payType.pay(minutesWorked, payRate);
	}

	// 전략 열거 타입
	enum PayType {
		WEEKDAY {
			int overtimePay(int minsWorked, int payRate) {
				return minsWorked <= MINS_PER_SHIFT ? 0 : (minsWorked-MINS_PER_SHIFT)*payRate/2;
			}
		},
		WEEKEND {
			int overtimePay(int minsWorked, int payRate) {
				return minsWorked * payRate / 2;
			}
		};

		abstract int overtimePay(int mins, int payRate);
		private static final int MINS_PER_SHIFT = 8 * 60;

		int pay(int minsWorked, int payRate) {
			int basePay = minsWorked * payRate;
			return basePay + overtimePay(minsWorked, payRate);
		}
	}
}
```
-> switch 문이나 상수별 메서드 구현이 필요 없게 됨

switch 문은 열거 타입의 상수별 동작을 구현하는 데 적합하지 않지만 기존 열거 타입에 상수별 동작을 혼합해 넣을 때는 switch 문이 좋은 선택이 될 수 있다.

```java
// switch문을 이용해 원래 열거 타입에 없는 기능을 수행
// 반대 기능 수행
public static Operation inverse(Operation op) {
	switch(op) {
		case PLUS: return Operation.MINUS;
		case MINUS: return Operation.PLUS;
		case TIMES: return Operation.DIVIDE;
		case DIVIDE: return Operation.TIMES;

		default: throw new AssertionError(“알 수 없는 연산: ”+op);
	}
}
```


정리하자면 필요한 원소를 컴파일 타임에 다 알 수 있는 상수 집합이라면 항상 열거타입 사용한다. (태양계 행성, 요일, 체스 말 ..)
열거 타입에 정의된 상수 개수가 영원히 고정 불변일 필요는 없다. (나중에 추가돼도 바이너리 수준에서 호환)


> ::핵심 정리:: 
> 열거 타입은 확실히 정수 상수보다 뛰어나다. (읽기 쉽고 안전하고 강력) 대다수 열거 타입이 명시적 생성자나 메서드 없이 쓰이지만, 각 상수를 특정 데이터와 연결 짓거나 상수마다 다르게 동작하게 할 때는 필요하다. 드물게는 하나의 메서드가 상수별로 다르게 동작해야 할 때도 있다. 이런 열거 타입에서는 switch 문 대신 상수별 메서드 구현을 사용하자. 열거 타입 상수 일부가 같은 동작을 공유한다면 전략 열거 타입 패턴을 사용하자.



