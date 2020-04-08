# #EffectiveJava/9장_일반적인프로그래밍원칙/68명명규칙

[Bool 변수 이름 제대로 짓기 위한 최소한의 영어 문법 · Soojin Ro](https://soojin.ro/blog/naming-boolean-variables)


## 68. 일반적으로 통용되는 명명 규칙을 따르라


자바의 명명 규칙은 크게 철자와 문법 두 범주로 나뉜다.

철자 규칙은 패키지, 클래스, 인터페이스, 메서드, 필드, 타입 변수의 이름을 다룬다. 

패키지와 모듈이름은 각 요소를 점(.)으로 구분하여 계층적으로 짓는다. 요소들은 모두 소문자 알파벳 혹은 (드물게) 숫자로 이뤄진다. 인터넷 도메인 이름을 역순으로 사용한다.
(edu.cmu, com.google, org,eff …)
예외적으로 표준 라이브러리와 선택적 패키지들은 각각 java와 javax로 시작한다. 도메인 이름을 패키지 이름의 접두어로 변환하는 자세한 규칙은 자바 언어 명세에 적혀 있다.
패키지 이름의 나머지는 해당 패키지를 설명하는 하나 이상의 요소로 이뤄진다. (8자 이하, 첫글자만 딴 awt 같은 경우도 가능)


클래스와 인터페이스(열거타입 애너테이션 포함)의 이름은 하나 이상의 단어로 이뤄지며, 각 단어는 대문자로 시작한다 (List, FutherTask 등) 여러 단어의 첫 글자만 딴 약자, 통용되지 않는 줄임말은 쓰지 않는다 (max 같은 것은 제외)
메서드와 필드 이름은 첫 글자를 소문자로 쓴다는 점만 빼면 클래스 명명 규칙과 같다. (첫 단어가 약자(줄임말)라면 전체가 소문자여야 한다)
단 상수 필드는 (모두 대문자로 쓰며 단어 사이는 _로 구분한다 , MAX_VALUE)
지역변수(매개변수 포함)는 약어를 써도 좋다. (i, denim, houseNum 등)

타입 매개변수 이름은 보통 한문자로 표현한다
- T : 임의의 타입
- E : 컬렉션 원소 타입
- K, V : 맵의 키와 값
- X : 예외
- R : 메서드의 반환 타입
- T, U, V (T1, T2 ..) : 타입의 시퀀스


---
식별자 타입			예
---
패키지와 모듈			org.junit.jupiter.api, com.goolge.common.collect
클래스와 인터페이스	Stream, FutureTask, LinkedHashMap, HttpClient
메서드와 필드			remove, groupingBy, getCrc
상수 필드				MIN_VALUE, NEGATIVE_INFINITY
지역 변수 				i, denim, housNum
타입 매개변수			T, E, K, V, X, R, U, V, T1, T2
---

문법 규칙은 유연한 편임

객체를 생성할 수 없는 클래스의 이름은 보통 복수형 명사로 짓는다 (Collections)
인터페이스 이름은 클래스와 똑같이 짓거나 able, ible로 끝나는 형용사로 짓는다

is, has, get, set 의 메서드 이름이 있고, 객체의 타입을 바꿔서 다른 타입의 또 다른 객체를 반환하는 인스턴스 메서드의 이름은 보통 [toType] 형태로 짓는다 (toString, toArray)
객체의 내용을 다른 뷰로 보여주는 메서드는 [asType] 형태로 짓는다 (asList)
기본 타입 값으로 반환하는 메서드의 이름은 보통 typeValue 형태로 짓는다. (intValue)
정적 팩토리의 이름은 다양하지만 from, of, valueOf, instance, getInstance, newInstance, getType, newType(아이템1)을 주로 사용한다

> **::핵심 정리::** 
> 표준 명명 규칙을 체화하여 자연스럽게 베어 나오도록 하자. 철자 규칙은 직관적이라 모호한 부분이 적지만 문법은 다양하고 느슨하다.


