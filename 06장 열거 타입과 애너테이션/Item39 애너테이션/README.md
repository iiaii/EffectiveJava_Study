# #EffectiveJava/6장_열거타입과애너테이션/39애너테이션

## 39. 명명 패턴보다 애너테이션을 사용하라

전통적으로 도구나 프레임워크가 특별히 다뤄야 할 프로그램 요소에는 구분되는 명명 패턴을 적용해 왔다. (JUnit 테스트 메서드 이름을 test로 시작하게끔 했다)
명명 패턴에는 여러 단점이 있다.
- 오타가 나면 무시하고 지나친다. (ex. tsetSafetyOverride)
- 올바른 프로그램 요소에서만 사용되리라 보증할 방법이 없다 (클래스이름은 Test~해도 무시함)
- 프로그램 요소를 매개변수로 전달할 마땅한 방법이 없다 (특정 예외를 던저야만 성공하는 테스트)
-> 애너테이션이 모든 문제를 해결한다

```java
// 마커 애너테이션 타입 선언
import java.lang.annotation.*;

/**
 * 테스트 메서드임을 선언하는 애너테이션이다.
 * 매개변수 없는 정적 메서드 전용이다
 */
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface Test {
}
```

선언에 다는 애너테이션을 메타애너테이션이라 하는데, @Retention(RetentionPolicy.RUNTIME) 메타애너테이션은 @Test가 런타임에도 유지되어야 한다는 표시다. 이 메타애너테이션을 생략하면 테스트 도구는 @Test를 인식할 수 없다. @Target(ElementType.METHOD) 메타애너테이션은 @Test 가 반드시 메서드 선언에서만 사용돼야 한다고 알려준다. 클래스 선언 필드 선언 등 다른 프로그램 요소에는 달 수 없다.

앞 코드의 메서드 주석에는 매개변수 없는 정적 메서드 전용이다라고 쓰여 있는데, 이 제약을 컴파일러가 강제하기 위해서는 애너테이션 처리기를 직접 구현해야 한다. 

```java
// 마커 애너테이션을 사용한 프로그램 예
public class Sample {
	@Test public static void m1() {} // 성공해야 한다
	public static void m2() {}
	@Test public static void m3() { // 실패해야 한다
		throw new RuntimeException(“실패”);
	}
	public static void m4() {}
	@Test public void m5() {} // 잘못 사용한 예: 정적 메서드가 아니다
	public static void m6() {}
	@Test public static void m7() { // 실패해야 한다.
		throw new RuntimeException(“실패”);
	}
	public static void m8() {}
}
```

Sample 클래스에는 정적 메서드가 7개고, 그중 4개에 @Test를 달았다. m3와 m7메서드는 예외를 던지고 m1과 m5는 그렇지 않다. 그리고 m5는 인스턴스 메서드 이므로 @Test를 잘못 사용한 경우다. 요약하면 4개의 테스트 메서드 중 1개는 성공, 2개는 실패, 1개는 잘못 사용했다. @Test가 붙지 않은 나머지 4개는 테스트 도구가 무시한다.

애너테이션은 클래스의 의미에 직접적인 영향을 주지는 않지만, 이 애너테이션에 관심이 있는 프로그램에게 추가 정보를 제공한다. 즉, 대상 코드의 의미는 그대로 둔 채 그 애너테이션에 관심 있는 도구에서 특별한 처리를 할 기회를 준다. 

```java
// 마커 애너테이션을 처리하는 프로그램
import java.lang.reflect.*;

public class RunTests {
	public static void main(String[] args) throw Exception {
		int tests = 0;
		int passed = 0;
		Class<?> testClass = Class.forName(args[0]);
		for(Method m : testClass.getDeclaredMethods()) {
			if(m.isAnnotationPresent(Test.class)) {
				tests++;
				try {
					m.invoke(null);
					passed++;
				} catch(InvocationTargetException wrappedExc) {
					Throwable exc = wrappedExc.getCause();
					System.out.println(m + “ 실패: “+exc);
				} catch(Exception exc) {
					System.out.println(“잘못 사용한 @Test: “+m);
				}
			}
		}
		System.out.println(“성공: %d, 실패\: %d%n”,passed, tests-passed);
	}
}
```

이 테스트 러너는 명렬줄로부터 완전 정규화된 클래스 이름을 받아 그 클래스에서 @Test 애너테이션이 달린 메서드를 차례로 호출한다. (isAnnotationPresent가 실행할 메서드를 찾아준다)
테스트 메서드가 예외를 던지면 리플렉션 메커니즘이 InvocationTargetException으로 감싸서 다시 던진다. (원래 예외에 담긴 실패 정보를 getCause로 추출해 출력한다)

InvocationTargetException외의 예외가 발생하면 @Test 애너테이션을 잘못 사용한 것이다. (인스턴스 메서드, 매개변수가 있는 메서드, 호출할 수 없는 메서드 등)

RunTests로 Sample을 실행했을 때의 출력 메시지
```
public static void Sample.m3() failed: RuntimeException: Boom
Invalid @Test: public void Sample.m5()
public static void Sample.m7() failed: RuntimeException: Crash
성공: 1, 실패: 3
```


```java
// 매개변수 하나를 받는 애너테이션 타입
import java.langannotation.*;

/**
 * 명시한 예외를 던져야만 성공하는 테스트 메서드용 애너테이션
 */
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface ExceptionTest {
	Class<? extends Throwable> value();
}
```

이 애너테이션의 매개변수 타입은 Class<? extends Throwable>인데, Throwable을 확장한 클래스의 Class 객체라는 뜻이기 때문에 모든 예외 타입을 다 수용한다. (한정적 타입 토큰-아이템33)

```java
// 매개변수 하나짜리 애너테이션을 사용한 프로그램
public class Sample2 {
	@ExceptionTest(ArithmeticException.class)
	public static void m1() { // 성공해야 한다
		int i = 0;
		i = i/i;
	}
	@ExceptionTest(ArithmeticException.class)
	public static void m2() { // 실패해야 한다 (다른 예외 발생)
		int[] a = new int[0];
		int i = a[1];
	}
	@ExceptionTest(ArithmeticException.class)
	public static void m3() {} // 실패해야 한다. (예외 발생하지 않음)
}


// 마커 애너테이션을 처리하는 프로그램
import java.lang.reflect.*;

public class RunTests {
	public static void main(String[] args) throw Exception {
		int tests = 0;
		int passed = 0;
		Class<?> testClass = Class.forName(args[0]);
		for(Method m : testClass.getDeclaredMethods()) {
			if(m.isAnnotationPresent(ExcptionTest.class)) {
				tests++;
				try {
					m.invoke(null);
					System.out.printf(“테스트 %s 실패: 예외를 던지지 않음%n”,m);
				} catch(InvocationTargetException wrappedEx) {
					Throwable exc = wrappedEx.getCause();
					Class<? extends Throwable> excType = m.getAnnotation(ExceptionTest.class).value();
					if(excType.isInstance(exc)) {
						passed++;
					} else {
						System.out.printf(“테스트 %s 실패: 기대한 예외 %s, 발생한 예외 %s%n”,m, excType.getName(), exc);
				} catch(Exception exc) {
					System.out.println(“잘못 사용한 @ExceptionTest: “+m);
				}
			}
		}
		System.out.println(“성공: %d, 실패\: %d%n”,passed, tests-passed);
	}
}
```

@Test 애너테이션용 코드와 비슷한데, 이 코드는 애너테이션 매개변수의 값을 추출하여 테스트 메서드가 올바른 예외를 던지는지 확인하는데 사용한다. 형변환 코드가 없으므로 ClassCastException 걱정은 없다. 테스트 프로그램이 문제없이 컴파일 되면 애너테이션 매개변수가 가리키는 예외가 올바른 타입이라는 뜻이다.  (단, 해당 예외의 클래스 파일이 컴파일 타임에는 존재했지만 런타임에는 존재하지 않을 수 있다. 이런 경우라면 테스트 러너가 TypeNotPresentException을 던진다)


예외를 여러개 명시하고 그중 하나가 발생하면 성공하게 만들 수도 있다. 애너테이션 메커니즘에는 이런 쓰임에 아주 유용한 기능이 기본으로 들어 있다. @ExceptionTest 애너테이션의 매개변수 타입을 Class 객체의 배열로 수정해보면
```java
// 배열 매개변수를 받는 애너테이션 타입
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface ExceptionTest {
	Class<? extends Throwable>[] value();
}
```

```java
// 배열 매개변수를 받는 애너테이션을 사용하는 코드
@ExceptionTest({ IndexOutOfBoundsException.class, NullPointerException.class })
public static void doublyBad() { // 성공해야 한ㄷ
	List<String> list = new ArrayList<>();

	// 자바 API 명세에 따르면 다음 메서드는 IndexOutOfBoundException이나
	// NullPointerException을 던질 수 있다
	list.addAll(5, null);
}
```

```java
// 테스트 러너 일부
if(m.isAnnotationPresent(ExceptionTest.class)) {
	tests++;
	try {
		m.invoke(null);
		System.out.prinf(“테스트 %s 실패: 예외를 던지지 않음%n”,m);
	} catch(Throwable wrappedExc) {
		Throwable exc = wrappedExc.getCause();
		int oldPassed = passed;
		Class<? extends Throwable>[] excTypes = m.getAnnotation(ExceptionTest.class).value();
		for(Class<? extends Throwable> excType : excTypes) {
			if(excType.isInstance(exc)) {
				passed++;
				break;
			}
		}
		if(passed == oldPassed)
			System.out.printf(“테스트 %s 실패: %s %n”, m, exc);
	}
}
```
-> 예외중 하나가 발생하면 통과


자바 8에서는 여러 개의 값을 받는 애너테이션을 다른 방식으로 만들 수 있다. @Repeatable 메타애너테이션을 다는 방식이다. @Repeatable을 단 애너테이션을 하나의 프로그램 요소에 여러번 달 수 있는데 주의할점이 있다
- @Repeatable을 단 애너테이션을 반환하는 컨테이너 애너테이션을 하나 더 정의하고 @Repeatable에 이 컨테이너 애너테이션의 class 객체를 매개변수로 전달해야 한다.
- 컨테이너 애너테이션은 내부 애너테이션 타입의 배열을 반환하는 value 메서드를 정의해야 한다. 
- 컨테이너 애너테이션 타입에는 적절한 보존 정책과 적용 대상을 명시해야 한다. 그렇지 않으면 컴파일되지 않는다.

```java
// 반복 가능한 애너테이션 타입
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
@Repeatable(ExceptionTestContainer.class)
public @interface ExceptionTest {
	Class<? extends Throwable> value();
}

// 컨테이너 애너테이션
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface ExceptionTestContainer {
	ExceptionTest[] value();
}
```

```java
// 반복 가능 애너테이션을 두 번 단 코드
@ExceptionTest(IndexOutOfBoundsException.class)
@ExceptionTest(NullPointerException.class)
public static void doublyBad() { … }
```

반복 가능 애너테이션은 처리할 때도 주의를 요한다. 반복 가능 애너테이션을 여러개 달면 하나만 달았을 때와 구분하기 위해 해당 컨테이너 애너테이션 타입이 적용된다. getAnnotationsByType 메서드는 이 둘을 구분하지 않아서 반복 가능 애너테이션과 그 컨테이너 애너테이션을 모두 가져오지만, isAnnotationPresent 메서드는 둘을 구분한다. 
-> 반복 가능 애너테이션을 여러번 달고 isAnnotationPresent로 반복 가능 애너테이션이 달렸는지 검사하면 그렇지 않다고 알린다. (컨테이너가 달렸기 때문에)
즉, 여러번 단 메서드는 모두 무시하고 지나친다. (같은 이유로 isAnnotationPresent로 컨테이너 애너테이션이 달렸는지 검사하면 반복가능 애너테이션을 한번만 단 메서드는 무시하고 지나간다)

```java
// 반복 가능 애너테이션 다루기
if(m.isAnnotationPresent(ExceptionTest.class) || m.isAnnotationPresent(ExceptionTestContainer.class)) {
	tests++;
	try {
		m.invoike(null);
		System.out.printf(“테스트 %s 실패: 예외를 던지지 않음%n”, m);
	} catch(Throwable wrappedExc) {
		Throwable exc = wrappedExc.getCause();
		int oldPassed = passed;
		ExceptionTest[] excTests = m.getAnnotationsByType(ExceptionTest.class);
		for(ExceptionTest exactest : excTests) {
			if(exactest.value().isInstance(exc)) {
				passed++;
				break;
			}
		}
		if(passed == oldPassed)
			System.out.printf(“테스트 %s 실패: %s %n”, m, exc);
	}
}
```
-> 애너테이션을 선언하고 이를 처리하는 부분의 코드 양이 늘어남 (오류 가능성 높아짐)

도구 제작자를 제외하고는 일반 프로그래머가 애너테이션 타입을 직접 정의할 일은 거의 없지만, 자바 프로그래머라면 예외 없이 자바가 제공하는 애너테이션 타입들을 사용해야 한다.(아이템40) IDE나 정적 분석 도구가 제공하는 애너테이션을 사용하면 해당 도구가 제공하는 진단 정보의 품질을 높여준다. (애너테이션으로 할수 있는 일을 명명 패턴으로 처리할 이유는 없음)






