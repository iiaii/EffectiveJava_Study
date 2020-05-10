# #EffectiveJava/12장_직렬화/88readObject


## 88. readObject 메서드는 방어적으로 작성하라


아이템 50에서는 불변인 날짜 범위 클래스를 만드는데 가변인 Date 필드를 이용했다. 그래서 불변식을 지키고 불변을 유지하기 위해 생성자와 접근자에서 Date 객체를 방어적으로 복사하느라 코드가 상당히 길어졌다. (다음이 그 클래스)

```java
// 방어적 복사를 사용하는 불변 클래스
public final class Period {
	private final Date start;
	private final Date end;

	/**
	 * @param start 시작 시각
	 * @param end 종료 시각; 시작 시각보다 뒤여야 한다.
	 * @throws IllegalArgumentException 시작이 종료 보다 늦을때 발생
	 * @throws NullPointerException start나 end가 null이면 발행
	 */
	public Period(Date start, Date end) {
		this.start = new Date(start.getTime());
		this.end = new Date(end.getTime());
		if(this.start.compareTo(this.end) > 0)
			throw new IllegalArgumentException(
				start+”가 “+end+”보다 늦다.”);
	}

	public Date start() { return new Date(start.getTime()); }
	public Date end() { return new Date(end.getTime()); }
	public String toString() { return start+” - “+end; }

	… // 나머지 코드 생략
}
```

이 클래스를 직렬화하기로 결정했을 때, Period 객체의 물리적 표현이 논리적 표현과 부합하므로 기본 직렬화 형태(그냥 implements Serializable)를 사용해도 나쁘지 않다. 하지만 이 클래스의 주요한 불변식을 더는 보장하지 못한다. 

문제는 readObject 메서드가 실직적으로 또 다른 public 생성자이기 때문이다. 따라서 다른 생성자와 같은 수준으로 주의를 기울여야 한다. 보통의 생성자처럼 readObject 메서드에서도 인수가 유효한지 검사해야 하고(아이템49) 필요하다면 매개변수를 방어적으로 복사해야 한다(아이템50). readObject가 이 작업을 제대로 수행하지 못하면 공격자는 손쉽게 해당 클래스의 불변식을 깨뜨릴 수 있다.

쉽게 말해, readObject는 매개변수로 바이트 스트림을 받는 생성자라 할 수 있다. 보통의 경우 바이트 스트림은 정상적으로 생성된 인스턴스를 직렬화해 만들어진다. 하지만 불변식을 깨뜨릴 의도로 임의 생성한 바이트 스트림을 건네면 문제가 생긴다. 정상적인 생성자로는 만들어낼 수 없는 객체를 생성해낼 수 있기 때문이다.

단순히 Period 클래스 선언에 implements Serializable만 추가했다고 해보자. 그러면 다음의 프로그램을 수행해 종료시각이 시작보다 앞서는 Period인스턴스를 만들 수 있다.

```java
// 허용되지 않는 Period 인스턴스를 생성할 수 있다.
public class BogusPeriod {
	// 진짜 Period 인스턴스에서는 만들어질 수 없는 바이트 스트림
	private static final byte[] serializedForm = {
		…
	}

	public static void main(String[] args) {
		Period p = (Period) deserialize(serializedForm);
		System.out.println(p);
	}

	// 주어진 직렬화 형태(바이트 스트림)로부터 객체를 만들어 반환한다.
	static Object deserialize(byte[] sf) {
		try {
			return new ObjectInputStrea(
				new ByteArrayInputStream(sf)).readObject();
		} catch(IOException | ClassNotFoundException e) {
			throw new IllegalArgumentException(e);
		}
	}
}
```

> 이 코드의 serializedForm에서 상위 비트가 1인 바이트 값들은 byte로 형변환했는데, 이는 자바가 바이트 리터럴을 지원하지 않고 byte 타입은 부호 있는(singed) 타입이기 대문이다.

-> **Period(객체)를 직렬화할 수 있도록 선언한 것만으로 클래스의 불변식을 깨뜨리는 객체를 만들 수 있다**

이 문제를 고치려면 Period의 readObject 메서드가 defaultReadObject를 호출한 다음 역직렬화된 객체가 유효한지 검사해야 한다. 이 유효성 검사에 실패하면 InvalidObjectException을 던지게 하여 잘못된 역직렬화가 일어나는 것을 막을 수 있다.

```java
// 유효성 검사를 수행하는 readObject 메서드 - 아직 부족하다!
private void readObject(ObjectInputStream s)
			throws IOException, ClassNotFoundException {
	s.defaultReadObject();

	// 불변식을 만족하는지 검사한다.
	if(start.compareTo(end) > 0)
		throw new InvalidObjectException(start+”가 “+end+”보다 늦다.”);
}
```

정상 Period 인스턴스에서 시작된 바이트 스트림 끝에 private Date 필드로의 참조를 추가하면 가변 Period 인스턴스를 만들어낼 수 있다. 공격자는 ObjectInputStream에서 Period 인스턴스를 읽은 후 스트림 끝에 추가된 이 ‘악의적인 객체 참조’를 읽어 Period 객체의 내부 정보를 얻을 수 있다. 이제 이 참조로 얻은 Date 인스턴스들을 수정할 수 있으니, Period 인스턴스는 더는 불변이 아니게 되는 것이다. 

```java
// 가변 공격의 예
public class MultablePeriod {
	//	Period 인스턴스
	public final Period period;

	// 시작 시각 필드 - 외부에서 접근할 수 없어야 한다.
	public final Date start;

	// 종료 시각 필드 - 외부에서 접근할 수 없어야 한다.
	public final Date end;

	public MutablePeriod() {
		try {
			ByteArrayOutputStream bos =
				new ByteArrayOutputStream();
			ObjectOutputStream out =
				new ObjectOutputStream(bos);

			// 유효한 Period 인스턴스를 직렬화
			out.writeObject(new Period(new Date(), new Date()));

			/*
			 * 악의적인 ‘이전객체참조’, 즉 내부 Date 필드로의 참조를 추가
			 * 상세 내용은 자바 객체 직렬화 명세의 6.4절을 참고
			 */
			byte[] ref = { 0x71, 0, 0x72, 0, 5 }; // 참조 #5
			bos.write(ref); // start 필드
			ref[4] = 4; // 참조 #4
			bos.write(red); // end 필드

			// Period 역직렬화 후 Date 참조를 ‘훔친다’
			ObjectInputStream in = new ObjectInputStrea(
				new ByteArrayInputStream(bos.toByteArray()));
			period = (Period) in.readObject();
			start = (Date) in.readObject();
			end = (Date) in.readObject();
		} catch(IOException | ClassNotFoundException e) {
			throw new AssertionError(e);
		}
	}
}


// 다음 코드를 실행하면 이 공격이 실제로 이뤄지는 모습을 확인할 수 있다.
public static void main(String[] args) {
	MutablePeriod mp = new MutablePeriod();
	Period p = mp.period;
	Date pEnd = mp.end;

	// 시간을 되돌리자!
	pEnd.setYear(78);
	System.out.println(p);

	// 60년대로 회귀!
	pEnd.setYear(69);
	System.out.println(p);
}

// 결과
… 1978
… 1969
```

Period 인스턴스는 불변식을 유지한 채 생성됐지만, 의도적으로 내부의 값을 수정할 수 있었다. 이처럼 변경 가능한 Period 인스턴스를 획득한 공격자는 이 인스턴스가 불변이라고 가정하는 클래스에 넘겨 엄청난 보안 문제를 일으킬 수 있다. (실제로도 보안 문제를 String이 불변이라는 사실에 기댄 클래스들이 존재함)

문제의 근원은 Period readObject 메서드가 방어적 복사를 충분히 하지 않은데 있다. **객체를 역직렬화할 때는 클라이언트가 소유해서는 안되는 객체 참조를 갖는 필드를 모두 반드시 방어적으로 복사해야 한다.** 따라서 readObject에서는 불변 클래스 안의 모든 private 가변 요소를 방어적으로 복사해야 한다. 다음의 readObject 메서드라면 Period의 불변식과 불변 성질을 지켜내기에 충분하다.

```java
// 방어적 복사와 유효성 검사를 수행하는 readObject 메서드
private void readObject(ObjectInputStream s)
			throws IOException, ClassNotFoundException {
	s.defaultReadObject();

	// 가변 요소들을 방어적으로 복사한다.
	start = new Date(start.getTime());
	end = new Date(end.getTime());

	// 불변식을 만족하는지 검사한다.
	if(start.compareTo(end) > 0)
		throw new InvalidObjectException(start+”가 “+end+”보다 늦다.”);
}
```

방어적 복사를 유효성 검사보다 앞서 수행하며, Date의 clone 메서드는 사용하지 않았음에 주목하자. 두 조치 모두 Period를 공격으로부터 보호하는데 필요하다(아이템50). 또한 final 필드는 방어적 복사가 불가능하니 주의하자. 그래서 이 readObject 메서드를 사용하려면 start와 end 필드에서 final 한정자를 제거해야 한다. (앞의 공격에 노출되는 것 보다 낫다)


기본 readObject 메서드를 써도 좋을지 판단하는 방법은 transient 필드를 제외한 모든 필드의 값을 매개변수로 받아 유효성 검사 없이 필드에 대입하는 public 생성자를 추가해도 괜찮은가? 답이 ‘아니오’라면 커스텀 readObject 메서드를 만들어 (생성자에서 수행했어야 할) 모든 유효성 검사와 방어적 복사를 수행해야 한다. (혹은 직렬화 프록시 패턴(아이템90)을 사용한다 - 권장)

final이 아닌 직렬화 가능 클래스라면 readObject와 생성자의 공통점이 있는데, 마치 생성자처럼 readObject 메서드도 재정의 가능 메서드를 (직/간접적으로든) 호출해서는 안된다(아이템19). 이 규칙을 어겻는데 해당 메서드가 재정의되면, 하위 클래스의 상태가 완전히 역직렬화되기 전에 하위 클래스에서 재정의된 메서드가 실행된다. 결국 프로그램도 오작동으로 이어진다.


**::핵심 정리::** 

> readObject 메서드를 작성할 때는 언제나 public 생성자를 작성하는 자세로 임해야 한다. readObject는 어떤 바이트 스트림이 넘어오더라도 유효한 인스턴스를 만들어내야 한다. 바이트 스트림이 진짜 직렬화된 인스턴스라고 가정해서는 안된다. 이번 아이템에서는 기본 직렬화 형태를 사용한 클래스를 예로 들었지만 커스텀 직렬화를 사용하더라도 모든 문제가 발생할 수 있다. 
> 안전한 readObject 메서드 작성하는 지침
> - private이어야 하는 객체 참조 필드는 각 필드가 가리키는 객체를 방어적으로 복사하라. (불변 클래스 내의 가변 요소가 여기속함)
> - 모든 불변식을 검사하여 어긋나는게 발견되면 InvalidObjectException을 던진다. (방어적 복사 다음에는 반드시 불변식 검사가 뒤따라야 한다)
> - 역직렬화 후 객체 그래프 전체의 유효성을 검사해야 한다면 ObjectInputValidation 인터페이스를 사용하라
> - 직접적이든 간접적이든, 재정의 가능한 메서드는 호출하지 말자



