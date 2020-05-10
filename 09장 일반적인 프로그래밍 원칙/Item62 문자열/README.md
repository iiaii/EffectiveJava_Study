# #EffectiveJava/9장_일반적인프로그래밍원칙/62문자열


## 62. 다른 타입이 적절하다면 문자열 사용을 피하라


##### 문자열이 적합하지 않은 경우

1. 문자열은 다른 값 타입을 대신하기에 적합하지 않다. (int, float, BigInteger 등의 적당한 수치 타입으로 변환해야 한다)

2. 문자열은 열거 타입을 대신하기에 적합하지 않다. (아이템34 열거 타입 활용하자!)

3. 문자열은 혼합 타입을 대신하기에 적합하지 않다.

```java
// 혼합 타입을 문자열로 처리 - 잘못된 예
String compoundKey = className + “#” + i.next();
```

두 요소를 구분해 주는 “#”이 더 쓰이면 혼란스러워 지며, 문자열 파싱으로 인해 느리고, 귀찮고, 오류 가능성이 커진다.
그래서 이런 경우 전용 클래스를 새로 만드는 것이 낫다.

4. 문자열은 권한을 표현하기에 적합하지 않다.

```java
// 문자열을 사용해 권한을 부여 - 잘못된 예
public class ThreadLocal {
	private ThreadLocal() { } // 객체 생성 불가

	// 현 스레드의 값을 키로 구분해 저장
	public static void set(String key, Object value);

	// 키가 가리키는 현 스레드의 값을 반환
	public static Object get(String key);
}

// Key 클래스로 권한을 구분
public class ThreadLocal {
	private ThreadLocal() { } 객체 생성 불가

	public static class Key { // 권한
		Key() { }
	}

	// 위조 불가능한 고유 키를 생성
	public static Key getKey() {
		return new Key();
	}

	public static void set(Key key, Object value);
	public static Object get(Key key);
}
```


이 방법은 앞서 문자열 기반 API의 문제 두가지를 모두 해결해주지만, 개선할 여지가 있다. set과 get은 이제 정적 메서드일 이유가 없으니 Key 클래스의 인스턴스 메서드로 바꾸자. 이렇게 하면 Key는 더 이상 스레드 지역변수를 구분하기 위한 키가 아니라, 그 자체가 스레드 지역변수가 된다. 결과적으로 지금의 톱레벨 클래스인 ThreaadLocal은 별달리 하는 일이 없어지므로 치워버리고 Key의 이름을 바꾸면

```java
// 리펙터링하여 Key를 ThreadLocal로 변경
public final class ThreadLocal {
	public ThreadLocal();
	public void set(Object value);
	public Object get();
}

// 매개변수화하여 타입 안정성 확보
public final class ThreadLocal<T> {
	public ThreadLocal();
	public void set(T value);
	public T get();
}
```
-> java.lang.ThreadLocal 과 흡사하면서도 문자열 기반 API 문제를 해결하고 빠르다


**::핵심 정리::** 

> 더 적합한 데이터 타입이 있거나 새로 작성 가능하면 그렇게 하라. 문자열은 잘못 사용하면 번거롭고, 덜 유연하고, 느리고, 오류 가능성도 크다. (기본 타입, 열거 타입, 혼합 타입 문제 해결하기!)






