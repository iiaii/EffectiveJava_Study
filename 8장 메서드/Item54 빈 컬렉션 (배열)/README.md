# #EffectiveJava/8장_메서드/54빈컬렉션(배열)


## 54. null이 아닌, 빈 컬렉션이나 배열을 반환하라


```java
// 컬렉션이 비었으면 null을 반환한다. - 따라하지 말 것!
private final List<Cheese> cheeseInStock = …;

/**
 * @return 매장 안의 모든 치즈 목록을 반환
 * 	단, 재고가 하나도 없다면 null을 반환
 */
public List<Cheese> getCheeses() {
	return cheesesInStock.isEmpty() ? null
		: new ArrayList<>(cheeseInStock);
}
```

재고가 없다고 해서 특별히 취급할 이유는 없지만 null을 반환하면 null을 처리하는 코드를 추가로 작성해야 한다.

```java
List<Cheese> cheeses = shop.getCheeses();
if(cheeses != null && cheeses.contains(Cheese.STILTON))
	System.out.println(“바로 요거!”);
```

이와 같이 방어 코드를 넣어줘야 한다. 클라이언트에서 null 체크를 하는 방어 코드를 넣어줘야 한다. 빈 컨테이너를 할당하는 데도 비용이 드니 null을 반환하는 것이 낫지 않냐고 하지만 두가지 측면에서 틀린 주장이다.

1. 성능 분석 결과 이 할당이 성능 저하의 주범이라고 확인되지 않는 한(아이템67), 이 정도 성능 차이는 신경 쓸 수준이 아니다.

2. 빈 컬렉션과 배열은 굳이 새로 할당하지 않고도 반환할 수 있다.

```java
// 빈 컬렉션을 반환하는 올바른 예
public List<Cheese> getCheeses() {
	return new ArrayList<>(cheesesInStock);
```



가능성은 작지만 사용 패턴에 따라 빈 컬렉션 할당이 성능을 눈에 띄게 떨어 뜨릴 수 도 있다. 해법은 매번 똑같은 빈 ‘불변' 컬렉션을 반환하는 것이다. 알다시피 불변 객체는 자요롭게 공유해도 안전하다. 다음 코드에서 사용하는 Collections.emptyList 메서드가 그러한 예다. 집합이 필요하면 Collections.emptySet을, 맵이 필요하면 Collections.emptyMap을 사용하면 된다. (최적화가 필요한 경우에 사용)

```java
// 최적화 - 빈 컬렉션을 매번 새로 할당하지 않도록 했다.
public List<Cheese> getCheeses() {
	return cheesesInStock.isEmpty() ? Collections.emptyList()
		: new ArrayList<>(cheesesInStock);
}
```

배열을 쓸 때 역시 null이 아닌 길이가 0인 배열을 반환하라. 
```java
// 길이가 0일 수도 있는 배열을 반환하는 올바른 방법
public Cheese[] getCheeses() {
	return cheesesInStock.toArray(new Cheese[0]);
}

// 최적화 - 빈 배열을 매번 새로 할당하지 않도록 했다
private static final Cheese[] EMPTY_CHEESE_ARRAY = new Cheese[0];

public Cheese[] getCheeses() {
	return cheesesInStock.toArray(EMPTY_CHEESE_ARRAY);
}

// <T> T[] List.toArray(T[] a) 메서드는 주어진 배열 a가 충분히 크면 a안에 원소를 담아 반환하고, 그렇지 않으면 T[] 타입 배열을 새로 만들어 그 안에 원소를 담아 반환한다. 
```

최적화 버전의 getCheeses는 항상 EMPTY_CHEESE_ARRAY를 인수로 넘겨 toArray를 호출하기 때문에 cheesesInStock이 비었을 때면 EMPTY_CHEESE_ARRAY를 반환하게 된다. (단순히 성능 개선 목적이라면 배열을 미리 할당하지 않는 것이 좋지 않다고 함)

```java
// 나쁜 예 - 배열을 미리 할당하면 성능이 나빠진다
return cheesesInStock.toArray(new Cheese[cheesesInStock.size()]);
```

> **::핵심 정리::** 
> null이 아닌, 빈 배열이나 컬렉션을 반환하라. null을 반환하는 API는 사용하기 어렵고 오류 처리 코드도 늘어난다. 그렇다고 성능이 좋은 것도 아님

