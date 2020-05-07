# #EffectiveJava/5장_제네릭/33타입안전이종컨테이너

## 33. 타입 안전 이종 컨테이너를 고려하라

제네릭은 Set<E>, Map<K,V> 등의 컬렉션과 ThreadLocal<T>, AtomicReference<T> 등의 단일원소 컨테이너에도 흔히 쓰인다. 이런 모든 쓰임에서 매개변수화되는 대상은 (원소가 아닌) 컨테이너 자신이다. 따라서 하나의 컨테이너에서 매개변수화할 수 있는 타입의 수가 제한된다. (컨테이너의 일반적인 용도에 맞게 설계된 것으로 문제될 건 없다)

하지만 더 유연한 수단이 필요할 때도 종종 있다. 데이터베이스의 행은 임의 개수의 열을 가질 수 있는데, 모두 열을 타입 안전하게 이용할 수 있으면 좋을 텐데, 쉬운 해법이 있다. 컨테이너에 값을 넣거나 뺄 때 매개변수화한 키를 함께 제공하면 된다. 이러한 설계 방식을 타입 안전 이종 컨테이너 패턴이라 한다. (제네릭 타입 시스템이 값의 타입이 키와 같음을 보장해 줌)

(class 리터럴의 타입은 Class<T>. String.class의 타입은 Class<String>)

```java
// 타입 안전 이종 컨테이너 패턴 - API
public class Favorites {
	public <T> void putFavorite(Class<T> type, T instance);
	public <T> T getFavorite(Class<T> type);
}

// 타입 안전 이종 컨테이너 패턴 - 클라이언트
public static void main(String[] args) {
	Favorites f = new Favorites();

	f.putFavorite(String.class, “Java”);
	f.putFavorite(Integer.class, 0xcafebabe);
	f.putFavoirte(Class.class, Favorites.class);

	String favoriteString = f.getFavorite(String.class);
	int favoriteInteger = f.getFavoretie(Integer.class);
	Class<?> favoriteClass = f.getFavorite(Class.class);

	System.out.printf(“%s %x %s%n”, favoriteString, favoriteInteger, favoriteClass.getName());
}

// 출력
// Java cafebabe Favorites
```

Favorites 인스턴스는 타입 안전하다. String을 요청했는데 Integer를 반환하는 일은 절대 없다. 또한 모든 키의 타입이 제각각이라, 일반적인 맵과 달리 여러가지 타입의 원소를 담을 수 있다. (Favorites는 타입 안전 이종 컨테이너)

```java
public class Favorites {
	private Map<Class<?>, Object> favorites = new HashMap<>();

	public <T> void putFavorite(Class<T> type, T instance) {
		favorites.put(Objects.requireNonNull(type), instance);
	}

	public <T> T getFavorite(Class<T> type) {
		return type.cast(favorites.get(type));
	}
}
// 악의적인 클라이언트가 Class객체를 제네릭이 아닌 로타입으로 넘기면 타입안정성이 깨진다. -> ClassCastException 발생
// 동적 형변환으로 런타임 타입 안전성 확보
public <T> void putFavorite(Class<T> type, T instance) {
	favorites.put(Objects.requireNonNull(type), type.cast(instance);
}
// 실체화 불가타입에는 사용 할 수 없는데 완벽히 해결 불가 (대안-슈퍼토큰)

// asSubclass 를 사용해 한정적 타입 토큰을 안전하게 형변환
static Annotation getAnnotation(AnnotatedElement element, String annotationTypeName) {
	Class<?> annotationType = null; // 비한정적 타입 토큰
	try {
		annotationType = Class.forName(annotationTypeName);
	} catch(Exception ex) {
		throw new IllegalArgumentException(ex);
	}
	return element.getAnnotation(annotationType.asSubclass(Annotation.class));
}
```


::핵심 정리:: 

> 컬렉션 API로 대표되는 일반적인 제네릭 형태에서는 한 컨테이너가 다룰 수 있는 타입 매개변수의 수가 고정되어 있다. 하지만 컨테이너 자체가 아닌 키를 타입 매개변수로 바꾸면 이런 제약이 없는 타입 안전 이종 컨테이너를 만들 수 있다. 타입 안전 이종 컨테이너는 Class를 키로 쓰며, 이런 식으로 쓰이는 Class 객체를 타입 토큰이라한다. 또한 직접 구현한 키 타입도 쓸 수 있다. 예컨대 데이터베이스의 행(컨테이너)을 표현한 DatabaseRow 타입에는 제네릭 타입인 Column<T>를 키로 사용할 수 있다.







