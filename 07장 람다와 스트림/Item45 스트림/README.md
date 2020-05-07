# #EffectiveJava/7장_람다와스트림/45스트림

## 45. 스트림은 주의해서 사용하라

스트림 API는 다량의 데이터 처리 작업을 돕고자 자바8에 추가 되었다. 이 API가 제공하는 추상 개념 중 핵심은 2가지다.

- 스트림은 데이터 원소의 유한 혹은 무한 시퀀스를 뜻한다
- 스트림 파이프라인은 이 원소들로 수행하는 연산 단계를 표현하는 개념이다

스트림의 원소들은 어디로부터든 올 수 있다. 대표적으로는 컬렉션, 배열, 파일 정규표현식 패턴 매처, 난수 생성기 등 다른 스트림이 있다. 스트림 안의 데이터 원소들은 객체 참조나 기본 타입 값이다. 기본 타입 값으로는 int, long, double 이렇게 3가지를 지원한다.

스트림 파이프라인은 소스 스트림에서 시작해 종단 연산으로 끝나며, 그 사이에 하나 이상의 중간 연산이 있을 수 있다. 각 중간 연산은 스트림을 어떠한 방식으로 변환한다. 
(예컨대 각 원소에 함수를 적용하거나 특정 조건을 만족 못하는 원소를 걸러낼 수 있다. 중간 연산들은 모두 한 스트림을 다른 스트림으로 변환하는데, 변환된 스트림의 원소 타입은 변화전 스트림의 원소 타입과 같을 수도 있고 다를 수도 있다)
종단 연산은 마지막 중간 연산이 내놓은 스트림에 최후의 연산을 가한다. 원소를 정렬해 컬렉션에 담거나, 특정 원소 하나를 선택하거나, 모든 원소를 출력하는 식이다.


스트림 파이프라인은 지연평가(lazy evaluation)된다. 평가는 종단 연산이 호출될 때 이뤄지며, 종단 연산에 쓰이지 않는 데이터 원소는 계산에 쓰이지 않는다. 이러한 지연 평가가 무한 스트림을 다룰 수 있게 해주는 열쇠다. 종단 연산이 없는 스트림 파이프라인은 아무 일도 하지 않는 명령어인 no-op과 같으므로 종단연산을 빼면 안된다.

스트림 API는 메서드 연쇄를 지원하는 플루언트 API다. 즉, 파이프라인 하나를 구성하는 모든 호출을 연결하여 단 하나의 표현식으로 완성할 수 있다. 파이프라인 여러 개를 연결해 표현식 하나로 만들 수도 있다. 
기본적으로 스트림 파이프라인은 순차적으로 수행된다. 파이프라인을 병렬로 실행하려면 파이프라인을 구성하는 스트림중 하나에서 parallel메서드를 호출해주기만 하면 되지만, 효과를 볼 수 있는 상황은 많지 않다(아이템48)

스트림을 제대로 사용하면 프로그램이 짧고 깔끔해지지만, 잘못 사용하면 읽기 어렵고 유지보수도 힘들어 진다. 

```java
// 사용자가 명시한 사전파일에서 각 단어를 읽어 맵에 저장하는 프로그램
// 사전 하나를 훑어 원소 수가 많은 아나그램 그룹들을 출력
public class Anagrams {
	public static void main(String[] args) throws IOException {		File dictionary = new File(args[0]);
		int minGroupSize = Integer.parseInt(args[1]);

		Map<String, Set<String>> groups = new HashMap<>();
		try(Scanner s = new Scanner(dictionary)) {
			while(s.hasNext()) {
				String word = s.next();
				groups.computeIfAbsent(alphabetize(word),
					(unused) -> new TreeSet<>()).add(word);
				// 맵안에 키가 있는지 찾고, 있으면 그 키에 매핑된 값 반환
				// 없으면 함수 객체를 키에 적용한 값을 매핑하고 반환
			}
		}
		for(Set<String> group : groups.values())
			if(group.size() >= minGroupSize)
				System.out.println(group.size()+”: “+group);
	}
	private static String alphabetize(String s) {
		char[] a = s.toCharArray();
		Arrays.sort(a);
		return new String(a);
	}
}
```
-> computeIfAbsent 메서드를 활용하면 다수의 값을 매핑하는 맵을 쉽게 구현할 수 있다.

```java
// 스트림을 과하게 사용했다 - 따라 하지 말것!
public class Anagrams {
	public static void main(String[] args) throws IOException {
		Path dictionary = Paths.get(args[0]);
		int minGroupSize = Integer.parseInt(args[1]);

		try (Stream<String> words = Files.lines(dictionary)) {
			words.collect(
				gourpingBy(word -> word.chars().sorted()
						.collect(StringBuilder::new,
							(sb, c) -> sb.append((char) c),
							StringBuilder::append).toString()))
			.values().stream()
			.filter(group -> group.size() >= minGroupSize)
			.map(group -> group.size() +”: “+group)
			.forEach(System.out::println);
		}
	}
}
```
-> 짧지만 읽기 어렵다. 스트림에 익숙하지 않으면 더욱 그렇고 스트림을 과도하게 사용하면 읽거나 유지보수하기 어려워진다.

```java
// 스트림을 적절히 사용하면 깔끔하고 명료해진다
public class Anagrams {
	public static void main(String[] args) throws IOException {
		Path dictionary = Path.get(args[0]);
		int minGroupSize = Integer.parseInt(args[1]);

		try(Stream<String> words = Files.lines(dictionary)) {
			words.collect(groupingBy(word -> alphabetize(word)))
				.values().stream()
				.filter(group -> group.size() >= minGroupSize)
				.forEach(g -> System.out.println(g.size()+”: “+g));
		}
	}
	// alphabetize 메서드는 동일
}
```

try-with-resources 블록에서 사전 파일을 열고, 파일의 모든 라인으로 구성된 스트림을 얻는다. 스트림 변수 이름을 words로 지어 스트림 안의 각 원소가 단어임을 명확히 했고, 스트림의 파이프라인에는 중간 연산은 없으며, 종단 연산에서 모든 단어를 수집해 맵으로 모은다. 이 맵은 단어들을 아나그램끼리 묶어놓은 것으로(아이템46), 앞선 두 프로그램이 생성한 맵과 실질적으로 같다. 

그다음 values()가 반환한 값으로 부터 새로운 Stream<List<String>> 스트림을 연다. (이 스트림의 원소는 아나그램 리스트) 그 리스트들 중 원소가 minGroupSize 보다 적은 것은 필터링돼 무시된다. 마지막으로 종단 연산인 forEach는 살아남은 리스트를 출력한다.

> 람다 매개변수의 이름은 주의해서 정해야 한다. 앞 코드에서 매개변수 g는 group이라고 하는게 낫다. (지면 폭이 부족해서 줄임) 람다에서는 타입 이름을 자주 생략하므로 매개변수 이름을 잘 지어야 스트림 파이프라인의 가독성이 유지된다.
>   한편 단어의 철자를 알파벳순으로 정렬하는 일은 별도 메서드인 alphabetize에서 수행했다. 연산에 적절한 이름을 지어주고 세부 구현을 주 프로그램 로직 밖으로 빼내 전체적인 가독성을 높인다. 도우미 메서드를 적절히 활용하는 일의 중요성은 일반 반복 코드에서보다 스트림 파이프라인에서 훨씬 크다. (파이프라인에서는 타입 정보가 명시되지 않거나 임시 변수를 자주 사용하기 때문

alphabetize 메서드도 스트림을 사용해 다르게 구현할 수 있다. 하지만 그렇게 하면 명확성이 떨어지고 잘못 구현할 가능성이 커진다. 심지어 느려 질수도 있다. (자바가 기본타입인 char용 스트림을 지원하지 않기 때문)

`“Hello world!”.chars().forEach(System.out::print);`

위의 출력 값은 710111… 과 같은 숫자이다. chars()가 반환하는 스트림의 원소는 int값이기 때문이다. 그래서 형변환을 명시적으로 해야된다

`“Hello world!”.chars().forEach(x -> System.out.print((char) x));`
(하지만 char값들을 처리할 대는 스트림을 삼가는 편이 낫다)


스트림을 처음 쓰기 시작하면 모든 반복문을 스트림으로 바꾸고 싶은 유혹이 일지만, 서두르지 않는 것이 좋다. 스트림으로 바꾸는 것이 가능하더라도 가독성과 유지보수 측면에서 손해를 볼 수 있기 때문이다. 스트림과 반복문을 적절히 조합하는 게 최선이며, 기존 코드는 스트림을 사용하도록 리펙터링하되, 새 코드가 더 나아 보일 때만 반영하는 것이 좋다.


함수 객체로는 할 수 없지만 코드블록으로 할 수 있는 일이 있다

- 코드 블록에서는 범위 안의 지역변수를 읽고 수정할 수 있다. 하지만 람다에서는 final이거나 사실상 final인 변수만 읽을 수 있고, 지역변수를 수정하는 건 불가능하다.
- 코드 블록에서는 return문을 사용해 메서드에서 빠져나가거나, break나 continue 문으로 블록 바깥의 반복문을 종료하거나 반복을 한번 건너뛸 수 있다. 또한 메서드 선언에 명시된 검사 예외를 던질 수 있다. (람다로는 X)

계산 로직에서 이상의 일들을 수행하야 한다면 스트림과 맞지 않는다. 반대로 다음의 경우에는 스트림이 좋다

##### 스트림이 적합한 경우
- 원소들의 시퀀스를 일관되게 변환한다
- 원소들의 시퀀스를 필터링한다
- 원소들의 시퀀스를 하나의 연산을 사용해 결합한다 (더하기, 연결하기, 최솟값구하기 등)
- 원소들의 시퀀스를 컬렉션에 모든다 (공통된 속성을 기준으로 묶어가며)
- 원소들의 시퀀스에서 특정 조건을 만족하는 원소를 찾는다


스트림이 처리하기 어려운 일의 예로, 한 데이터가 파이프라인의 여러 단계를 통과 할 때 이 데이터의 각 단계에서 값들에 동시에 접근하기는 어려운 경우다. (스트림 파이프라인은 한 값을 다른 값에 매핑하고 나면 원래의 값은 잃는 구조이기 때문)
원래 값과 새로운 값의 쌍을 저장하는 객체를 사용해 매핑하는 우회방법도 있지만 별로다. (매핑 객체가 필요한 단계가 여러 곳이라면 특히)
가능한 경우라면 앞 단계의 값이 필요할 때 매핑을 거꾸로 수행하는 것이 낫다.

```java
// 처음 20개의 메르센 소수를 출력하는 프로그램
static Stream<BigInteger> primes() {
	return Stream.iterator(Two, BigInteger::nextProbablePrime);
}
public static void main(String[] args) {
	primes().map(p -> TWO.pow(p.intValueExact()).subtract(ONE))
		.filter(mersenne -> mersenne.isProbablePrime(50))
		.limit(20)
		.forEach(mp -> System.out.println(mp.bitLength()+”: “+mp);
}
```

```java
// 카드덱을 초기화 하는 작업 - 스트림 vs 반복
// 데카르트 곱 계산

// 반복문
private static List<Card> newDeck() {
	List<Card> result = new ArrayList<>();
	for(Suit suit : Suit.values())
		for(Rank rank : Rank.values())
			result.add(new Card(suit, rank));
	return result;
}

// 중간연산으로 사용한 flatMap은 스트림의 원소 각각을 하나의 스트림으로 매핑한다음 다시 하나의 스트림으로 합친다
// 스트림
private static List<Card> newDeck() {
	return Stream.of(Suit.values())
		.flatMap(Suit -> 
			Stream.of(Rank.values())
				.map(rank -> new Card(suit, rank)))
		.collect(toList());
}
```
-> 본인과 동료들이 이해하기 쉽고 선호하는 것을 사용하자


::핵심 정리:: 

> 스트림을 사용해야 멋지게 처리할 수 있는 일이 있고, 반복 방식이 더 알맞은 일도 있다.
> 그리고 수많은 작업이 이 둘을 조합했을 때 가장 멋지게 해결된다. 어느 쪽을 선택하든 상관 없지만, 참고할 만한 지침 정도는 있다. 스트림과 반복 중 어느쪽이 나은지 확신하기 어렵다면 둘다 해보고 더 나은 쪽을 택해라


