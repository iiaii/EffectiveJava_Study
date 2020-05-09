# #EffectiveJava/10장_예외/72표준예외


## 72. 표준 예외를 사용하라


숙련된 프로그래머는 그렇지 못한 프로그래머보다 더 많은 코드를 재사용한다. 예외도 마찬가지로 재사용하는 것이 좋으며, 자바 라이브러리는 대부분 API에서 쓰기에 충분한 수의 예외를 제공한다.

표준 예외를 재사용하면 얻는게 많다. 그중 최고는 다른 사람이 익히고 사용하기 쉬워진다. 덕분에 읽기도 쉬워진다. 예외 클래스 수가 적을 수록 메모리 사용량도 줄고 클래스를 적재하는 시간도 적게 걸린다.


가장 많이 재사용되는 예외는 IllegalArgumentException(아이템49)이다. 호출자가 인수로 부적절한 값을 넘길때 던지는 예외로 반복횟수를 지정하는 매개변수에 음수를 건넬 때 쓸 수 있다.

IllegalStateException도 자주 재사용된다. 이 예외는 대상 객체의 상태가 호출된 메서드를 수행하기에 적합하지 않을 때 주로 던진다. 예컨대 제대로 초기화되지 않은 객체를 사용하려 할 때 던질 수 있다.

메서드가 던지는 모든 예외를 잘못된 인수나 상태라고 뭉뚱그릴 수도 있지만, 그중 특수한 일부는 따로 구분해 쓰는 게 보통이다. null 값을 허용하지 않는 메서드에 null을 건네면 관례상 IllegalArgumentException이 아닌 NullPointerException을 던진다. 어떤 시퀀스의 허용 범위를 넘는 값을 건넬때도 IllegalArgumnetException 보다 IndexOutOfBoundsException을 던진다.

ConcurrentModificationException은 단일 스레드에서 사용하려고 설계한 객체를 여러 스레드가 동시에 수정하려 할 때 던진다. 이 예외는 문제가 생길 가능성을 알려주는 정도의 역할로 쓰인다.
사실 동시 수정을 확실히 검출할 수 있는 안정된 방법은 없으니
UnsupportedOperationException은 클라이언트가 요청한 동작을 대상 객체가 지원하지 않을 때 던진다. 대부분 객체는 자신이 정의한 메서드를 모두 지원하니 흔히 쓰이는 예외는 아니다. 보통은 구현하려던 인터페이스의 메서드 일부를 구현할 수 없을 때 쓰는데, 원소만 넣을 수 있는 List 구현체에 누군가 remove메서드를 호출하는 경우에 이 예외를 던진다.

Exception, RuntimeException, Throwable, Error는 직접 사용하지 말자. 이 클래스들은 추상 클래스라고 생각하길 바란다. 이 예외들은 다른 예외들의 상위 클래스 이므로, 여러 셩격의 예외들을 포괄하는 클래스는 안정적인 테스트가 불가능 하다.

---
예외								주요 쓰임
---
IllegalArgumentException			허용하지 않는 값이 인수로 건네졌을때
								(null은 NullPointerException으로 처리)
IllegalStateException				객체가 메서드 수행하기 적절치 않은 상태일 때
NullPointerException				null을 허용 않는 메서드에 null을 건넸을 때
IndexOutOfBoundsException		인덱스가 범위를 넘어섰을 때
ConcurrentModificationException 	허용하지 않는 동시 수정이 발견됐을 때
UnsupportedOperationException 	호출한 메서드를 지원하지 않을 때
---

이외에도 다양한 표준 예외가 있고 이를 확장해도 좋다. (단, 예외는 직렬화할 수 있다는 사실을 기억해야 함. 왠만하면 안만드는게 좋음)

