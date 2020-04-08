# #EffectiveJava/9장_일반적인프로그래밍원칙/61박싱기본타입


## 61. 박싱된 기본 타입 보다는 기본 타입을 사용하라


자바는 크게 2가지 타입으로 나뉜다. 기본 타입과 참조 타입. 기본 타입과 대응하는 참조 타입이 있으며 이를 박싱된 기본타입이라 한다.


아이템6 에서 언급한것 처럼 오토박싱과 오토언박싱 덕분에 큰 구분없이 사용 가능하지만, 차이가 사라지는 것이 아니기 때문에 주의해서 사용해야 한다.

차이는 크게 3가지 이다
1. 기본 타입은 값만 가지고 있으나, 박싱된 기본 타입은 값에 더해 식별성이란 속성을 갖는다. (박싱된 두 인스턴스는 값이 같더라도 다르다. 객체이기 때문에 주소값이 다름)
2. 기본 타입의 값은 언제나 유효하나, 박싱된 기본 타입은 null을 가질 수 있다.
3. 기본 타입이 박싱된 기본 타입보다 시간과 메모리 사용면에서 더 효율적이다.



```java
// 잘못 구현된 비교자 - 문제를 찾기!
Comparator<Integer> naturalOrder = 
	(i,j) -> (i < j) ? -1 : (i == j ? 0 : 1);
```
-> 값 비교가 아닌 식별성 비교가 일어난다 (주소 값을 비교함)
즉, 박싱된 기본 타입에 == 연산자를 사용하면 오류가 일어 난다
(실무에서 기본타입을 다루는 비교자가 필요하면 Comparator.naturalOrder()를 사용)

```java
// 문제를 수정한 비교자
Comparator<Integer> naturalOrder = (iBoxed, jBoxed) -> {
	int i = iBoxed, j = jBoxed; // 오토박싱
	return i < j ? -1 : (i == j ? 0 : 1);
}
```


```java
// NullPointerException 발생
public class Unbelievable {
	staic Integer i;
	
	public static void main(String[] args) {
		if(i == 42) // 기본 타입과 혼용하면 박싱이 풀림
			System.out.println(“믿을 수 없군!”);
	}
}

// 끔찍이 느리다. 객체가 만들어지는 위치는?
public static void main(String[] args) {
	Long sum = 0L;
	for(long i=0 ; i<=Integer.MAX_VALUE ; i++) {
		sum += i; // 계속해서 박싱과 언박싱이 반복해서 일어남
	}
	System.out.println(sum);
}
```


박싱된 타입이 사용되는 경우는 ..
1. 컬렉션의 원소, 키 값으로 쓴다
2. 리플렉션(아이템65)을 통해 메서드를 호출할 때 박싱된 기본 타입을 쓴다


> **::핵심 정리::** 
> 기본타입은 간단하고 빠르다. 기본 타입을 주로 사용하되 박싱된 기본 타입을 쓰는 경우에 주의를 기울여야 한다. (오토박싱 문제, == 비교, 같은 연산에서 기본 타입과 박싱된 기본타입 혼용시 언박싱으로 NullPointerException 발생, 객체 생성 부작용)  


