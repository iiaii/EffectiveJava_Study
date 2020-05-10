# #EffectiveJava/8장_메서드/53가변인수


## 53. 가변인수는 신중히 사용하라


가변인수 메서드는 명시한 타입의 인수를 0개 이상 받을 수 있다. 가변인수 메서드를 호출하면, 가장 먼저 인수의 개수와 길이가 같은 배열을 만들고 인수들을 이 배열에 저장하여 가변인수 메서드에 건네준다.

```java
// 가변인수 활용 예
static int sum(int… args) {
	int sum = 0;
	for(int arg : args)
		sum += arg;
	return sum;
}
```

인수 1개 이상이어야 할때도 있다. (최소값을 찾는 메서드인 경우)

```java
// 인수가 1개 이상이어야 하는 가변인수 메서드 - 잘못 구현한 예!
static int min(int… args) {
	if (args.length == 0)
		throw new IllegalArgument(“인수가 1개 이상 필요!”);
	int min = args[0];
	for(int i=1 ; i<args.length ; i++)
		if(args[i] < min)
			min = args[i];
	return min;
}
```

이 방식에는 문제가 있는데, 인수를 0개 넣어 호출하면 런타임에 실패한다. (코드도 지저분 하다) 

```java
// 인수가 1개 이상이어야 할때 (가변인수 제대로 활용)
static int min(int firsArg, int… remainArgs) {
	int min = firstArg;
	for(int arg : remainArgs)
		if(arg < min)
			min = arg;
	return min;
}
```

이상의 예에서 보듯 가변인수는 인수 개수가 정해지지 않았을 때 유용하다. printf는 가변인수와 한묶음으로 자바에 도입되었고, 이때 핵심 리플렉션(아이템65) 기능도 재정비 되었다. (printf와 리플렉션은 가변인수를 활용함)

성능에 민감한 상황이라면 가변인수가 문제가 될 수 있다. 가변인수 메서드는 호출 될때 마다 배열을 새로 하나 할당하고 초기화한다. 다행히, 이 비용을 감당할 수는 없지만 가변인수의 유연성이 필요할 때 선택할 수 있는 멋진 패턴이 있다. 
어떤 메서드 호출의 95%가 인수를 3개 이하로 사용한다고 하면, 인수가 0개인 것 부터 4개인 것까지, 총 5개를 다중정의하자. 마지막 다중정의 메서드가 인수 4개 이상인 5%의 호출을 담당한다.

```java
public void foo() {}
public void foo(int a1) {}
public void foo(int a1, int a2) {}
public void foo(int a1, int a2, int a3) {}
public void foo(int a1, int a2, int a3, int… rest) {}
```

EnumSet의 정적 팩토리도 이 기법을 사용해 열거 타입 집합 생성 비용을 최소화한다.


**::핵심 정리::** 

> 인수 개수가 일정하지 않은 메서드를 정의해야 한다면 가변인수가 반드시 필요하다. 메서드를 정의할 때 필수 매개변수는 가변인수 앞에 두고, 가변인수를 사용할 때는 성능 문제까지 고려하자.

