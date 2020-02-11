# #EffectiveJava/6장_열거타입과애너테이션/36EnumSet

## 36. 비트 필드 대신 EnumSet을 사용하라

열거한 값들이 주로 단독이 아닌 집합으로 사용될 경우, 예전에는 각 상수에 서로 다른 2의 거듭제곱 값을 할당한 정수 열거 패턴을 사용해 왔다. 

```java
// 비트필드 열거 상수 - 구닥다리 기법
public class Text {
	public static final int STYLE_BOLD = 1 << 0; // 1
	public static final int STYLE_BOLD = 1 << 0; // 1
	public static final int STYLE_BOLD = 1 << 0; // 1
	public static final int STYLE_BOLD = 1 << 0; // 1

	// 매갭녀수 styles는 0개이상의 STYLE_ 상수를 비트별 OR한 값
	public void applyStyles(int styles) { … }
}
```

`text.applyStyles(STYLE_BOLD | STYLE_ITALIC);`

비트 필드를 사용하면 비트별 연산을 사용해 합집합과 교집합 같은 집합 연산을 효율적으로 수행할 수 있다. 하지만 비트 필드는 정수 열거 상수의 단점을 그대로 지닌다.
##### 단점들
- 비트 필드 값이 그대로 출력되면 단순한 정수 열거 상수를 출력할 때보다 해석하기가 훨씬 어렵다.
- 모든 원소를 순회하기도 까다롭다
- 최대 몇 비트가 필요한지를 API작성 시 미리 예측하여 적절한 타입을 선택해야 한다.


EnumSet 클래스는 열거 타입 상수의 값으로 구성된 집합을 효과적으로 표현해준다. Set 인터페이스를 완벽히 구현하며, 타입 안전하고, 다른 어떤 Set 구현체와 함께 사용 가능하다 (EnumSet 내부는 벡터로 구현됨)

```java
// EnumSet - 비트 필드를 대체하는 현대적 기법
public class Text {
	public enum Style { BOLD, ITALIC, UNDERLINE, STRIKETHROUGH }

	// 어떤 Set을 넘겨도 되나, EnumSet이 가장 좋다
	public void applyStyles(Set<Style> styles) { … }
}
```

`text.applyStyles(EnumSet.of(Style.BOLD,Style.ITALIC));`

applyStyles 메서드가 EnumSet<Style>이 아닌 Set<Style>을 받은 이유를 생각해 보면 EnumSet을 건네리라 짐작되는 상황이라도 이왕이면 인터페이스로 받는게 일반적으로 좋은 습관이다(아이템64). 이렇게 하면 좀 특이한 다른 Set 구현체를 넘기더라도 처리가 가능하다.


> ::핵심 정리:: 
> 열거할 수 있는 타입을 한데 모아 집합 형태로 사용한다고 해도 비트 필드를 사용할 이유는 없다. EnumSet 클래스가 비트 필드 수준의 명료함과 성능을 제공하고 아이템34에서 설명한 열거 타입의 장점까지 선사한다. EnumSet의 유일한 단점이라면 불변 EnumSet을 반들 수 없다는 것이다(자바9까지는). 향후 릴리스에서는 수정되리라 본다(자바11까지 수정 안됨) Collections.unmodifiableSet으로 EnumSet을 감싸 사용할 수 있다.

