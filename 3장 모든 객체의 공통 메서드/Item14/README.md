# #EffectiveJava/3장_모든객체의공통메서드/14Comparable

## 14. Comparable을 구현할지 고려하라

Comparable 인터페이스의 유일한 메서드인 compareTo는 Object의 메서드가 아니다. 하지만 성격은 두가지만 빼면 equals와 같다. compareTo는 단순 동치성 비교에 더해 순서까지 비교할 수 있으며, 제네릭하다. Comparable을 구현했다는 것은 그 클래스의 인스턴스들에는 자연적인 순서가 있음을 뜻한다. 그래서 Comparable을 구현한 객체들의 배열은 다음처럼 손쉽게 정렬할 수 있다.

`Arrays.sort(a);`

검색, 극단값 계산, 자동 정렬되는 컬렉션 관리도 쉽게 할 수 있다

```java
public class WordList {
	public static void main(String[] args) {
		Set<String> s = new TreeSet<>();
		Collections.addAll(s, args);
		System.out.println(s);
	}
}
```

사실 자바 플랫폼 라이브러리의 모든 값 클래스와 열거타입이 Compareable을 구현했다. 순서가 명확하다면 반드시 Comparable 인터페이스를 구현하자.

compareTo 메서드의 일반 규약은 equals 규약과 비슷하다
> 이 객체와 주어진 객체의 순서를 비교한다. 이 객체가 주어진 객체보다 작으면 음의 정수를, 같으면 0을, 크면 양의 정수를 반환한다. 이 객체와 비교할 수 없는 타입의 객체가 주어지면 ClassCastException을 던진다.
>   다음 설명에서 sgn(표현식) 표기는 수학에서 말하는 부호함수를 뜻하며 표현식의 값이 음수, 0, 양수일 때 -1, 0, 1을 반환하도록 정의했다. 
> — Comparable을 구현한 클래스는 모든 x,y에 대해 sgn(x.compareTo(y)) == -sgn(y.compareTo(x))여야 한다(따라서 x.compareTo(y)는 y.compareTo(x)가 예외를 던질때에 한해 예외를 던져야 한다)
> — Comparable을 구현한 클래스는 추이성을 보장해야 한다. (x.compareTo(y) > 0 && y.compareTo(z) > 0) 이면 x.compareTo(z) > 0이다
> — Comparable을 구현한 클래스는 모든 z에 대해 x.compareTo(y) == 0이면 sgn(x.compareTo(z)) == sgn(y.compareTo(z))이다
> — 필수는 아니지만 (x.compareTo(y) == 0) == (x.equals(y)) 여야 한다. 이 권고를 지키지 않는 모든 클래스는 “주의: 이클래스의 순서는 equals메서드와 일관되지 않다”고 명시해야함

equals와 마찬가지로 기존 클래스를 확장한 구체 클래스에서 새로운 값 컴포넌트를 추가 했다면 compareTo 규약을 지킬 방법이 없다. 객체지향적 추상화의 이점을 포기할 생각이 아니라면 말이다 (리스코프치환원칙 - 상속받은 자식은 부모의 기능을 활용할수 있어야 함, 우회법으로 내부적으로 멤버에 해당 인스턴스를 주입받아서 사용했다 Item10 참고)

Comparable을 구현한 클래스를 확장해 값 컴포넌트를 추가하고 싶다면, 확장하는 대신 독리된 클래스를 만들고, 이 클래스에 원래 클래스의 인스턴스는 가리키는 필드(멤버)를 두자. 그런 다음 내부 인스턴스를 반환하는 뷰 메서드를 제공하면 된다.

정렬된 컬렉션들(TreeMap,TreeSet)은 동치성을 비교할때 compareTo를 사용한다. 따라서 equals와 일치하지 않아도 동작은 한다. compareTo와 equals가 일관되지 않는 BigDecimal 클래스는 빈 HashSet 인스턴스를 생성한 다음 new BigDecimal(“1.0”)과 new BigDecimal(“1.00”)을 차례로 추가하면 내부적으로 equals를 사용해서 2개의 원소를 갖지만, TreeSet을 사용하면 1개의 원소를 갖게 된다.

compareTo에 null을 넣어 호출하면 NullPointerException을 던져야 한다. 

Comparable을 구현하지 않은 필드나 표준이 아닌 순서로 비교해야 한다면 비교자를 대신 사용한다.  비교자는 직접 만들거나 제공하는 것 중에 골라 사용하면 된다.

```java
// 객체 참조 필드가 하나 뿐인 비교자
public final class CaseInsensitiveString implements Comparable<CaseInsensitiveString> {
	public int compareTo(CaseInsensitiveString cis) {
		return String.CASE_INSENSITIVE_ORDER.compare(s, cis.s);
	}
	// 나머지 코드 생략
}
```

이 책의 2판에서는 compareTo 메서드에서 정수 기본 타입 필드를 비교할 때는 관계 연산자인 <,>를 실수 기본 타입 필드를 비교할때는 정적 메서드인 Double.compare, Float.compare를 사용하라고 권했지만, 자바7부터 박싱된 기본 타입 클래스들에 새로 추가된 정적 메서드인 compare을 이용하면 된다. **compareTo 메서드에서 관계 연산자 <, > 를 사용하는 이전 방식은 이제 추천하지 않는다.**

```java
// 기본 타입필드가 여럿일때의 비교자
public int compareTo(PhoneNumber pn) {
	int result = Short.compare(areaCode, pn.areaCode);
	if(result == 0) {
		result = Short.compare(prefix, pn.prefix);
		if(result == 0)
			result = Short.compare(lineNum, pn.lineNum);
	}
	return result;
}
```


자바8에서는 Comparator 인터페이스가 일련의 비교자 생성 메서드와 팀을 꾸려서 메서드 연쇄방식으로 비교자를 생성할 수 있게 되었다. (약간의 성능저하는 있음)

```java
// 비교자 생성 메서드를 활용한 비교자 (정적 임포트 사용함)
private static final Comparator<PhoneNumber> COMPARATOR = comparingInt((PhoneNumber pn) -> pn.areaCode)
		.thenComparingInt(pn -> pn.prefix)
		.thenComparingInt(pn -> pn.lineNum);

public int compareTo(PhoneNumber pn) {
	return COMPARATOR.compare(this, pn);
}
```

Comparator는 많은 보조 생성 메서드들이 있다. (comparingInt, thenComparingInt)

```java
// 정적 compare 메서드를 활용한 비교자
static Comparator<Object> hashCodeOrder = new Comparator<>() {
	public int compare(Object o1, Object o2) {
		return Integer.compare(o1.hashCode(), o2.hashCode());
	}
}

// 비교자 생성 메서드를 활용한 비교자
static Comparator<Object> hashCodeOrder = Comparator.comparingInt(o -> o.hashCode());
```


> ::핵심 정리:: 
> 순서를 고려해야 하는 값 클래스를 작성한다면 Comparable 인터페이스를 구현해라. compareTo 메서드에서 필드의 값을 비교할때 <, > 대신 박싱된 기본 타입 클래스가 제공하는 정적 compare 메서드나 Comparator 인터페이스가 제공하는 비교자 생성 메서드를 사용하자

