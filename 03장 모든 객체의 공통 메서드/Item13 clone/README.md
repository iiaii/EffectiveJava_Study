# #EffectiveJava/3장_모든객체의공통메서드/13clone

## 13. clone 재정의는 주의해서 진행하라

Cloneable은 복제해도 되는 클래스임을 명시하는 용도의 믹스인 인터페이스지만 아쉽게도 의도한 목적을 제대로 이루지 못했다. 가장 큰 문제는 clone 메서드가 선언된 곳이 Cloneable이 아닌 Object이고, 그마저도 protected 라는데 있다. 그래서 Cloneable을 구현하는 것 만으로는 외부 객체에서 clone 메서드를 호출할 수 없다. 리플렉션을 사용하면 가능하지만 100% 성공하는 것도 아니다. 해당 객체가 접근이 허용된 clone 메서드를 제공한다는 보장이 없기 때문이다. 하지만 이를 포함한 여러 문제점에도 불구하고 Cloneable 방식은 널리 쓰이고 있어서 잘 알아두는 것이 좋다. (이번 아이템에서는 clone 메서드를 잘 동작하게끔 해주는 구현 방법과 언제 그렇게 해야 하는지를 알려주고, 가능한 다른 선택지에 관해 논의하겠다)

Cloneable 인터페이스는 Object의 protected 메서드인 clone의 동작 방식을 결정한다. Cloneable을 구현한 클래스의 인스턴스에서 clone을 호출하면 그 객체의 필드들을 복사한 객체를 반환하며, 그렇지 않은 클래스의 인스턴스에서 호출하면 CloneNotSupportedException을 던진다. (이는 인터페이스를 이례적으로 사용한 예이니 따라 하지는 말자) ::인터페이스를 구현한다는 것은 일반적으로 해당 클래스가 그 인터페이스에서 정의한 기능을 제공한다고 선언하는 행위다.:: 그런데 Cloneable의 경우 상위 클래스에 정의된 protected 메서드의 동작방식을 변경한 것이다.


실무에서 Cloneable을 구현한 클래스는 clone 메서드를 public으로 제공하며, 사용자는 당연히 복제가 제대로 이뤄지리라 기대한다. 

이 기대를 만족시키려면 그 클래스와 모든 상위 클래스는 복잡하고, 강제할 수 없고, 허술하게 기술되 프로토콜을 지켜야만 하는데, 그 결과로 깨지기 쉽고, 위험하고, 모순적인 메커니즘이 탄생한다. **생성자를 호출하지 않고도 객체를 생성할 수 있게 되는 것이다.**

clone 메서드의 일반 규약은 허술하다.

> 이 객체의 복사본을 생성해 반환한다. 복사의 정확한 뜻은 그 객체를 구현한 클래스에 따라 다를 수 있다. 일반적인 의도는 다음과 같다. 어떤 객체 x에 대해 다음 식은 참이다.
>  
> x.clone() != x
> 또한 다음 식도 참이다
> x.clone().getClass() == x.getClass()
>  
> 하지만 이상의 요구를 반드시 만족해야 하는 것은 아니다. 한편 다음식도 일반적으로 참이지만, 역시 필수는 아니다.
>  
> x.clone().equals(x)
>  
> 관례상, 이 메서드가 반환하는 객체는 super.clone을 호출해 얻어야 한다. 이 클래스와 (Object를 제외한) 모든 상위 클래스가 이 관례를 따른다면 다음 식은 참이다.
>  
> x.clone().getClass() == x.getClass()
>  
> 관례상, 반환된 객체와 원본 객체는 독립적이어야 한다. 이를 만족하려면 super.clone으로 얻은 객체의 필드 중 하나 이상을 반환 전에 수정해야 할 수도 있다.

강제성이 없다는 점만 빼면 생성자 연쇄와 살짝 비슷한 메커니즘이다. 즉, clone 메서드가  super.clone이 아닌, 생성자를 호출해 얻은 인스턴스를 반환해도 컴파일러는 상관안한다. 하지만 이 클래스의 하위 클래스에서 super.clone을 호출하면 잘못된 클래스의 객체가 만들어져, 결국 하위 클래스의 clone 메서드가 제대로 동작하지 않게 된다. 
-> B클래스가 A클래스를 상속할 때, 하위 클래스인 B의 clone은 B 타입 객체를 반환해야 한다. 그런데 A의 clone이 자신의 생성자, 즉 new A(…)로 생성한 객체를 반환한다면 B의 clone도 A 타입 객체를 반환할 수 밖에 없다. 달리 말해 super.clone을 연쇄적으로 호출하도록 구현해 두면 clone이 처음 호출된 하위 클래스의 객체가 만들어진다.

clone을 재정의한 클래스가 final이라면 무시해도 된다. 하지만 final 클래스의 clone 메서드가 super.clone을 호출하지 않으면 Cloneable을 구현할 이유도 없다. Object의 clone 구현의 동작 방식에 기댈 필요가 없기 때문이다. 


```java
// 가변 상태를 참조하지 않는 클래스용 clone 메서드
@Override
public PhoneNumber clone() {
	try {
		return (PhoneNumber) super.clone();
	} catch(CloneNotSupportedException e) {
		throw new AssertionError(); // 일어날 수 없는 일이다
	}
}
```

자바가 공변 반환 타이핑을 지원해서 가능하고 권장하는 방식이다. 재정의한 메서드의 반환 타입은 상위 클래스 메서드가 반환하는 타입의 하위 타입일 수 있다. 이 방식으로 클라이언트가 형 변환하지 않아도 되게끔 clone메서드를 재정의 하였다.


이전에 구현했던 Stack 클래스를 복제하도록 할때 clone 메서드가 단순히 super.clone을 반환하면 size와 같은 필드는 올바르지만 elements필드와 같은 배열을 원본 stack 인스턴스와 똑같은 배열을 참조한다. (원본이나 복제본 중 하나를 수정하면 다른 하나도 같이 수정되어 불변식을 해친다)

Stack 클래스의 하나뿐인 생성자를 호출한다면 이러한 상황은 일어나지 않는다. **clone 메서드는 사실상 생성자와 같은 효과를 낸다. 즉, clone은 원본 객체에 아무런 해를 끼치지 않는 동시에 복제된 객체의 불변식을 보장해야 한다.** 그래서 Stack의 clone 메서드는 제대로 동작하려면 스택 내부 정보를 복사해야 하는데 가장 쉬운 방법은 elements 배열의 clone을 재귀적으로 호출해 주는 것이다

```java
// 가변 상태를 참조하는 클래스용 clone 메서드
@Override
public Stack clone() {
	try {
		Stack result = (Stack) super.clone();
		result.elements = elements.clone();
		return result;
	} catch(CloneNotSupportedException e) {
		throw new AssertionError();
	}
}
```

배열을 복제할 대는 배열의 clone메서드를 사용하라고 권장한다. (배열은 clone 기능을 제대로 사용하는 유일한 예)

한편 elements 필드가 final이 었다면 앞서 방식은 작동하지 않는다. final 필드에는 새로운 값을 할당할 수 없기 때문이다. 이는 근본적인 문제로, 직렬화와 마찬가지로 **Cloneable 아키텍처는 ‘가변 객체를 참조하는 필드는 final로 선언하라’라는 일반 용법과 충돌한다.**(단, 원본과 복제된 객체가 그 가변 객체를 공유해도 안전하면 괜찮다. 복제할 수 있는 클래스를 만들기 위해 일부 필드에서 final 한정자를 제거해야 할 수도 있다) 

clone을 재귀적으로 호출하는 것만으로는 충분하지 않을 때도 있다. 해시테이블 내부는 버킷들의 배열이고, 각 버킷은 키-값 쌍을 담는 연결 리스트의 첫번째 엔트리를 참조한다. 방금 처럼 버킷을 그냥 복사해 버리면 같은 연결리스트를 참조하여 문제가 생길 수 있다. 이를 해결하면..

```java
public class HashTable implements Cloneable {
	private Entry[] buckets = …;

	private static class Entry {
		final Object key;
		Object value;
		Entry next;

		Entry(Object key, Object value, Entry next) {
			this.key = key;
			this.value = value;
			this.next = next;
		}

		// 이 엔트리가 가리키는 연결 리스트를 재귀적으로 복사
		Entry deepCopy() {
			return new Entry(key, value, next==null ? null : next.deepCopy());
		}
	}

	@Override
	public HashTable clone() {
		try {
			HashTable result = (HashTable) super.clone();
			result.buckets = new Entry[buckets.length];
			for(int i=0 ; i<buckets.length ; i++) 
				if(buckets[i] != null)
					result.buckets[i] = buckets[i].deepCopy();
			return result;
		} catch(CloneNotSupportedException e) {
			throw new AssertionError();
		}
	}
	// 나머지 생략
}
```

private 클래스인 HashTable.Entry는 깊은복사를 지원하도록 보강되었다. clone 메서드는 버킷 배열을 순회하면 비지 않은 각 버킷에 대해 깊은 복사를 수행한다. 이때 Entry의 deepCopy 메서드는 자신이 가리키는 연결 리스트 전체를 복사하기 위해 재귀호출을 한다. (버킷이 길지 않으면 잘 작동함. 길면 오버플로우 발생 위험)

```java
// 재귀호출 -> 반복자
Entry deepCopy() {
	Entry result = new Entry(key, value, next);
	for(Entry p=result ; p.next!=null ; p=p.next)
		p.next = new Entry(p.next.key, p.next.value, p.next.next);
	return result;
}
```


##### 복잡한 가변 객체를 복제하는 방법

먼저 super.clone을 호출하여 얻은 객체의 모든 필드를 초기 상태로 설정한 다음, 원본 객체의 상태를 다시 생성하는 고수준 메서드들을 호출한다. (Cloneable 아키텍처의 기초가 되는 필드 단위 객체 복사를 우회하기 때문에 전체 Cloneable 아키텍처와는 어울리지 않고 조금 느리다)

생성자에서는 재정의 될 수 있는 메서드를 호출하지 않아야 하는데 clone 메서드도 마찬가지다. 만약 clone이 하위 클래스에서 재정의한 메서드를 호출하면, 하위 클래스는 복제 과정에서 자신의 상태를 교정할 기회를 잃게 되어 원복과 복제본이 달라질 가능성이 크다.
따라서 put(key, value) 메서드는 final이거나 private이어야 한다 (private이라면 final이 아닌 public메서드가 사용하는 도우미 메서드일 것임)

Object 의 clone 메서드는 CloneNotSupportedException을 던진다고 선언했지만 재정의한 메서드는 그렇지 않다. public clone 메서드에서는 throws 절을 없애야 한다. (검사 예외를 던지지 않아야 그 메서드를 사용하기 편하기 때문)

상속해서 쓰기 위한 클래스 설계방식 2가지중 어느 쪽에서든 상속용 클래스는 Cloneable을 구현해서는 안된다. (Object 방식을 모방해서 이미 제대로 작동하는 clone 메서드를 구현해 protected로 두고 CloneNotSupportedException도 던질수 있다고 선언할 수도 있다.(super.clone을 부른다면 저 예외는 나올 수가 없음) 이 방식은 Object를 바로 상속할 때처럼 Cloneable 구현 여부를 하위 클래스에서 선택하도록 해준다. clone을 동작하지 않게 구현해놓고 하위 클래스에서 재정의하지 못하게 할 수도 있다. 
```java
@Override
protected final Object clone() throws CloneNotSupportedException {
	throw new CloneNotSupportedException();
}
```


Cloneable을 구현한 스레드 안전 클래스를 작성할 때는 적절히 동기화 해야 한다. Object clone메서드는 동기화를 신경 쓰지 않았기 때문에 super.clone 호출 외에 다른 할일이 없더라도 clone을 재정의하고 동기화해야 한다.

요약하자면, Cloneable을 구현하는 모든 클래스는 clone을 재정의 해야한다. 이때 접근 제한자는 public으로, 반환 타입은 클래스 자신으로 변경한다. 이 메서드는 가장 먼저 super.clone을 호출한 후 필요한 필드를 전부 적절히 수정한다. (기본 타입필드와 불변 객체 참조만 갖는 클래스는 아무 필드도 수정할 필요없다. 고유번호는 제외)


Cloneable을 구현하지 않은 상황에서는 **복사 생성자와 복사 팩토리**라는 대안이 있다. 복사생성자는 단순히 자신과 같은 클래스 인스턴스를 인수로 받는 생성자를 말한다.

```java
// 복사 생성자
public Yum(Yum yum) { … };

// 복사 팩토리 (복사 생성자를 모방한 정적 팩토리)
public static Yum newInstance(Yum yum) { … };
```
Cloneable/clone 보다 나은 면이 많다.
생성자를 쓰지 않는 객체 생성을 하지 않고, 문서화된 규약에 기대지 않고, 정상적인 final 필드 용법과 충돌하지 않고, 불필요한 검사 예외를 던지지 않고 형변환도 하지 않는다. 또한 해당 클래스가 구현한 인터페이스 타입의 인스턴스를 인수로 받을 수 있다. 예컨대 관례상 모든 범용 컬렉션 구현체는 Collection이나 Map 타입을 받는 생성자를 제공한다. 인터페이스 기반 복사 생성자와 복사 팩토리의 더 정확한 이름은 ‘변환 생성자’와 ‘변환 팩토리’이다. (HashSet 객체 s를 TreeSet 타입으로 복제할 수있다. clone으로는 불가능한 이 기능을 변환 생성자로는 간단히 new TreeSet<>(s)로 처리한다)


::핵심 정리:: 

> Cloneable의 문제를 봤을때 새로운 인터페이스를 만들때는 절대 Cloneable을 확장해서는 안되며, 새로운 클래스도 이를 구현해서는 안된다. final 클래스라면 Cloneable을 구현해도 위험이 크지 않지만, 성능 최적화 관점에서 검토한 후 드물게 허용해야한다. 기본 원칙은 복제 기능은 생성자와 팩토리를 이용하는 것이 최고라는 것이다. 단 배열만은 clone 메서드 방식이 가장 깔끔한, 이 규칙의 합당한 예외이다.






