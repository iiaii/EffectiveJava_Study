# #EffectiveJava/12장_직렬화/87커스텀직렬화


## 87. 커스텀 직렬화 형태를 고려해보라

개발 일정에 쫓기는 상황에서는 API 설계에 노력을 집중하는 편이 낫다. (다음 릴리스에서 제대로 다시 구현하기로 하고, 이번 릴리스에서 동작만 하도록 만들어 놓으라는 의미이다) 보통은 크게 문제되지 않는 전략이지만 클래스가 Serializable을 구현하고 기본 직렬화 형태를 사용한다면 다음 릴리스 때 버리려 한 현재의 구현에 영원히 발이 묶이게 된다. (기본 직렬화 형태를 버릴 수 없게 된다. 실제로 BigInteger 같은 일부 자바 클래스가 이 문제에 시달리고 있다)

**먼저 고민해보고 괜찮다고 판단될 때만 기본 직렬화 형태를 사용하라.** 기본 직렬화 형태는 유연성, 성능, 정확성 측면에서 신중히 고민한 후 합당할 때나 사용해야 한다. 일반적으로 직접 설계하더라도 기본 직렬화 형태와 거의 같은 결과가 나올 경우에만 기본 형태를 써야한다.  
 어떤 객체의 기본 직렬화 형태는 그 객체를 루트로 하는 객체 그래프의 물리적 모습을 나름 효율적으로 인코딩한다. 즉, 객체가 포함한 데이터들과 그 객체에서부터 시작해 접근할 수 있는 모든 객체를 담아내며, 심지어 이 객체들이 연결된 위상까지 기술한다. 

**객체의 물리적 표현과 논리적 내용이 같다면 기본 직렬화 형태라도 무방하다.**
```java
// 기본 직렬화 형태에 적합한 후보
public class Name implements Serializable {
	/**
	 * 성. null이 아니어야 함.
	 * @serial
	 */
	private final String firstName;

	/**
	 * 중간이름. 중간이름이 없다면 null.
	 * @serial
	 */
	private final String firstName;

	/**
	 * 중간이름. 중간이름이 없다면 null
	 * @serial
	 */
	private final String middleName;

	… // 나머지 코드는 생략
}
```

성명은 논리적으로 이름, 성, 중간이름이라는 3개의 문자열로 구성되며, 앞 코드의 인스턴스 필드들은 이 논리적 구성요소를 정확히 반영했다.
 **기본 직렬화 형태가 적합하다고 결정했더라도 불변식 보장과 보안을 위해 readObject 메서드를 제공해야 할 때가 많다.** 앞의 Name 클래스의 경우에는 readObject 메서드가 lastName과 firstName 필드가 null이 아님을 보장해야 한다.

> Name의 세필드 모두 private임에도 주석이 달려있다. 이 필드들은 결국 클래스의 직렬화 형태에 포함되는 공개 API에 속하며 공개 API는 모두 문서화해야 하기 때문이다. private 필드의 설명을 API 문서에 포함하라고 자바독에 알려주는 역할은 @serial 태그가 한다. @serial 태그로 기술한 내용은 API 문서에서 직렬화 형태를 설명하는 특별한 페이지에 기록된다.


```java
// 기본 직렬화 형태에 적합하지 않은 클래스
public final class StringList implements	Serializable {
	private int size = 0;
	private Entry head = null;

	private static class Entry implements Serializable {
		String data;
		Entry next;
		Entry previous;
	}
	… // 나머지 코드 생략
}
```

논리적으로 이 클래스는 일련의 문자열을 표현한다. 물리적으로는 문자열들을 이중 연결 리스트로 연결했다. 이 클래스에 기본 직렬화 형태를 사용하면 각 노드의 양방향 연결 정보를 포함해 모든 엔트리를 철두철미하게 기록한다.

객체의 물리적 표현과 논리적 표현의 차이가 클 때 기본 직렬화 형태를 사용하면 크게 네가지 면에서 문제가 생긴다.

1. **공개 API가 현재의 내부 표현 방식에 영구히 묶인다.** 앞의 예에서 private 클래스인 StringList.Entry 가 공개 API가 되어 버린다. 다음 릴리스에서 내부 표현 방식을 바꾸더라도 StringList 클래스는 여전히 연결 리스트로 표현된 입력도 처리할 수 있어야 한다. 즉, 연결 리스트를 더는 사용하지 않더라도 관련 코드를 절대 제거할 수 없다.
2. **너무 많은 공간을 차지할 수 있다.** 앞 예의 직렬화 형태는 연결 리스트의 모든 엔트리와 연결 정보까지 기록했지만, 엔트리와 연결 정보는 내부 구현에 해당하니 직렬화 형태에 포함할 가치가 없다. 이처럼 직렬화 형태가 너무 커져서 디스크에 저장하거나 네트워크로 전송하는 속도가 느려진다.
3. **시간이 너무 많이 걸릴 수 있다.** 직렬화 로직은 객체 그래프의 위상에 관한 정보가 없으니 그래프를 직접 순회해볼 수 밖에 없다. 앞의 예에서는 간단히 다음 참조를 따라 가보는 정도로 충분하다.
4. **스택 오버플로를 일으킬 수 있다.** 기본 직렬화 과정은 객체 그래프를 재귀 순회하는데, 이 작업은 중간 정도 크기의 객체 그래프에서도 자칫 스택 오버플로를 일으킬 수 있다. (플랫폼 마다 다를 수 있음)


StringList를 위한 합리적인 직렬화 형태는 StringList의 물리적인 상세 표현은 배제한 채 논리적인 구성만 담는 것이다. (writeObject와 readObject가 직렬화 형태를 처리하며 일시적이란 뜻의 transient 한정자는 해당 인스턴스 필드가 기본 직렬화 형태에 포함되지 않는다는 표시다)

```java
// 합리적인 커스텀 직렬화 형태를 갖춘 StringList
public final class StringList implements Serializable {
	private	transient int size = 0;
	private transient Entry head = null;

	// 이제는 직렬화되지 않는다.
	private static class Entry {
		String data;
		Entry next;
		Entry previous;
	}

	// 지정한 문자열을 이 리스트에 추가한다.
	public final void add(string s) { … }

	/**
	 * 이 {@code StringList} 인스턴스를 직렬화한다.
	 *
	 * @serialData 이 리스트의 크기(포함된 문자열의 개수)를 기록한 후
	 * ({@code int}), 이어서 모든 원소를(각각은 {@code String})
	 * 순서대로 기록한다.
 	 */
	private void writeObject(ObjectOutputStream s)
			throws IOException {
		s.defaultWriteObject();
		s.writeInt(size);

		// 모든 원소를 올바른 순서로 기록한다.
		for(Entry e = head ; e != null; e = e.next)
			s.writeObject(e.data);

}

	private void readObject(ObjectInputStream s)
			throws IOException, ClassNotFoundException {
		s.defaultReadObject();
		int numElements = s.readInt();

		// 모든 원소를 읽어 이 리스트에 삽입한다.
		for(int i=0 ; i<numElements ; i++)
			add((String) s.readObject());
	}
	… // 나머지 코드는 생략
}	
```

StringList의 필드 모두가 transient더라도 writeObject와 readObject는 각각 가장 먼저 defaultWriteObject와 defaultReadObject를 호출한다. 클래스의 인스턴스 필드 모두가 transient면 defaultWriteObject와 defaultReadObject를 호출하지 않아도 된다고 들었을지 모르지만, 직렬화 명세는 이 작업을 무조건 하라고 요구한다. 이렇게 해야 향후 릴리스에서 transient가 아닌 이스턴스 필드가 추가되더라도 상호(상위 하위 모두) 호환되기 때문이다. 신버전 인스턴스를 직렬화 한 후 구 버전으로 역직렬화하면 새로 추가된 필드들은 무시될 것이다. 구버전 readObject 메서드에서 defaultReadObject를 호출하지 않는다면 역직렬화할 때 StreamCorruptedException이 발생할 것이다. (절반정도의 공간, 두배정도 빠르며 스택 오버플로가 발생하지 않는다.)

> writeObject는 private 메서드임에도 문서화 주석이 달려 있다. 앞서 Name 클래스의 private 필드에 문서화 주석을 단 이유의 연장선이다. 이 private 메서드는 직렬화 형태에 포함되는 공객 API에 속하며, 공개 API는 모두 문서화해야 한다. 필드용의 @serial 태그처럼 메서드에 달린 @serialData 태그는 자바독 유틸리티에게 이 내용을 직렬화 형태 페이지에 추가하도록 요청하는 역할을 한다.


기본 직렬화를 수용하든 하지 않든 defaultWriteObject 메서드를 호출하면 transient로 선언하지 않은 모든 인스턴스 필드가 직렬화된다. 따라서 transient로 선언해도 되는 인스턴스 필드에는 모두 transient 한정자를 붙여야 한다. 캐시된 해시 값처럼 다른 필드에서 유도되는 필드도 여기 해당한다. JVM을 실행할 때마다 값이 달라지는 필드도 마찬가지인데, 네이티브 자료구조를 가리키는 long 필드가 여기 속한다. **해당 객체의 논리적 상태와 무관한 필드라고 확신할 때만 transient 한정자를 생략해야 한다.** 그래서 커스텀 직렬화 형태를 사용한다면, 앞서의 StringList 예에서처럼 대부분의 (혹은 모든) 인스턴스 필드를 transient로 선언해야 한다.

기본 직렬화를 사용한다면 transient 필드들은 역직렬화될 때 기본값으로 초기화됨을 잊지 말자. 객체 참조 필드는 null로, 숫자 기본 타입 필드는 0으로, boolean 필드는 false로 초기화된다. 기본값을 그대로 사용해서는 안된다면 readObject 메서드에서 defaultReadObject를 호출한 다음, 해당 필드를 원하는 값으로 복원하자(아이템88). 혹은 그 값을 처음 사용할 때 초기화하는 방법도 있다. (아이템83)

기본 직렬화 사용여부와 상관없이 객체의 전체 상태를 읽는 메서드에 적용해야 하는 동기화 메커니즘을 직렬화에도 적용해야 한다. 따라서 예컨대 모든 메서드를 synchronized로 선언하여 스레드 안전하게 만든 객체(아이템82)에서 기본 직렬화를 사용하려면 writeObject도 다음 코드처럼 synchronized로 선언해야 한다.

```java
// 기본 직렬화를 사용하는 동기화된 클래스를 위한 writeObject 메서드
private synchronized void writeObject(ObjectOutputStream s) 
		throws IOException {
	s.defaultWriteObject();
}
```

writeObject 메서드 안에서 동기화하고 싶다면 클래스의 다른 부분에서 사용하는 락 순서를 똑같이 따라야 한다. 그렇지 않으면 자원 순서 교착상태에 빠질 수 있다.
 **어떤 직렬화 형태를 택하든 직렬화 가능 클래스 모두에 직렬 버전 UID를 명시적으로 부여하자.** 이렇게 하면 직렬 버전 UID가 일으키는 잠재적인 호환성 문제가 사라진다.(아이템86) 성능도 조금 빨라지는데, 직렬 버전 UID를 명시하지 않으면 런타임에 이 값을 생성하느라 복잡한 연산을 수행하기 때문이다. 직렬 버전 UID 선언은 각 클래스에 아래 같은 한 줄만 추가해주면 끝이다.

`private static final long serialVersionUID = <무작위로 고른 long값>;`

새로 작성하는 클래스에서는 어떤 long값을 선택하든 상관없다. 클래스 일련번호를 생성해주는 serialver 유틸리티를 사용해도 되며, 그냥 생각나는 아무 값이나 선택해도 된다. 직렬 버전 UID가 꼭 고유할 필요는 없다. 한편 직렬 버전 UID가 없는 기존 클래스를 구버전으로 직렬화된 인스턴스와 호환성을 유지한 채 수정하고 싶다면, 구버전에서 사용한 자동 생성된 값을 그대로 사용해야 한다. 이 값은 직렬화된 인스턴스가 존재하는 구버전 클래스를 serialver 유틸리티에 입력으로 주어 실행하면 얻을 수 있다.

기존 버전 클래스와의 호환성을 끊고 싶다면 단순히 직렬 버전 UID의 값을 바꿔주면 된다. 이렇게 하면 기존 버전의 직렬화된 인스턴스를 역직렬화할 때 InvalidClassException 던져질 것이다. **구버전으로 직렬화된 인스턴스들과의 호환성을 끊으려는 경우를 제외하고는 직렬 버전 UID를 절대 수정하지 말자.**


**::핵심 정리::** 

> 클래스를 직렬화하기로 했다면(아이템86) 어떤 직렬화 형태를 사용할지 심사숙고해야 한다. 자바의 기본 직렬화 형태는 객체를 직렬화한 결과가 해당 객체의 논리적 표현에 부합할 때만 사용하고, 그렇지 않으면 객체를 적절히 설명하는 커스텀 직렬화 형태를 고안하라. 직렬화 형태도 공개 메서드(아이템51)를 설계할 때에 준하는 시간을 들여 설계해야 한다. 한번 공개된 메서드는 향후 릴리스에서 제거할 수 없듯이, 직렬화 형태에 포함된 필드도 마음대로 제거할 수 없다. 직렬화 호환성을 유지하기 위해 영원히 지원해야 하는 것이다. 잘못된 직렬화 형태를 선택하면 해당 클래스의 복잡성과 성능에 영구히 부정적인 영향을 남긴다. 





