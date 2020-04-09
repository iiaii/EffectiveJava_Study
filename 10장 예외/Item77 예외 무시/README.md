# #EffectiveJava/10장_예외/77예외무시


## 77. 예외를 무시하지 말라

API 설계자가 메서드 선언에 예외를 명시하는 것은 적절한 조치를 취하라고 말하는 것이다. 이를 무시해서는 안된다.

```java
// catch 블록을 비워두면 예외가 무시된다. 아주 의심스러운 코드!!
try {
	… //
} catch(SomeException e) {
}
```

catch 블록을 비워두면 예외가 존재할 이유가 없어진다. (큰 참사로 이어질 수도 있다)

예외를 무시하기로 했다면 catch 블록 안에 그렇게 결정한 이유를 주석으로 남기고 예외 변수의 이름도 ignored로 바꿔놓자.

```java
Future<Integer> f = exec.submit(planarMap::chromaticNumber);
int numColors = 4; // 기본 값. 어떤 지도라도 이 값이면 충분하다
try {
	numColors = f.get(1L, TimeUnit.SECONDS);
} catch (TimeOutException | ExecutionException ignored) {
	// 기본 값을 사용한다 (색상 수를 최소화하면 좋지만, 필수는 아니다).
}
```

검사, 비검사 예외에 똑같이 적용되며 catch 블록을 못 본 척 지나치면 그 프로그램은 오류를 내재한 채 동작하게 된다. (처리하거나 전파되게만 나둬도 디버깅 정보를 남긴채 프로그램이 신속히 중단될 수 있다)