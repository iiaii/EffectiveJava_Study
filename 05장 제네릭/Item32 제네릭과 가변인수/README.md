# #EffectiveJava/5장_제네릭/32제네릭과가변인수

## 32. 제네릭과 가변인수를 함께 쓸 때는 신중하라

가변인수 메서드(아이템53)와 제네릭은 자바5때 함께 추가되었지만 잘 어우러지지는 않는다. 
가변인수는 메서드에 넘기는 인수의 개수를 클라이언트가 조절할 수 있게 해주는데 허점이 있다. 가변인수 메서드를 호출하면 가변인수를 담기 위한 배열이 자동으로 하나 만들어진다. 내부로 감춰야 했을 이 배열이 클라이언트에 노출되는 문제가 생겼는데, varargs 매개변수에 제네릭이나 매개변수화 타입이 포함되면 알기 어려운 컴파일경고가 발생한다.

아이템28에서 실체화 불가 타입은 런타임에는 컴파일타임보다 타입 관련 정보를 적게 담고 있음을 배웠다. 그리고 거의 모든 제네릭과 매개변수화 타입은 실체화되지 않는다. 메서드를 선언할 때 실체화 불가 타입으로 varargs 매개변수를 선언하면 컴파일러가 경고를 보낸다. (가변인수 메서드를 호출할 때도 varargs 매개변수가 실체화 불가 타입으로 추론되면, 그 호출에 대해서도 경고를 낸다)

`warning: [unchecked] Possible heap pollution from parameterized varargs type List<String>`

매개변수화 타입의 변수가 타입이 다른 객체를 참조하면 힙 오염이 발생한다. 다른 객체를 참조하는 상황에서는 컴파일러가 자동 생성한 형변환이 실패할 수 있어서 제네릭 타입 시스템의 안전성의 근간이 흔들리게 된다.

```java
// 제네릭과 varargs를 혼용하면 타입 안정성 깨진다!
static void dangerous(List<String>… stringLists) {
	List<Integer> intList = List.of(42);
	Object[] objects = stringLists;
	objects[0] = intList;	// 힙 오염 발생
	String s= stringLists[0].get(0);	// ClassCastException
	// 마지막 줄에 컴파일러가 생성한 형변환으로 ClassCastException을 던진다
}
```
-> 타입 안정성이 깨져 제네릭 varargs 배열 매개변수에 값을 저장하는 것은 안전하지 않다.

제네릭 배열을 프로그래머가 직접 생성하는 것은 허용하지 않으면서 제네릭 varargs 매개변수를 받는 메서드를 선언하게 한 이유는, 매개변수화 타입의 varargs 매개변수를 받는 메서드가 실무에서 유용하기 때문이다.

자바 라이브러리도 이런 메서드를 제공한다
- Arrays.asList(T… a)
- Collections.addAll(Collections<? super T> c, T… elements)
- EnumSet.of(E first, E… rest) 


자바7에서 @SafeVarargs 애너테이션이 추가되어 제네릭 가변인수 메서드 작성자가 클라이언트 측에서 발생하는 경고를 숨길 수 있게 되었다. @SafeVarargs 는 메서드 작성자가 그 메서드가 타입 안전함을 보장하는 장치다. (메서드가 안전한게 확실하지 않으면 @SafeVarargs 애너테이션을 달아서는 안된다)
-> 가변인수 메서드를 호출할때 varargs 매개변수를 담는 제네릭 배열이 만들어지는데, 이 배열에 아무것도 저장하지 않고 그 배열의 참조가 밖으로 노출되지 않는다면 타입 안전하다. (즉, 매개변수 배열이 호출자로 부터 그 메서드로 순수하게 인수들을 전달하는 일만 한다면 그 메서드는 안전하다)


```java
// 자신의 제네릭 매개변수 배열의 참조를 노출 - 안전하지 않다!
static <T> T[] toArray(T… args) {
	return args;
}
```
-> 힙 오염을 이 메서드를 호출한 쪽의 콜스택으로 전이할 수 있음
(힙 오염 - 매개변수화 타입의 변수가 타입이 다른 객체를 참조)

```java
static <T> T[] pickTwo(T a, T b, T c) {
	swithc(ThreadLocalRandom.current().nextInt(3)) {
		case 0: return toArray(a, b);
		case 1: return toArray(a, c);
		case 2: return toArray(b, c);
	}
	throw new AssertrionError(); // 도달할 수 없음
}

// 클라이언트 측
public static void main(String[] args) {
	String[] attributes = pickTwo(“좋은”, “빠른”, “저렴한”);
}
```
-> 컴파일되지만, 실행하면 ClassCastException을 던진다. 

Object[]는 String[]의 하위타입이 아니므로 형변환이 실패하지만, 힙 오염을 발생시킨 진짜 원인인 toArray로 부터 두단계나 떨어져있고, varargs 매개변수 배열은 실제 매개변수가 저장된 후 변경된 적이 없다.

이 예는 제네릭 varargs 매개변수 배열에 다른 메서드가 접근하도록 허용하면 안전하지 않다는 것을 보여준다. 

```java
// 제네릭 varargs 매개변수를 안전하게 사용하는 메서드
@SafeVarargs
static <T> List<T> flatten(List<? extends T>… lists) {
	List<T> result = new ArrayList<>();
	for(List<? extends T> list : lists)
		result.addAll(list);
	return result;
}
```
-> 제네릭이나 매개변수화 타입의 vargs 매개변수를 받는 모든 메서드에 @SafeVarargs를 달면, 사용자를 헷갈리게 하는 컴파일러 경고를 없앨 수 있다. (안전하지 않은 varargs메서드는 작성하면 안됨)

##### 다음을 만족하는 제네릭 varargs메서드는 안전하다
- varargs 매개변수 배열에 아무것도 저장하지 않는다
- 그 배열(혹은 복제본)을 신뢰할 수 없는 코드에 노출하지 않는다

> @SafeVarargs 애너테이션은 재정의할 수 없는 메서드에만 달아야 한다. 재정의한 메서드도 안전할지 보장할 수 없기 때문이다. 자바8에서 이 애너테이션은 오직 정적 메서드와 final 인스턴스 메서드에만 붙일 수 있고, 자바9는 private 인스턴스 메서드에 허용된다

@SafeVarargs가 유일한 정답은 아니다. 아이템28의 조언을 따라 (실체는 배열인) varargs 매개변수를 List 매개변수로 바꿀 수도 있다.

```java
// 제네릭 varargs 매개변수를 List로 대처한 예 - 타입 안전하다
static <T> List<T> flatten(List<List<? extends T>> lists) {
	List<T> result = new ArrayList<>();
	for(List<? extends T> list: lists)
		result.addAll(list);
	return result;
}
```

정적 팩토리 메서드인 List.of를 활용하면 다음 코드와 같이 ㅣ메서드에 임의의 개수의 인수를 넘길 수 있다. (List.of에도 @SafeVarargs 애너테이션이 달려 있음)

`audience = flatten(List.of(friends, romans, countrymen));`

```java
static <T> List<T> pickTwo(T a, T b, T c) {
	switch(ThreadLocalRandom.current().nextInt(3)) {
		case 0: return List.of(a, b);
		case 1: return List.of(a, c);
		case 2: return List.of(b, c);
	}
	throw new AssertrionError();
}

// 클라이언트
public static void main(String[] args) {
	List<String> attributes = pickTwo(“좋은”, “빠른”, “저렴한”);
}
```
-> 배열없이 제네릭만 사용하므로 타입 안전함


::핵심 정리:: 

> 가변인수와 제네릭은 궁합이 좋지 않다. 가변인수 기능은 배열을 노출하여 추상화가 완벽하지 못하고, 배열과 제네릭의 타입 규칙이 서로 다르기 때문이다. 제네릭 varargs 매개변수는 타입 안전하지는 않지만, 허용된다. 메서드에 제네릭 (혹은 매개변수화된) varargs 매개변수를 사용하고자 한다면, 먼저 그 메서드가 타입 안전한지 확인한 다음 @SafeVarargs 애너테이션을 달아 사용하는데 불편함이 없게끔 하자.


