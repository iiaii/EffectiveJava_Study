# #EffectiveJava/5장_제네릭/29제네릭타입

## 29. 이왕이면 제네릭 타입으로 만들라


JDK가 제공하는 제네릭 타입과 메서드를 사용하는 일은 쉽지 않지만, 제네릭 타입을 새로 만드는 일이 좀 더 어렵다. 

```java
// Object 기반 스택 - 제네릭이 절실한 강력 후보
public class Stack {
	private Object[] elements;
	private int size = 0;
	private static final int DEFAULT_INITIAL_CAPACITY = 16;

	public Stack() {
		elements = new Object[DEFAULT_INITIAL_CAPACITY];
	}

	public void push(Object e) {
		ensureCapacity();
		elements[size++] = e;
	}

	public Object pop() {
		if(size == 0)
			throw new EmptyStackException();
		Object result = elements[--size];
		elements[size] = null; // 다 쓴 참조 해제
		return result;
	}

	public boolean isEmpty() {
		return size == 0;
	}

	private void ensureCapacity() {
		if(elements.length == size)
			elements = Arrays.copyOf(elements, 2*size+1);
	}
}
```

이 클래스는 제네릭 타입이어야 한다. 제네릭으로 바꾸더라도 현재 버전을 사용하는 클라이언트에는 아무런 해가 없다. 오히려 지금 상태에서의 클라이언트는 스택에서 꺼낸 객체를 형변환해야 하는데, 이때 런타임 오류가 날 위험이 있다.

일반 클래스를 제네릭 클래스로 만드는 첫 단계는 클래스 선언에 타입 매개 변수를 추가하는 일이다. 

```java
// 제네릭 스택으로 가는 첫 단계 - 컴파일되지 않는다
public class Stack<E> {
	private E[] elements;
	private int size = 0;
	private static final int DEFAULT_INITIAL_CAPACITY = 16;

	public Stack() {
		elements = new E[DEFAULT_INITIAL_CAPACITY];
	}

	public void push(E e) {
		ensureCapacity();
		elements[size++] = e;
	}

	public E pop() {
		if(size == 0)
			throw new EmptyStackException();
		E result = elements[--size];
		elements[size] = null; // 다 쓴 참조 해제
		return result;
	}

	… // isEmpty와 ensureCapacity 메서드는 그대로
}

// 하나이상의 오류나 경고 발생
Stack.java:8: generic array creation
		elements = new E[DEFAULT_INITIAL_CAPACITY];
```

아이템 28의 설명 처럼 **E와 같은 실체화 불가 타입으로는 배열을 만들 수 없다.** 배열을 사용하는 코드를 제네릭으로 만들려 할 때는 이런 문제가 발생하는데 해결책은 2가지 이다.

첫 번째는 제네릭 배열 생성을 금지하는 제약을 대놓고 우회하는 방법이다. Object 배열을 생성한 다음 제네릭 배열로 형변환 한다. 컴파일러는 오류 대신 경고를 내보내며 일반적으로 타입 안전하지 않다.

```
Stack.java:8: warning: [unchecked] unchecked cast
found: Object[], required: E[]
		elements = (E[]) new Object[DEFAULT_INITIAL_CAPACITY];
```

컴파일러는 이 프로그램이 타입 안전한지 증명할 방법이 없지만, 비검사 형변환이 프로그램의 타입 안전성을 해치지 않음을 스스로 확인해야 한다. 문제의 배열 elements는 private 필드에 저장되고, 클라이언트로 반환되거나 다른 메서드에 전달되는 일이 전혀 없다. push 메서드를 통해 배열에 저장되는 원소의 타입은 항상 E다. 따라서 이 비검사 형변환은 확실히 안전하다
-> 비검사 형변환이 안전함을 직접 증명했으므로 범위를 최소로 좁혀 @SuppressWarnings 애너테이션으로 해당 경고를 숨긴다.(아이템27). 깔끔히 컴파일되고 명시적으로 형변환하지 않아도 ClassCastException 걱정 없다

```java
// 배열을 사용한 코드를 제네릭으로 만드는 방법 1

// 배열 elements는 push(E)로 넘어온 E 인스턴스만 담는다
// 따라서 타입 안전성을 보장하지만, 이 배열의 런타임 타입은 E[]가 아닌 Object[]다
@SuppressWarnings(“unchecked”)
public Stack() {
	elements = (E[]) new Object[DEFAULT_INITIAL_CAPACITY];
}
```


제네릭 배열 생성 오류를 해결하는 두 번째 방법은 elements 필드의 타입을 E[]에서 Object[]로 바꾸는 것이다. 이렇게 하면 다른 오류가 발생한다.

```
Stack.java:19: incompatible types
found: Object, required: E
		E result = elements[--size];

// 배열이 반환한 원소를 E로 형변환하면 오류 대신 경고가 뜬다
Stack.java:19: warning: [unchecked] unchecked cast
found: Object, required: E
	E result = (E) elements[—size];
```

E는 실체화 불가 타입이므로 컴파일러는 런타임에 이뤄지는 형변환이 안전한지 증명할 방법이 없다. (이번에도 직접 증명하고 경고를 숨긴다)

```java
// 배열을 사용한 코드를 제네릭으로 만드는 방법 2
// 비검사 경고를 적절히 숨긴다
public E pop() {
	if(size == 0)
		throw new EmptyStackException();

	//push에서 E 타입만 허용하므로 이 형변환은 안전하다
	@SuppressWarnings(“unchecked”) 
	E result = (E) elements[--size];

	elements[size] = null; // 다 쓴 참조 해제
	return result;
}
```

첫 번째 방법은 가독성이 더 좋고, 배열의 타입을 E[]로 선언하여 오직 E타입 인스턴스만 받음을 확실히 한다. (코드도 짧다) 그리고 형변환을 배열 생성시 한번만 해주면 되지만, 두번째 방식에서는 배열에서 원소를 읽을 때마다 해줘야 한다. 따라서 현업에서는 첫 번째를 더 선호한다.

하지만 E가 Object가 아닌한 배열의 런타임 타입이 컴파일 타임 타입과 달라 힙 오염을 일으킨다. (아이템32) 이게 걱정되면 두 번째 방법을 사용한다.

```java
// 제네릭 Stack을 사용하는 맛보기 프로그램
public static void main(String[] args) {
	Stack<String> stack = new Stack<>();
	for(String arg : args)
		stack.push(arg);
	while(!stack.isEmpty())
		System.out.println(stack.pop().toUpperCase());
}
```

아이템28과 상반되는 내용이기도 한데, 사실 제네릭 타입 안에서 리스트를 사용하는 게 항상 가능하고 꼭 좋은것이 아니다. 자바가 리스트를 기본 타입으로 제공하지 않으므로 ArrayList 같은 제네릭 타입도 결국은 기본 타입인 배열을 사용해 구현해야 한다. 또한 HashMap 같은 제네릭 타입은 성능을 높일 목적으로 배열을 사용하기도 한다.


제네릭 타입은 매개변수에 아무런 제약을 두지 않는다. (Stack<Object>, Stack<int[]>, Stack<List<String>>, Stack 등 어떤 참조 타입으로도 Stack을 만들 수 있다. 단 기본 타입은 사용할 수 없다. 이는 제네릭 타입 시스템의 근본적인 문제이지만 박싱된 기본타입을 사용해 우회할 수 있다)

타입 매개변수에 제약을 두는 제네릭 타입도 있다. 

`class DelayQueue<E extends Delayed> implements BlockingQueue<E>`

<E extends Delayed>는 Delayed의 하위 타입만 받는다는 뜻이다. (자신 포함)


> ::핵심 정리:: 
> 클라이언트에서 직접 형변환해야 하는 타입보다 제네릭 타입이 더 안전하고 쓰기 편하다. 새로운 타입을 설계할 때는 형변환 없이도 사용할 수 있도록 하고 그러려면 제네릭 타입으로 만들어야 하는 경우가 많다. 기존 타입 중 제네릭이었어야 하는게 있다면 제네릭 타입으로 변경하자. 기존 클라이언트에는 아무 영향을 주지 않으면서, 새로운 사용자를 훨씬 편하게 해준다.


