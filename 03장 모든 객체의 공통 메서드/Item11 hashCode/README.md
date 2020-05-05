# #EffectiveJava/3장_모든객체의공통메서드/11hashCode

## 11. equals를 재정의하려거든 hashCode도 재정의하라

equals를 재정의한 클래스 모두에서 hashCode도 재정의해야 한다. 그렇지 않으면 hashCode 일반 규약을 어기게 되어 해당 클래스의 인스턴스를 HashMap이나 HashSet 같은 컬렉션의 원소로 사용할 때 문제를 일으킨다.

##### Object 명세
> 규약1. equals 비교에 사용되는 정보가 변경되지 않았다면, 애플리케이션이 실행되는 동안 그 객체의 hashCode 메서드는 몇 번을 호출해도 일관되게 항상 같은 값을 반환해야 한다
> (단, 애플리케이션을 다시 실행한다면 이 값이 달라져도 상관없다)

> 규약2. equals(Object)가 두 객체를 같다고 판단했다면, 두 객체의 hashCode는 똑같은 값을 반환해야 한다

> 규약3. equals(Object)가 두 객체를 다르다고 판단했더라도, 두 객체의 hashCode가 서로 다른 값을 반환할 필요는 없다 단, 다른 객체에 대해서는 다른 값을 반환해야 해시테이블의 성능이 좋아진다

hashCode 재정의를 잘못했을 때 크게 문제가 되는 조항은 두 번째다. 즉, 논리적으로 같은 객체는 같은 해시코드를 반환해야 한다. 아이템 10에서 보았듯이 equals는 물리적으로 다른 두 객체를 논리적으로 같다고 할 수 있다. 하지만 Object의 기본과 hashCode 메서드는 이 둘이 전혀 다르다고 판단하여, 규약과 달리 서로 다른 값을 반환한다.

```java
Map<PhoneNumber, String> m = new HashMap<>();
m.put(new PhoneNumber(707,867,5309), “제니”);
m.get(new PhoneNumber(707,867,5309)); // null

// 최악의 hashCode 구현 - 사용금지!
@Override
public int hashCode() { return 42; }
```

위의 hashCode는 모든 객체에서 똑같은 해시코드를 반환하니 바로 위의 예시에서는 통과하지만 해시테이블의 버킷 하나에 담겨 마치 LinkedList 처럼 작동한다. (평균 수행 시간이 O(1)에서 O(n)으로 느려진다)

좋은 해시 함수라면 서로 다른 인스턴스에 다른 해시코드를 반환한다. (세 번째 규약)
이상적인 해시함수는 주어진 인스턴스들을 32비트 정수 범위에 균일하게 분배해야 한다. 

###### hashCode를 작성하는 간단한 요령
1. int 변수 result를 선언한 후 값 c로 초기화 한다. 이때 c는 해당 객체의 첫 번째 핵심 필드를 단계 2.a 방식으로 계산한 해시코드다. (여기서 핵심필드란 equals에 비교에 사용되는 필드)

2. 해당 객체의 나머지 핵심 필드 f 각각에 대해 다음 작업을 수행한다.
	
a. 해당 필드의 해시코드 c를 계산한다
		
i. 기본 타입 필드라면, Type.hashCode(f)를 수행한다. (Type은 기본타입 박싱 클래스)	
ii. 참조 타입 필드면서 이 클래스의 equals 메서드가 이 필드의 equals를 재귀적으로 호출해 비교한다면, 이 필드의 hashCode를 재귀적으로 호출한다. 계산이 더 복잡해질 것 같으면, 이 필드의 표준형을 만들어 그 표준형의 hashCoode를 호출한다. (필드의 값이 null이면 0을 사용)
iii. 필드가 배열이라면 핵심 원소 각각을 별도 필드처럼 다룬다. 이상의 규칙을 재귀적으로 적용해 각 핵심 원소의 해시코드를 계산한 다음, 단계2.b방식으로 개선한다. 배열에 핵심 원소가 하나도 없다면 단순히 상수(0을 추천)를 사용한다. 모든 원소가 해시 원소라면 Arrays.hashCode를 사용한다

b. 단계 2.a에서 계산한 해시코드 c로 result를 갱신한다. result = 31*result+c;

3. result를 반환한다


hashCode를 다 구현했다면 이 메서드가 동치인 인스턴스에 대해 똑같은 해시코드를 반환할지 자문하자. 그리고 직관을 검증할 단위 테스트를 작성하자. (AutoValue로 작성했다면 건너뛰어도 됨)

파생 필드는 해시코드 계산에서 제외해도 된다. 즉, 다른 필드로부터 계산해 낼 수 있는 필드는 모두 무시해도 된다. 또한 equals 비교에 사용되지 않은 필드는 반드시 제외해야 한다. (그렇지 않으면 두 번째 규약을 어기게 됨)

곱할 숫자를 31로 정한 이유는 31이 홀수이면서 소수이기 때문이다. 2를 곱하는 것은 시프트 연산과 같은 결과를 냄


```java
// 전형적인 hashCode 메서드
@Override
public int hashCode() {
	int result = Short.hashCode(areaCode);
	result = 31 * result + Short.hashCode(prefix);
	result = 31 * result + Short.hashCode(lineNum);
	return result;
}
```

이 메서드는 PhoneNumber 인스턴스의 핵심 필드 3개만을 사용해 간단한 계산만 수행한다. 그 과정에 비결정적 요소는 전혀 없으므로 동치인 PhoneNumber 인스턴스들은 같은 해시코드를 가질 것이 확실하다. (위의 코드는 PhoneNumber에 딱 맞게 구현한 hashCode이다. 품질적으로도 뒤지지 않는다)

```java
// 성능이 살짝 아쉬운 hashCode 메서드
@Override
public int hashCode() {
	return Objects.hash(lineNum, prefix, areaCode);
}
```


클래스가 불변이고 해시코드를 계산하는 비용이 크다면 매번 새로 계산하기 보다는 캐싱하는 방식을 고려해야 한다. 이 타입의 객체가 주로 해시의 키로 사용될 것 같다면 인스턴스가 만들어질 때 해시코드를 계산해둬야 한다. 해시의 키로 사용되지 않는 경우라면 hashCode가 처음 불릴 때 계산하는 지연 초기화 전략도 가능하다. (대신 스레드 안전성을 고려해야 함)

```java
// 해시코드를 지연 초기화하는 hashCode 메서드 - 스레드 안정성 고려
private int hashCode; // 자동으로 0

@Override
public int hashCode() {
	int result hashCode;
	if(result == 0) {
		result = Short.hashCode(areaCode);
		result = 31*result + Short.hashCode(prefix);
		result = 31*result + short.hashCode(lineNum);
		hashCode = result;
	}
	return result;
}
```


성능을 높이겠다고 해시코드를 계산할 때 핵심 필드를 생략하면 안된다. 속도야 빠르겠지만 해시의 품질이 나빠져 해시테이블의 성능을 심각하게 떨어뜨릴 수 있다.

hashCode가 반환하는 값의 생성 규칙을 API사용자에게 자세히 공표하지 말자. 그래야 클라이언트가 이 값에 의지하지 않게 되고, 추후에 계산 방식을 바꿀 수도 있다.


::핵심 정리:: 

> eauals를 재정의 할 때는 hashCode도 반드시 재정의 해야한다. 그렇지 않으면 프로그램이 제대로 동작하지 않는다. 재정의한 hashCode는 Object의 API문서에 기술된 일반 규약을 따라야 하면, 서로 다른 인스턴스라면 되도록 해시코드도 서로 다르게 구현해야 한다. 이렇게 구현하기가 어렵지는 않지만 조금 따분하긴 하다. 하지만 AutoValue 프레임 워크를 사용하면 equals와 hashCode를 자동으로 만들어 준다. (IDE도 가능)



