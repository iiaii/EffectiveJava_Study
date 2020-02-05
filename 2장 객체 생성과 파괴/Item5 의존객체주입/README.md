# #EffectiveJava/2장_객체생성과파괴/5의존객체주입

## 5. 자원을 직접 명시하지 말고 의존 객체 주입을 사용하라

많은 클래스가 하나 이상의 자원에 의존한다. 가령 맞춤법 검사기는 사전에 의존하는데, 이런 클래스를 정적 유틸리티 클래스로 구현한 모습을 두물지 않게 볼 수 있다.

- 정적 유틸리티를 잘못 사용한 예 - 유연하지 않고 테스트하기 어렵다
```java
public class SpellChecker {
	private static final Lexicon dictionary = …;

	private SpellChecker() {} // 객체 생성방지

	public static boolean isValid(String word) { … }
	public static List<String> suggenstions(String typo) { … }
}
```

- 싱글턴을 잘못 사용한 예 - 유연하지 않고 테스트하기 어렵다
```java
public class SpellChecker {
	private final Lexicon dictionary = …;

	private SpellChecker(…) {}
	public static SpellChecker INSTANCE = new SpellChecker(…);

	public boolean isValid(String word) { … }
	public List<String> suggestions(String typo) { … }
}
```

두 방식 모두 사전을 단 하나만 사용한다고 가정한다는 점에서 그리 훌륭해 보이지 않는다. 실전에서는 사전이 언어별로 따로 있고 특수 어휘용 사전을 별도로 두기도 한다. 심지어 테스트용 사전도 필요할 수 있다. 사전 하나로 이 모든 쓰임에 대응할 수 있기를 바라는건 너무 순진한 생각이다

SpellChecker가 여러 사전을 사용할 수 있도록 만들어보자. 간단히 dictionary 필드에서 final 한정자를 제거하고 다른 사전으로 교체하는 메서드를 추가할 수도 있지만, 아쉽게도 이방식은 어색하고 오류를 내기 쉬우며 멀티스레드 환경에서는 쓸수 없다. **사용하는 자원에 따라 동작이 달라지는 클래스에는 정적 유틸리티 클래스나 싱글턴 방식이 적합하지 않다**

대신 클래스가 여러 자원 인스턴스를 지원해야 하며, 클라이언트가 원하는 자원을 사용해야 한다. 이 조건을 만족하는 간단한 패턴은 **인스턴스를 생성할 때 생성자에 필요한 자원을 넘겨주는 방식**이다. (이는 의존 객체 주입의 한 형태로, 맞춤법 검사기를 생성할 때 의존객체인 사전을 주입해주면 된다)

```java
public class SpellChecker {
	private final Lexicon dictionary;

	public SpellChecker(Lexicon dictionary) {
		this.dictionary = Objects.requireNonNull(dictionary);
	}

	public boolean isValid(String word) { … }
	public List<String> suggestions(String typo) { … }
}
```


의존 객체 주입 패턴은 아주 단순해서 수 많은 프로그래머가 이름이 있다는 사실도 모른 채 사용해왔다. (자원이 몇 개든 의존 관계가 어떻든 상관없이 잘 작동한다. 또한 불변을 보장하여 여러 클라이언트가 의존 객체들을 안심하고 공유할 수 있기도 하다. 의존 객체 주입은 생성자, 정적 팩토리, 빅더 모두에게 똑같이 응용할 수 있다.

이 패턴의 쓸만한 변형으로, 생성자에 자원 팩토리를 넘겨주는 방식이 있다. 팩토리란 호출할 때마다 특정 타입의 인스턴스를 반복해서 만들어주는 객체를 말한다.

자바 8에서 소개한 Supplier<T> 인터페이스가 팩토리를 표현한 완벽한 예다. Supplier<T>를 입력으로 받는 메서드는 일반적으로 한정적 와일드 카드 타입을 사용해 팩토리의 타입 매개변수를 제한해야 한다. 이를 사용해 클라이언트는 자신이 명시한 타입의 하위 타입이라면 무엇이든 생성할 수 있는 팩토리를 넘길 수 있다. 

`Mosaic create(Supplier<? extends Tile> tileFactory { … }`
-> 클라이언트가 제공한 팩토리가 생성한 타일들로 구성된 모자이크를 만드는 메서드

의존 객체 주입이 유연성과 테스트 용이성을 개선해주긴 하지만, 의존성이 수천 개나 되는 큰 프로젝트에서는 코드를 어지럽게 만들기도 한다. Spring 같은 의존 객체 주입 프레임워크를 사용하면 이런 어질러짐을 해소할 수 있다. 

> ::핵심 정리::
> 클래스가 내부적으로 하나 이상의 자원에 의존하고, 그 자원이 클래스 동작에 영향을 준다면 싱글턴과 정적 유틸리티 클래스는 사용하지 않는 것이 좋다. 이 자원들을 클래스가 직접 만들게 해서도 안된다. 대신 필요한 자원을 (혹은 그 자원을 만들어 주는 팩톨를) 생성자에 (혹은 정적 팩토리나 빌더에) 넘겨주자. 의존 객체 주입이라 하는 이 기법은 클래스의 유연성, 재사용성, 테스트 용이성을 개선해 준다


