# #EffectiveJava/3장_모든객체의공통메서드/10equals

Object는 객체를 만들 수 있는 구체 클래스지만 기본적으로 상속해서 사용하도록 설계됨
Object에서 final이 아닌 메서드(equals, hashCode, toString, clone, finalize)는 모두 재정의를 염두하고 설계되어서 재정의 시 지켜야 하는 일반 규약이 명확히 정의되었다.

Object를 상속하는 클래스, 즉 모든 클래스는 이 메서드들을 일반 규약에 맞게 재정의 해야 한다. 메서드를 잘못 구현하면 대상 클래스가 이 규약을 준수한다고 가정하는 클래스(HashMap과 HashSet 등)를 오동작하게 만들 수 있다. 

이번 장에서는 final이 아닌 Object 메서드들을 언제 어떻게 재정의해야 하는지를 다룬다. (finalize는 제외, Comparable.compareTo의 경우 Object 메서드는 아니지만 성격이 비슷해서 같이 다룸)

---

## 10. equals는 일반 규약을 지켜 재정의하라

equals 메서드는 재정의 하기 쉬워 보이지만 함정이 있다. (문제를 회피하는 가장 쉬운 방법은 아예 재정의 하지 않는것) 그냥 두면 그 클래스의 인스턴스는 오직 자기 자신과만 같게 된다.

— 재정의 하지 않는 것이 오히려 나은 경우

- 각 인스턴스가 본질적으로 고유하다

값을 표현하는게 아니라 동작하는 개체를 표현하는 클래스가 여기 해당한다. Thread가 좋은 예로, Object의 equals메서드는 이러한 클래스에 맞게 구현되었다.

- 인스턴스의 ‘논리적 동치성(logical equality)’을 검사할 일이 없다

java.util.regex.Pattern은 equals를 재정의해서 두 Pattern의 인스턴스가 같은 정규표현식을 나타내는지 검사하는, 논리적 동치성을 검사하는 방법도 있다. 하지만 설계자는 클라이언트가 이 방식을 원치 않거나 애초에 필요하지 않다고 판단할 수도 있다. (후자라면 Object의 equals로 해결 가능)

- 상위 클래스에서 재정의한 equals가 하위 클래스에도 딱 들어맞는다. 

대부분의 Set구현체는 AbstractSet이 구현한 equals를 상속받아 쓰고, List 구현체들은 AbstractList로 부터, Map구현체들은 AbstractMap으로 부터 상속받아 그대로 쓴다

- 클래스가 private이거나 package-private이고 equals메서드를 호출할 일이 없다

실수로라도 호출되는걸 막는다면
```java
@Override
public boolean equals(Object o) {
	throw new AssertionError(); // 호출 금지!
}
```


반대로 equals를 재정의 해야할때는

객체 식별성(object identity; 두 객체가 물리적으로 같은가)이 아니라 논리적 동치성을 확인해야 하는데, 상위 클래스의 equals가 논리적 동치성을 비교하도록 재정의되지 않았을때. -> 논리적 동치성 비교를 해야하는 경우 (Integer, String과 같은 값 클래스들)

값 클래스의 경우 두 값 객체를 equals로 비교하는 것이 아니라 값 비교를 하고 싶은 것. 논리적 동치성을 확인하도록 재정의해두면 값 비교는 물론, Map의 키와 Set의 원소로 사용할 수 있게 된다.


값 클래스라 해도, 값이 같은 인스턴스가 둘 이상 만들어지지 않음을 보장하는 인스턴스 통제 클래스(아이템 1)이라면 equals를 재정의하지 않아도 된다. Enum도 여기 해당한다. 이런 클래스에서는 어차피 논리적으로 같은 인스턴스가 2개 이상 만들어지지 않으니 논리적 동치성과 객체 식별성이 사실상 똑같은 의미가 된다. 따라서 Object의 equals가 논리적 동치성까지 확인해 준다고 볼 수 있다.


equals 메서드를 재정의 할때는 반드시 일반 규약을 따라야 한다

> equals 메서드는 동치 관계를 구현하며, 다음을 만족한다
>  
> — 반사성(reflexivity) : null이 아닌 모든 참조값 x에 대해, x.equals(x)는 true다
> — 대칭성(symmetry) : null이 아닌 모든 참조값 x,y에 대해, x.equals(y)가 true면 y.equals(x)도 true
> — 추이성(transitivity) : null이 아닌 모든 참조값 x,y,z에 대해 x.equals(y)가 true고 y.equals(z)도 true면 x.equals(z)도 true
> — 일관성(consistency) : null이 아닌 모든 참조 값 x,y에 대해, x.equals(y)를 반복해서 호출하면 항상 true를 반환하거나 항상 false를 반환한다
> — null-아님 : null이 아닌 모든 참조값 x에 대해 x.equals(null)은 false다


한 클래스의 인스턴스는 다른 곳으로 빈번히 전달된다. 그리고 컬렉션 클래스들을 포함해 수 많은 클래스는 전달받은 객체가 equals규약을 지킨다고 가정하고 동작한다.
(equals 규약을 반드시 지켜야 함)


Object 명세에서 말하는 동치관계는 집합을 서로 같은 원소들로 이루어진 부분집합으로 나누는 연산을 말한다. 이 부분집합을 동치류라 한다. equals 메서드가 쓸모 있으려면 모든 원소가 같은 동치류에 속한 어떤 원소와도 서로 교환할 수 있어야 한다.


*반사성*은 단순히 말하면 객체는 자기 자신과 같아야 한다는 뜻이다. 이 요건을 어긴 클래스의 인스턴스를 컬렉션에 넣은 다음 contains 메서드를 호출하면 방금 넣은 인스턴스가 없다고 답할 것임 (contains는 내부적으로 indexOf를 사용하고 indexOf는 for문과 equals를 사용)


*대칭성*은 두 객체는 서로에 대한 동치 여부에 똑같이 답해야 한다는 뜻이다
(새로운 String 클래스를 선언해서 equalsIgnoreCase를 사용해도 String과 연동되지 않는다, String은 대소문자를 구분하기 때문에)


*추이성*은 첫 번째(객체)와 두 번째가 같고 두 번째와 세 번째가 같으면 첫 번째와 세 번째도 같아야 한다는 뜻.
(point (x,y)에 color 가 추가된 클래스, 즉 구체클래스를 확장해 새로운 값을 추가하면서 equals의 규약을 만족시키는 방법은 존재하지 않는다)

대신 우회방법으로 상속대신 컴포지션을 사용하면 가능하다 (아이템18)
Point를 상속하는 대신 Point를 ColorPoint의 private 필드로 두고, ColorPoint와 같은 위치의 일반 Point를 반환하는 뷰 메서드를 public으로 추가하는 식이다
```java
public class ColorPoint {
	private final Point point;
	private final Color color;

	public ColorPoint(int x, in y, Color color) {
		point new Point(x,y);
		this.color = Objects.requireNonNull(color);
	}

	// 이 ColorPoin의 Point 뷰를 반환
	public Point asPoint() {
		return point;
	}

	@Override
	pulibc boolean equals(Object o ) {
		if(!(o instanceof ColorPoint))
			return false;
		ColorPoint cp = (ColorPoint) o;
		return cp.point.equals(point) && cp.color.equals(color);
	}
	…
}
// Timestamp와 Date를 섞어 쓸때는 주의해야함 (Timestamp가 확장해서 필드가 추가됨)
```

추상 클래스의 하위 클래스에서라면 equals규약을 지키면서도 값을 추가할 수 있다. “태그 달린 클래스보다는 클래스 계층구조를 활용하라”는 아이템 23의 조언을 따른다. 상위클래스를 직접 인스턴스로 만드는게 불가능하다면 이런 문제는 일어나지 않는다. (인터페이스나 추상클래스)


*일관성*은 두 객체가 같다면 (어느 하나 혹은 두 객체 모두가 수정되지 않는 한) 앞으로도 영원히 같아야 한다는 뜻이다. 가변 객체는 비교 시점에 따라 결과가 다를 수 있지만, 불변객체의 경우는 한번 다르면 끝까지 달라야 한다.

클래스가 불변이든 가변이든 equals의 판단에 신뢰할 수 없는 자원이 끼어들게 해서는 안된다. (이 제약을 어기면 일관성 조건을 만족시키기 어렵다)

java.ne.URL의 equals는 주어진 URL과 매핑된 호스트의 IP주소를 이용해 비교한다. 호스트 이름을 IP주소로 바꾸려면 네트워크를 통해야 하는데, 그 결과가 항상 같다고 보장할 수 없다. (이는 URL의 equals가 일반 규약을 어기게 하고, 실무에서도 종종 문제를 일으킨다. 즉, 잘못 구현되었다) 이런 문제를 피하려면 equals는 항시 메모리에 존재하는 객체만을 사용한 결정적 계산만 수행해야 한다.


*null-아님*은 모든 객체가 null과 같지 않아야 한다는 뜻이다. null검사를 굳이 할 필요 없이 instanceof를 사용하면 첫 번째 피연산자가 null이면 false를 반환한다. 
```java
// 묵시적 null 검사
@Override
public booelean equals(Object o) {
	if(!(o instanceof MyType))
		return false;
	MyType mt = (MyType) o;
	…
}
```


equals메서드 구현 방법을 정리하자면

1. ==연산자를 사용해 입력이 자기 자신의 참조인지 확인한다

자기자신이면 true를 반환하며 비교작업이 복잡한 상황일 때 사용한다

2. instanceof 연산자로 입력이 올바른 타입인지 확인한다

올바르지 않으면 false를 반환한다. 종종 다른 클래스 끼리도 비교할 수 있도록 (인터페이스를 구현한 클래스들) equals 규약을 수정하기도 한다. (Set, List, Map, Map.Entry 등)

3. 입력을 올바른 타입으로 형변환한다

instanceof 검사를 통해 가능해진다

4. 입력 객체와 자기 자신의 대응되는 ‘핵심’ 필드들이 모두 일치하는지 하나씩 검사한다

모든 필드가 일치하면 true를, 하나라도 다르면 false를 반환한다.


float와 double을 제외한 기본 타입 필드는 ==연산자로 비교하고, 참조 타입 필드는 각각의 equals메서드로, float와 double 필드는 각각 정적 메서드인 Float.compare(float, float)와 Double.compare(double, double)로 비교한다. float와 double을 특별 취급하는 이유는 Float.NaN, -0.0f, 특수한 부동소수 값 등을 다뤄야 하기 때문이다. (Float.equals 는 오토박싱을 동반하므로 성능상 좋지 않음)

Objects.equals(Object, Object)로 비교하면 NullPointerException을 예방할 수 있다. 

어떤 필드 먼저 비교하느냐에 따라 성능을 좌우하기도 한다. 동기화용 락 필드와 같이 객체의 논리적 상태와 관련 없는 필드는 비교하면 안된다.

equals를 다 구현한뒤 3가지만 자문해 보자.

- 대칭적인가?
- 추이성이 있는가?
- 일관적인가?

그리고 단위 테스트를 작성해서 돌려보자. (equals 메서드를 AutoValue를 이용해 작성했다면 테스트를 생략해도 안심할 수 있다)

```java
import sun.net.www.content.text.plain;

public final class PhoneNumber {
    private final short areaCode, prefix, lineNum;

    public PhoneNumber(int areaCode, int prefix, int lineNum) {
        this.areaCode = rangeCheck(areaCode, 999, “지역코드”);
        this.prefix = rangeCheck(prefix, 999, “프리픽스”);
        this.lineNum = rangeCheck(lineNum, 9999, “가입자 번호”);
    }

    private static short rangeCheck(int val, int max, String arg) {
        if(val<0 || val>max)
            throw new IllegalArgumentException(arg+”: “+val);
        return (short)val;
    }

    @Override
    public boolean equals(Object o) {
        if(o == this)
            return true;
        if(!(o instanceof PhoneNumber))
            return false;
        PhoneNumber pn = (PhoneNumber)o;
        return pn.lineNum==lineNum && pn.prefix==prefix && pn.areaCode ==areaCode;
    }
    // 나머지 코드 생략
}
```


주의사항

- equals를 재정의할 땐 hashCode도 반드시 재정의하자(아이템11)
- 너무 복잡하게 해결하려 하지 말자. (필드들의 동치성만 검사해도 지킬 수 있다)
- Object 외의 타입을 매개변수로 받는 equals 메서드는 선언하지 말자 (오버라이딩이 아니라 새로 정의한것, 즉 오버로딩된 것임)

AutoValue 프레임워크를 사용하면 클래스에 애너테이션 하나만 추가하면 이 메서드들을 알아서 작성해 준다 (대다수의 IDE도 해준다)


::핵심 정리:: 

> 꼭 필요한 경우가 아니면 equals를 재정의하지 말자. 많은 경우에 Object의 equals가 원하는 비교를 정확히 수행해준다. 재정의해야 할 때는 그 클래스의 핵심 필드 모두를 빠짐없이, 다섯가지 규약을 확실히 지켜가며 비교해야 한다







