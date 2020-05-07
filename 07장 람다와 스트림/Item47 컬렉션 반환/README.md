# #EffectiveJava/7장_람다와스트림/47컬렉션반환

## 47. 반환 타입으로는 스트림보다 컬렉션이 낫다

원소 시퀀스(일련의 원소를 반환하는) 메서드의 반환 타입으로 Collection, Set, List 같은 컬렉션 인터페이스나 Iterable이나 배열을 사용했다. (기본은 컬렉션이지만, collection이 불가능하면 Iterable, 기본타입이거나 성능에 민감하면 배열 사용)
-> 스트림이 생기면서 선택이 복잡해짐

스트림은 반복을 지원하지 않기 때문에 스트림과 반복을 알맞게 조합해야 좋은 코드가 된다. (for-each로 반복하길 원하면 불만 발생)

Stream 인터페이스는 Iterable 인터페이스가 정의한 추상 메서드를 전부 포함하고 Iterable 인터페이스가 정의한 방식대로 동작하지만 Stream은 Iterable을 extends(확장)하지 않았기 때문에 for-each 로 스트림을 반복할 수 없다.


```java
// 자바 타입 추론의 한계로 컴파일 되지 않음
for(ProcessHandle ph : ProcessHandle.allProcesses()::iterator) {
	// 프로세스 처리
}

// 컴파일 오류 발생
Test.java:6: error: method reference not expected here
…

// 스트림을 반복하기 위한 끔찍한 우회 방법
for(ProcessHandle ph : (Iterable<ProcessHandle>)
						ProcessHandle.allProcesses()::iterator){
	//프로세스 처리
}
```
-> 작동은 하지만 난잡하고 직관성 떨어짐

위의 경우를 어댑터 메서드를 활용해서 해결 할 수 있다
```java
// Stream<E>를 Iterable<E>로 중개해주는 어댑터
public static <E> Iterable<E> iterableOf(Stream<E> stream) {
	return stream::iterator;
}

// 어댑터를 사용하여 for-each 문으로 반복
for(ProcessHandle p : iterableOf(ProcessHandle.allProcesses())) {
	// 프로세스 처리
}
```

```java
// Iterable<E> 를 Stream<E>로 중개해주는 어댑터
public static <E> Stream<E> streamOf(Iterable<E> utterable) {
	return StreamSupport.stream(utterable.spliterator(), false);
}
```

객체 시퀀스를 반환하는 메서드를  작성하는데, 스트림 파이프라인에서만 쓰인다면 스트림을 반환해야 하지만, 반복문에서만 쓰인다면 Iterable을 반환하자. (하지만 공개 API는 모두 고려해야 한다)

Collection 인터페이스는 Iterable의 하위 타입이고 stream 메서드도 제공하니 반복과 스트림을 동시에 제공한다. 
-> 원소 시퀀스를 반환하는 공개 API의 반환 타입에는 Collection이나 그 하위 타입을 쓰는게 일반적으로 최선이다. (하지만 단지 컬렉션을 반환한다는 이유로 덩치 큰 시퀀스를 메모리에 올려서는 안된다)

반환할 시퀀스가 크지만 표현을 간결하게 할 수 있다면 전용 컬렉션을 구현하는 방안을 검토해 보아야 한다. AbstractList를 이용하면 훌륭한 전용 컬렉션을 손쉽게 구현할 수 있다.

```java
// 입력 집합의 멱집합을 전용 컬렉션에 담아 반환
public class PowerSet {
	public static final <E> Collection<Set<E>> of(Set<E> s) {
		List<E> src = new ArrayList<>(s);
		if(src.size() > 30)
			throw new IllegalArgumentException(
				“집합에 원소가 너무 많습니다(최대 30개).:”+s);
		
		return new AbstractList<Set<E>>() {
			@Override public int size() {
			// 멱집합의 크기는 2를 원래 집합의 원소 수만큼 거듭제곱한것과 같다
				return 1<<src.size();
			}
			@Override public boolean contains(Object o) {
				return o instanceof Set && src.containsAll((Set)o);
			}
			@Override public Set<E> get(int index) {
				Set<E> result = new HashSet<>();
				for(int i=0 ; index != 0 ; i++, index >>= 1)
					if((index & 1) == 1)
						result.add(src.get(i));
				return resume;
			}
		};
	}
}
```

> 입력집합의 원소수가 30을 넘으면 예외를 던진다. 이는 Stream이나 Iterable 이 아닌 Collection 반환 타입으로 쓸 때의 단점이다

기하 급수적으로 늘어나는 멱집합 보다는 낫지만 여전히 좋은 방식은 아니다. 


::핵심정리:: 

> 원소 시퀀스를 반환하는 메서드를 작성할때 이를 스트림으로 처리하기를 원하는 사용자와 반복으로 처리하길 원하는 사용자 모두 있을 수 있음을 떠올리고, 양쪽을 다 만족시켜야 하낟. 컬렉션을 반환할 수 있다면 그렇게 하자. 반환 전부터 이미 원소들을 컬렉션에 담아 관리하고 있거나 컬렉션을 하나 더 만들어도 될 정도로 원소 개수가 적다면 ArrayList 같은 표준 컬렉션에 담아 반환하자. 그렇지 않으면 전용 컬렉션 구현을 고민하자. 컬렉션을 반환하는 게 불가능하면 스트림과 Iterable 중 더 자연스러운 것을 반환하자. 만약 나중에 Stream 인터페이스가 Iterable을 지원하도록 자바가 수정 된다면, 그때 스트림을 반환하면 될 것 이다




