# #EffectiveJava/8장_메서드/56문서화

## 56. 공개된 API 요소에는 항상 문서화 주석을 작성하라


API를 쓸모 있게 하려면 잘 작성된 문서도 곁들여야 한다. 전통적으로 API 문서는 사람이 직접 작성하므로 코드가 변경되면 매번 함께 수정해줘야 하는데, 자바독이 이를 도와준다.


문서화 주석을 작성하는 규칙은 공식 언어 명세에 속하지 않지만 알아야 한다. 

API를 올바로 문서화하려면 공개된 모든 클래스, 인터페이스, 메서드, 필드 선언에 문서화 주석을 달아야 한다. (직렬화 가능 클래스면 직렬화 형태(아이템87)에 대해서도 적어야함)
참고로 기본 생성자에는 문서화 주석을 달 방법이 없으므로 공개 클래스는 절대 기본 생성자를 사용하면 안된다. 또한 공개되지 않은 클래스, 인터페이스 등에도 문서화 주석을 달아야 한다.

메서드용 문서화 주석에는 해당 메서드와 클라이언트 사이의 규약을 명료하게 기술해야 한다. (그 메서드가 어떻게 동작하는지가 아니라 무엇을 하는지 (how가 아닌 what))
그리고 해당 메서드를 호출하기 위한 전제조건을 모두 나열해야 한다. (일반적으로 전제 조건은 @throws 태그로 비검사 예외를 선언하여 암시적으로 기술한다)
@param / @return / @throws 태그 작성

```java
// 예시

/**
 * Returns the element at the specified position in this list.
 *
 * <p>This method is <i>not</i> guaranteed to run in constant
 * time. In some implementations it may run in time proportional to the element position.
 *
 * @param  index index of element to return; must be
 *         non-negative and less than th size of this list
 * @return the element at the specified position in this list
 * @throws IndexOutOfBoundsException if the index is out of range
 * 		 ({@coe index < 0 || index >= this.size()})
 */
E get(int index);

// 한글
/**
 * 이 리스트에서 지정한 위치의 원소를 반환한다.
 * 
 * <p>이 메서드는 상수 시간에 수행됨을 보장하지 <i>않는다</i>. 구현에 따라
 * 원소의 위치에 비례해 시간이 걸릴 수도 잇다.
 *
 * @param  index 반환할 원소의 인덱스; 0이상이고 리스트 크기보다 작아야 한다
 * @return 이 리스트에서 지정한 위치의 원소
 * @throws IndexOutOfBoundException index가 범위를 벗어나면,
 * 		 즉, ({@code index < 0 || index >= this.size()})이면 발생한다.
 */
E get(int index);


// @implSpec 예시
@implSpec
This implementation returns {@code this.size() == 0}.
// 이 구현은 {@code this.size() == 0}의 결과를 반환한다.


// 제네릭 타입 예시
@param <K> the type of Keys maintained by this map
// @param <K> 이 맵이 관리하는 키의 타입


// 열거 타입 예시 - 모든 멤버에 주석을 달아야 함
public enum OrchestraSection {
	/** Woodwins, such as flute, clarinet, and oboe. */
  /** 플루트, 클라리넷, 오보 같은 목관악기 */
	WOODWIND,
	… // 생략


// Retention / Target 예시
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface ExceptionTest {
	/**
	 * 이 애너테이션을 단 테스트 메서드가 성공하려면 던져야 하는 예외.
	 * (이 클래스의 하위 타입 예외는 모두 허용된다.)
	 */
	Class<? extends Throwable> value();
}
```

이외에도 스레드 안전 수준을 반드시 API 설명에 포함해야 한다.
잘 쓴 문서인지 확인하려면 자바독 유틸리티가 생성한 웹페이지를 참고해야 한다.

> **::핵심 정리::** 
> 문서화 주석은 API를 문서화하는 가장 효과적인 방법이다. 공개 API라면 빠짐없이 설명을 달아야 한다. 표준 규약을 일관되게 지키고, 문서화 주석에 임의의 HTML 태그를 사용할 수 있음을 기억하라. (HTML 메타문자는 특별하게 취급해야 한다)

