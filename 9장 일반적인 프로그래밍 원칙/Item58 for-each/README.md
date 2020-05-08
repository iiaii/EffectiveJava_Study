# #EffectiveJava/9장_일반적인프로그래밍원칙/58for-each


## 58. 전통적인 for문 보다는 for-each문을 사용하라


기존 for 문은 원소만 필요한 경우에도 불필요한 i 변수가 등장할 일도 없고, 이로 인한 오류 가능성도 적어진다. (i 변수 남용 가능성 적어짐)

```java
for(Element e : elements) {
	… //
}

// 반복자가 포함된 for 문에서의 문제점이 모두 해결됨
// 컬렉션이나 배열의 중첩 반복을 위한 권장 관용구
for(Suit suit : suits)
	for(Rank rank : ranks)
		deck.add(new Card(suit, rank));
```


하지만 for-each를 사용할 수 없는 경우가 있다

1. **파괴적인 필터링**

컬렉션을 순회하면서 선택된 원소를 제거해야 한다면 반복자의 remove 메서드를 호출해야 한다. 자바 8부터 Collection의 removeIf 메서드를 사용해 컬렉션을 명시적으로 순회하는 일을 피할 수 있다.

```java
List<Integer> list = new ArrayList<Integer>();
// … list 에 값을 채움
list.removeIf( i -> i == 0 ); // list 안의 값이 0이면 제거
```

2. **변형**

리스트나 배열을 순회하면서 그 원소의 값 일부 혹은 전체를 교체해야 한다면 리스트의 반복자나 배열의 인덱스를 사용해야 한다.

3. **병렬 반복**

여러 컬렉션을 병렬로 순회해야 한다면 각각의 반복자와 인덱스 변수를 사용해 엄격하고 명시적으로 제어해야 한다. 


for-each 문은 컬렉션과 배열, Iterable 인터페이스를 구현한 객체라면 무엇이든 순회 가능하다. Iterable 인터페이스는 다음과 같이 메서드가 하나 뿐이다.

```java
public interface Iterable<E> {
	// 이 객체의 원소들을 순회하는 반복자를 반환한다.
	Iterator<E> iterator();
}
```


**::핵심 정리::** 

> for-each 문은 명료하고, 유연하고, 버그를 예방해준다. 성능 저하도 없다. 꼭 필요한 경우가 아니라면 for-each문을 사용하자

