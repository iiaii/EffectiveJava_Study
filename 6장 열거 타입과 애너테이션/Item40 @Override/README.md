# #EffectiveJava/6장_열거타입과애너테이션/40@Override

## 40. @Override 애너테이션을 일관되게 사용하라

보통의 프로그래머에게 가장 중요한 @Override는 메서드 선언에만 달 수 있으며, 이 애너테이션이 달렸다는 것은 상위 타입의 메서드를 재정의했음을 뜻한다. 이 애너테이션을 일관되게 사용하면 여러가지 버그를 예방한다

```java
// 영어 알파벳2개로 구성된 문자열을 표현하는 클래스 - 버그를 찾아보자
public class Bigram {
	private final char first;
	private final char second;
	
	public Bigram(char first, char second) {
		this.first = first;
		this.second = second;
	}
	public boolean equals(Bigram b) {
		return b.first == first && b.second == second;
	}
	public int hashCode() {
		return 31 * first + second;
	}

	public static void main(String[] args) {
		Set<Bigram> s = new HashSet<>();
		for(int i=0 ; i<10 ; i++)
			for(char ch=‘a’ ; ch<=‘z’ ; ch++)
				s.add(new Bigram(ch, ch));
		System.out.println(s.size());
	}
}

// 출력
// 260
```

26개를 10번 반복하지만 Set이기 때문에 26이 나와야 한다. 하지만 결과는 260이 나온다. equals와 hashCode를 재정의한 것처럼 보이지만 다중정의(오버로딩, 아이템52)해 버렸다. equals를 재정의 하려면 타입을 Object로 해야만 하는데, 그렇게 하지 않아서 새로 정의한게 되어버렸다. (이 오류는 컴파일러가 잡아내기는 한다)

@Override 애너테이션을 추가해 재정의한다는 의도를 명시해야 한다. 
@Override 애너테이션을 equals에 추가하면 다음의 컴파일 오류가 발생한다
```
Bigram.java:10: method does not override or implement a method from a super type
		@Override public boolean equals(Bigram b) {
```

상위 타입으로 부터 재정의 되지 않았다고 해서 이를 수정하면
```java
@Override public boolean equals(Object o) {
	if(!(o instanceof Bigram))
		return false;
	Bigram b = (Bigram) o;
	return b.first == first && b.second == second;
}
```

그러니 상위 클래스의 메서드를 재정의하려는 모든 메서드에 @Override 애너테이션을 달면 어디가 문제인지도 알수 있다. 

구체 클래스에서 상위 클래스의 추상메서드를 재정의할 때는 굳이 @Override를 달지 않아도 된다. 구체클래스인데 아직 구현하지 않은 추상메서드가 있으면 컴파일러가 그 사실을 알려준다. (대부분의 IDE는 자동으로 붙여준다)

한편, IDE는 @Override를 일관되게 사용하도록 부추기기도 한다. (@Override가 달려있지 않은 메서드를 재정의했다고 알림)


@Override는 클래스뿐 아니라 인터페이스의 메서드를 재정의할 때도 사용할 수 있는데, 디폴트 메서드를 지원하기 시작하면서, 인터페이스 메서드를 구현한 메서드에도 @Override를 다는 습관을 들이면 시그니처가 올바른지 재차 확신할 수 있다. 구현하려는 인터페이스에 디폴트 메서드가 없음을 안다면 이를 구현한 메서드에서는 @Override를 생략해 코드를 조금 더 깔끔히 유지해도 좋다. 

하지만 추상 클래스나 인터페이스에서는 상위 클래스나 상위 인터페이스의 메서드를 재정의하는 모든 메서드에 @Override를 다는 것이 좋다. (상위 클래스가 구체클래스든 추상 클래스든 마찬가지)


> ::핵심 정리:: 
> 재정의한 모든 메서드에 @Override 애너테이션을 의식적으로 달면 실수 했을때 컴파일러가 바로 알려준다. 예외는 구체클래스에서 상위 클래스의 추상 메서드를 재정의한 경우인데 그래도 달자



