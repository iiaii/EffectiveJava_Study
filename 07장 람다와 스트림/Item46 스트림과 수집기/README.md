# #EffectiveJava/7장_람다와스트림/46스트림과수집기

## 46. 스트림에서는 부작용 없는 함수를 사용하라

스트림은 함수형 프로그래밍에 기초한 패러다임이기 때문에 스트림이 제공하는 표현력, 속도, (상황에 따라서는) 병렬성을 얻으려면 API와 패러다임까지 받아 들여야 한다.

스트림 패러다임의 핵심은 계산을 일련의 변환으로 재구성하는 부분이다. 이때 각 변환 단계는 가능한 한 이전 단계의 결과를 받아 처리하는 순수 함수여야 한다. (순수 함수 : 오직 입력만이 결과에 영향을 주는 함수, 다른 가변상태를 참조하지 않고, 함수 스스로도 다른 상태를 변경하지 않는다)
-> 스트림 연산에 건네는 함수 객체는 모두 부작용이 없어야 한다

```java
// 스트림 패러다임을 이해하지 못한채 API만 사용 - 따라 하지 말 것!
Map<String, Long> freq = new HashMap<>();
try(Stream<String> words = new Scanner(file).tokens()) {
	words.forEach(word -> {
		freq.merge(word.toLowerCase(), 1L, Long::sum);
	});
}
```

스트림, 람다, 메서드 참조를 사용하고 결과도 올바르지만 스트림 코드가 아니다. 이러한 코드는 스트림의 이점을 살리지 못하고 같은 기능의 반복적 코드보다 조금 더 길고, 읽기 어렵고, 유지보수에 좋지 않다. 
모든 작업이 종단연산인 forEach에서 일어나는데 외부 상태를 수정하는 람다를 실행하면서 문제가 생긴다. forEach가 그저 스트림이 수행한 연산 결과를 보여주는 일 이상을 하는 것(이 예에서는 람다가 상태를 수정함)이 좋지 못하다.

```java
// 스트림을 제대로 활용해 빈도표를 초기화
Map<String, Long> freq;
try(Stream<String> words = new Scanner(file).tokens()) {
	freq = words
		.collect(groupingBy(String::toLowerCase, counting()));
}
```

> Scanner 메서드인 tokens는 자바9부터 지원한다

-> **forEach 연산은 스트림 계산 결과를 보고할 때만 사용하고, 계산하는 데는 쓰지 말자** (가씀 스트림 계산 결과를 기존 컬렉션에 추가하는 용도로 쓰일수 있음)

위 코드는 collector를 사용하는데, 스트림을 사용하려면 꼭 배워야하는 새로운 개념이다. (축소 전략을 캡슐화한 블랙박스, 스트림의 원소들을 객체 하나에 취합하는데 collector가 생성하는 객체는 일반적으로 컬렉션이다)

collector를 사용하면 스트림의 원소를 손쉽게 컬렉션으로 모을 수 있다. 
- toList() -> List 반환
- toSet() -> Set 반환
- toCollection(collectionFactory) -> 지정한 컬렉션 타입 반환

```java
// 빈도표에서 가장 흔한 단어 10개를 뽑아내는 파이프라인
List<String> topTen = freq.KeySet().stream()
	.sorted(comparing(freq::get).reversed())
	.limit(10)
	.collect(toList());
```

> toList는 Collectors의 정적 임포트한 메서드

comparing 메서드는 키 추출 함수를 받는 비교자(Comparator) 생성 메서드다.(아이템14) 그리고 한정적 메서드 참조이자 키 추출함수로 쓰인 freq::get은 입력받은 단어를 빈도표에서 찾아 그 빈도를 반환한다. 가장 흔한 단어가 위로 오도록 비교자를 역순으로 정렬하고 스트림에서 10개를 뽑아 리스트에 담는다.


Collectors의 대부분 메서드는 스트림을 맵으로 취합하는 기능으로, 진짜 컬렉션에 취합하는 것보다 복잡하다. 스트림의 각 원소는 키 하나와 값 하나에 연관되어 있고 다수의 스트림은 원소가 같은 키에 연관될 수 있다.


##### toMap

가장 간단한 맵 수집기(collector)는 toMap(keyMapper, valueMapper)로 스트림 원소를 키에 매핑하는 함수와 값에 매핑하는 함수를 인수로 받는다.

```java
// toMap 수집기(collector)를 사용하여 문자열을 열거 타입 상수에 매핑한다
private static final Map<String, Operation> stringToEnum = 
	Stream.of(values()).collect(
		toMap(Object::toString, e -> e));
```

각 원소가 고유한 키에 매핑되어 있을 때 적합하며, 스트림 원소 다수가 같은 키를 사용하면 파이프라인이 IllegalStateException을 던지며 종료된다.

복잡한 형태의 toMap이나 groupingBy는 이런 충돌을 다루는 다양한 전략을 제공한다.  키 매퍼와 값 매퍼, 병합 함수까지 제공할 수 있다. (병합 함수 형태은 BinaryOperator<U> 이며 U는 해당 맵의 값 타입) -> 같은 키를 공유하는 값들은 병합 함수를 사용해 기존 값에 합쳐진다. (병합 함수가 곱셈이면 모든 값을 곱한 결과)

인수 3개를 받는 toMap은 어떤 키와 그 키에 연관된 원소들 중 하나를 골라 연관 짓는 맵을 만들 때 유용하다. 

```java
// 예를 들어 다양한 음악가의 앨범들을 담은 스트림을 가지고, 음악가와 그 음악가의 베스트 앨범을 연관 짓고 싶은 경우
// 각 키와 해당 키의 특정 원소를 연관 짓는 맵을 생성하는 수집기(collector)
Map<Artist, Album> topHits = albums.collect(
	toMap(Album::artist, a->a, maxBy(comparing(Album::sales))));
```

비교자로는 BinaryOperator 에서 정적 임포트한 maxBy라는 정적 팩터리 메서드를 사용했다. (maxBy는 Comparator<T>를 입력받아 BinaryOperator<T>를 돌려 준다)
Comparing이  maxBy에 넘겨줄 비교자를 반환하는데, 자신의 키 추출 함수로 Album::sales를 받는다. 
-> 앨범 스트림을 맵으로 바꾸는데, 이 맵은 각 음악가와 그 음악가의 베스트 앨범을 짝지은 것

```java
// 충돌이 나면 마지막에 쓴 값을 취하는 수집기(collector) (collector)
toMap(keyMapper, valueMapper, (oldVal, newVal) -> newVal)
```
(이외에 자세한 것은 API 확인)


##### Collectors 가 제공하는 groupingBy

이 메서드는 입력으로 분류 함수를 받고 출력으로 원소들을 카테고리별로 모아 놓은 맵을 담은 collector 를 반환 (분류함수는 입력받은 원소가 속하는 카테고리를 반환하고 이 카테고리가 해당 원소의 맵 키로 쓰인다)

`words.collect(groupingBy(word -> alphabetize(word)))`
-> 알파벳화한 단어를 알파벳화 한 결과가 같은 단어들의 리스트로 매핑하는 맵을 생성

groupingBy 가 반환하는 collector 가 리스트 외의 값을 갖는 맵을 생성하게 하려면 분류함수와 함께 다운스트림 collector 도 명시해야 한다. (다운스트림 collector 은 해당 카테고리의 모든 원소를 담은 스트림으로부터 값을 생성함)

```java
Map<String, Long> freq = words
	.collect(groupingBy(String::toLowerCase, counting()));
```
-> 각 카테고리(키)를 해당 카테고리에 속하는 원소의 개수와 매핑한 맵을 얻음
(다양한 기능 및 partitioningBy (분류함수 자리에 프레디키트를 받고 Boolean인 맵을 반환) 와 같은 것도 있는데 이외에 자세한 것은 API 확인)

Collections 에는 counting 과 같은 속성의 메서드가 여러가지 있다. (summing, averaging, summarizing 으로 시작하며 int, long, double 이 각각 존재)

다중정의된 reducing 메서드 filtering, mapping, flatMapping, collecting, AndThen 메서드가 있음. (설계 관점에서 보면 이 수집기들은 스트림 기능의 일부를 복제하여 다운스트림 수집기 (collector)를 작은 스트림처럼 동작하게 함)


Collectors 에서 minBy, maxBy 는 인수로 받은 비교자를 이용해 스트림에서 값이 가장  작은 혹은 가장 큰 원소를 찾아 반환한다. Stream 인터페이스의 min과 max 메서드를 (살짝) 일반화 한것임

Collectors의 joining은 CharSequence 인스턴스의 스트림에만 적용 가능하다. 이중 매개변수가 없는 joining은 단순히 원소들을 연결하는 수집기(collector)를 반환한다. (이외의 오버로딩 된 메서드는 API 확인)


::핵심 정리:: 

> 스트림 파이프라인 프로그래밍의 핵심은 부작용 없는 함수 객체에 있다. 스트림뿐 아니라 스트림 관련 객체에 건네지는 모든 함수 객체가 부작용이 없어야 한다. 종단 연산 중 forEach는 스트림이 수행한 계산 결과를 보고할 때만 이용해야 한다. 계산 자체에는 이용하지 말자. 스트림을 올바로 사용하려면 수집기 (collector)를 잘 알아야 한다. 가장 중요한 수집기 팩토리는 toList, toSet, toMap, groupingBy, joining 이다






