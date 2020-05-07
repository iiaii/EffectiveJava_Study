# #EffectiveJava/6장_열거타입과애너테이션/37EnumMap

## 37. ordinal 인덱싱 대신 EnumMap을 사용하라

이따금 배열이나 리스트에서 원소를 꺼낼 때 ordinal 메서드(아이템35)로 인덱스를 얻는 코드가 있다.

```java
// 식물 나타내는 클래스
class Plant {
	enum LifeCycle { ANNUAL, PERENNIAL, BIENNIAL }

	final String name;
	final LifeCycle lifeCycle;

	Plant(String name, LifeCycle lifeCycle) {
		this.name = name;
		this.lifeCycle = lifeCycle;
	}

	@Override
	public String toString() {
		return name;
	}
}
```

```java
// ordinal()을 배열 인덱스로 사용 - 따라하지 말 것!
Set<Plant>[] plantsByLifeCycle = (Set<Plant>[]) new Set[Plant.LifeCycle.values().length];

for(int i=0 ; i<plantsByLifeCycle.length; i++)
	plantsByLifeCycle[i] = new HashSet<>();

for(Plant p : garden)
	plantsByLifeCycle[p.lifeCycle.ordinal()].add(p);

// 결과 출력
for(int i=0 ; i<plantsByLifeCycle.length ; i++) {
	System.out.printf(“%s: %s%n”,Plant.LifeCycle.values()[i], plantsByLifeCycle[i]);
}
```

동작은 하지만 문제가 많다. 배열은 제네릭과 호환되지 않으니(아이템28) 비검사 형변환을 수행해야 하고 깔끔히 컴파일 되지는 않는다. 배열은 각 인덱스의 의미를 모르니 출력 결과에 직접 레이블을 달아야 한다. 가장 심각한 문제는 정확한 정숫값을 사용한다는 것을 직접 보증해야 한다. 정수는 열거 타입과 달리 타입 안전하지 않기 때문이다. 잘못된 값을 사용하면 잘못된 동작을 수행하거나 ArrayIndexOutOfBoundsException을 던진다.


EnumMap으로 해결
```java
// EnumMap을 사용해 데이터와 열거 타입을 매핑
Map<Plat.LifeCycle, Set<Plant>> plantsByLifeCycle = new EnumMap<>(Plant.LifeCycle.class);

for(Plant.LifeCycle lc : Plant.LifeCycle.values())
	plantsByLifeCycle.put(lc, new HashSet<>());

for(Plant p : garden)
	plantsByLifeCycle.get(p.lifeCycle).add(p);
System.out.println(plantsByLifeCycle);
```

EnumMap의 성능이 ordinal을 쓴 배열에 비견되는 이유는 그 내부에서 배열을 사용하기 때문, 내부 구현 방식을 안으로 숨겨서 Map의 타입 안전성과 배열의 성능을 모두 얻어 낸 것이다. 여기서 EnumMap의 생성자가 받는 키 타입 Class 객체는 한정적 타입 토큰으로 런타임 제네릭 타입 정보를 제공한다(아이템33).

스트림(아이템45)을 사용해 맵을 관리하면 코드를 더 줄일 수 있다.
```java
// 스트림을 사용한 코드 1 - EnumMap을 사용하지 않는다!
System.out.println(Arrays.stream(garden).collect(groupingBy(p -> p.lifeCycle)));
```

고유한 맵 구현체를 사용했기 때문에 EnumMap을 써서 얻은 공간과 성능의 이점이 사라진다는 문제가 있다.

```java
// 스트림을 사용한 코드 2 - EnumMap을 이용해 데이터와 열거 타입을 매핑
System.out.println(Arrays.stream(garden).collect(groupingBy(p.lifeCycle, () -> new EnumMap<>(LifeCycle.class), toSet()))));
```

EnumMap 버전은 식물의 생애주기당 하나씩의 중첩 맵을 만들지만, 스트림 버전에서는 해당 생애주기에 속하는 식물이 있을 때만 만든다.


```java
// 배열들의 배열의 인덱스에 ordinal()을 사용 - 따라 하지 말 것!
public enum Phase {
	SOLID, LIQUID, GAS;

	public enum Transition {
		MELT, FREEZE, BOIL, CONDENSE, SUBLIME, DEPOSIT;

		// 행은 from의 ordinl을 열은 to의 ordinal을 인덱스로 쓴다
		private static final Transition[][] TRANSITIONS = {
			{ null, MELT, SUBLIME },
			{ FREEZE, null, BOIL },
			{ DEPOSIT, CONDENSE, null }
		};
		
		// 한 상태에서 다른 상태로의 전이를 반환한다
		public static Transition from(Phase from, Phase to) {
			return TRANSITIONS[from.ordinal()][to.ordinal()];
		}
	}
}
```

컴파일러는 ordinal과 배열의 인덱스 관계를 알 수 없기 때문에 Phase 나 Phase.Transition 열거 타입을 수정하면서 상전이 표 TRANSITIONS를 함께 수정하지 않거나 실수로 잘못 수정하면 런타입 오류가 날 것이다. ArrayIndexOutOfBoundsException이나 NullPointerException을 던질 수도 있고, (운이 나쁘면) 예외도 던지지 않고 이상하게 동작할 수도 있다. 그리고 상전이 표의 크기는 상태의 가짓수가 늘어나면 제곱해서 커지며 null로 채워지는 칸도 늘어날 것이다.

```java
// 중첩 EnumMap으로 데이터와 열거 타입 쌍을 연결했다.
public enum Phase {
	SOLID, LIQUID, GAS;

	public enum Transition {
		MELT(SOLID, LIQUID), FREEZE(LIQUID, SOLID),
		BOIL(LIQUID, GAS), CONDENSE(GAS, LIQUID),
		SUBLIME(SOLID, GAS), DEPOSIT(GAS, SOLID);

		private final Phase from;
		private final Phase to;
	
		Transition(Phase from, Phase to) {
			this.from = from;
			this.to = to;
		}
		
		// 상전이 맵을 초기화한다
		private static final Map<Phase, Map<Phase, Transition>> m = Stream.of(values()).collect(groupingBy(t -> t.from, () -> new EnumMap<>(Phase.class), toMap(t -> t.to, t -> t, (x,y) -> y, () -> new EnumMap<>(Phase.class))));

		public static Transition from(Phase from, Phase to) {
			return m.get(from).get(to);
		}
	}
}
```

이 맵의 타입인 Map<Phase, Map<Phase, Transition>>은 이전 상태에서 이후 상태에서 전이로의 맵에 대응시키는 맵 이라는 뜻이다. (복잡하지만 코드를 차근히 다시 볼 것)


여기서 새로운 상태인 플라스마를 추가할때, 이 상태와 연결된 전이는 2개다. 첫 번째는 기체에서 플라스마로 변하는 이온화이고, 둘째는 플라스마에서 기체로 변하는 탈이온화다. 배열로 만든 코드 37-5를 수정하려면 새로운 상수를 Phase에 1개, Phase.Transition에 2개를 추가하고, 원소 9개 짜리인 배열들의 배열을 원소 16개짜리로 교체해야 한다. 

EnumMap 버전에서는 상태 목록에 PLASMA를 추가하고 전이 목록에 IONIZE(GAS, PLASMA), DEIONIZE(PLASMA, GAS)를 추가한다.
```java
// EnumMap 버전에 새로운 상태 추가하기
public enum Phase {
	SOLID, LIQUID, GAS, PLASMA;

	public enum Transition {
		MELT(SOLID, LIQUID), FREEZE(LIQUID,SOLID),
		…
		IONIZE(GAS, PLASMA), DEIONIZE(PLASMA, GAS);
		… // 나머지코드 그대로
	}
}
// 코드 간략히 하고자 해당 전이가 없을때 null을 사용함
```


::핵심 정리:: 

> 배열의 인덱스를 얻기 위해 ordinal을 쓰는 것은 일반적으로 좋지 않으니 EnumMap을 사용하라. 다차원 관계는 EnumMap<…, EnumMap<…>>으로 표현하라. 애플리케이션 프로그래머는 Enum.ordinal을 사용하지 말아야 한다 는 일반 원칙의 특수한 사례다.





