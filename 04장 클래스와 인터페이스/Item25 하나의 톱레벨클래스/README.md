# #EffectiveJava/4장_클래스와인터페이스/25하나의톱레벨클래스

## 25. 톱레벨 클래스는 한 파일에 하나만 담으라

소스 파일 하나에 톱레벨 클래스를 여러개 선언하더라도 자바 컴파일러는 불평하지 않는다. 하지만 아무런 득이 없을 뿐더러 심각한 위험을 감수해야하는 행위다. 이렇게 하면 한 클래스를 여러가지로 정의할 수 있으며, 그중 어느 것을 사용할지는 어느 소스 파일을 먼저  컴파일하냐에 따라 달라지기 때문이다.

```java
// 메인 클래스 하나를 담고 있고 메인 클래스는 다른 톱레벨 클래스 2개를 참조한다
pubic class Main {
	public static void main(String[] args) {
		System.out.println(Utensil.NAME + Dessert.Name);
	}
}

// 두 클래스가 한 파일에 정의되어 있다 - 따라하지 말것 (Utensil.java)
class Utensil {
	static final String NAME = “pan”;
}
class Dessert {
	static final String NAME = “cake”;
}

// 여기까지는 Main을 실행하면 pancake를 출력한다

// 우연히 똑같은 두 클래스를 담은 Dessert.java를 만들면
class Utensil {
	static final String NAME = “pot”;
}
class Dessert {
	static final String NAME = “pie”;
}
```

운 좋게 java Main.java Dessert.java 명령으로 컴파일한담녀 컴파일 오류가 나고 클래스를 중복정의 했다고 알려준다. 
(컴파일러는 Main.java 를 컴파일하고 그안에서 Utensil 참조를 만나면 Utensil.java 파일을 살펴 두 클래스를 찾아낸다. 그런 다음 컴파일러가 두번째 명령줄 인수로 넘어온 Dessert.java를 처리하려 할때 같은 클래스의 정의가 이미 있음을 알게된다)

한편 java Main.java 나 java Main.java Utensil.java 명령으로 컴파일하면 pancake를 출력한다. 그러나 java Dessert.java Main.java 명령으로 컴파일하면 potpie를 출력한다. (컴파일러에 어느 소스 파일을 먼저 건네느냐에 따라 동작이 달라짐)


이를 해결하기 위해서는 톱레벨 클래스들을 서로 다른 소스 파일로 분리하면 된다. 굳이 톱레벨 클래스를 한 파일에 담고 싶으면 정적 멤버 클래스(아이템24)를 사용하는 방법을 고민해 볼 수 있다. (다른 클래스에 딸린 부차적인 클래스라면 정적 멤버클래스가 나음)
읽기 좋고, private으로 선언하면(아이템15) 접근 범위도 최소로 관리할 수 있기 때문이다. 

```java
public class Test {
	public static void main(String[] args) {
		System.out.println(Utensil.NAME + Dessert.NAME);
	}
	
	private static class Utensil {
		static final String NAME = “pan”;
	}

	private static class Dessert {
		static final String NAME = “cake”;
	}
}
```


::핵심 정리:: 

> 소스 파일 하나에는 반드시 톱레벨 클래스(혹은 톱레벨 인터페이스)를 하나만 담아두자. 이규칙만 따른다면 컴파일러가 한 클래스에 대한 정의를 여러개 만들어 내는 일은 사라진다. 소스파일을 어떤 순서로 컴파일하든 바이너리 파일이나 프로그램의 동작이 달라지는 일은 결코 일어나지 않는다




