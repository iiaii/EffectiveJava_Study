# #EffectiveJava/5장_제네릭/27비검사경고

## 27. 비검사 경고를 제거하라

제네릭을 사용하기 시작하면 수많은 컴파일러 경고를 보게 된다. 비검사 형변환 경고, 비검사 메서드 호출 경고, 비검사 매개변수화 가변인수 타입 경고, 비검사 변환 경고 등이다. 대부분의 비검사 경고는 쉽게 제거할 수 있다.

`Set<Lark> exaltation = new HashSet();`

```
// javac 명령줄 인수에 -Xlint:uncheck 옵션 추가
Venery.java:4: warning: [unchecked] unchecked conversion
		Set<Lark> exaltation = new HashSet();

	required: Set<Lark>
	found: HashSet
```

`Set<Lark> exaltation = new HashSet<>();`

자바7부터 지원하는 다이아몬드 연산자만으로 해결할 수 있다. 그러면 컴파일러가 올바른 실제 타입 매개변수를 추론해준다.

할수 있는한 모든 비검사 경고를 제거하면 그 코드는 타입 안전성이 보장된다. (ClassCastException 발생 X)
경고를 제거할 수는 없지만 타입 안전하다고 확신할 수 있다면 @SuppressWarnings(“unchecked”)를 달아 경고를 숨기자. (타입 안전함을 검증하지 않은채 경고를 숨기면 잘못된 보안 인식을 심어주는 꼴임. 그 코드는 경고없이 컴파일 되지만 런타임에서 여전히 ClassCastException을 던질 수 있다)
안전하다고 검증된 비검사 경고를 숨기지 않고 그대로 두면, 진짜 문제를 알리는 새로운 경고가 나와도 눈치채지 못할 수 있다. 제거하지 않은 수많은 거짓 경고 속에 새로운 경고가 파묻힐 것이기 때문이다.

@SupperssWarnings 애너테이션은 개별 지역변수 선언부터 클래스 전체까지 어떤 선언에도 달 수 있다. 하지만 가능한 한 좁은 범위에 적용하자. (클래스,메서드 전체는 금기사항, 선언문에만 가능함 return X)

```java
public <T> T[] toArray(T[] a) {
	if(a.length < size) {
		// 생성한 배열과 매개변수로 받은 배열의 타입이 모두 T[]로 같음
		// 올바른 형변환
		@SuppressWarnings(“unchecked”)
		T[] result = (T[]) Arrays.copyOf(elements, size, a.getClass());
		return result;
	}
	System.arraycopy(elements, 0, a, 0, size);
	if(a.length > size)
		a[size] = null;
	return a;
}
```

@SuppressWarnings(“unchecked”) 애너테이션을 사용할 때면 그 경고를 무시해도 안전한 이유를 항상 주석으로 남겨야 한다. 다른 사람이 그 코드를 이해하는데 도움이 되고, 다른 사람이 그 코드를 잘못 수정하여 타입 안정성을 잃는 상황을 줄여준다. 


::핵심 정리:: 

> 비검사 경고는 중요하니 무시하지 말자. 모든 비검사 경고는 런타임에 ClassCastException을 일으킬수 있는 잠재적 가능성을 뜻하니 최선을 다해 제거해야 한다. 경고를 없앨 방법이 없다면, 그 코드가 타입 안전함을 증명하고 가능한 범위를 좁혀 @SuppressWarnings(“unchecked”) 애너테이션으로 경고를 숨기고 주석을 남긴다.








