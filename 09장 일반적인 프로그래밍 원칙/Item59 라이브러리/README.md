# #EffectiveJava/9장_일반적인프로그래밍원칙/59라이브러리


## 59. 라이브러리를 익히고 사용하라

무작위 정수를 생성하는 코드인데 3가지 문제가 있다

```java
// 흔하지만 문제가 심각한 코드!
static Random rnd = new Random();

static int random(int n) {
	return Math.abs(rnd.nextInt())%n;
}
```

-> n이 크지 않은 2의 제곱수라면 얼마 지나지 않아 같은 수열이 반복된다
-> n이 2의 제곱수가 아니라면 몇몇 숫자가 평균적으로 더 자주 반환된다.
-> 바깥의 수가 종종 튀어나올 수 있다

이 문제를 해결하려면 의사난수 생성기, 정수론, 2의 보수 계산 등에 조예가 깊어야 한다.
Random.nextInt(int)를 사용하면 해결 가능함. (자바7부터는 Random이 아닌 ThreadLocalRandom으로 대체하는 것이 좋다. 고품질의 무작위 수를 생성하고, 속도도 더 빠르다.)



표준 라이브러리를 사용하면 …
1. 코드를 작성한 전문가의 지식과 앞서 사용한 다른 프로그래머들의 경험을 활용할 수 있다
2. 핵심적인 일과 크게 관련 없는 문제를 해결하는데 시간을 허비하지 않아도 된다
3. 따로 노력하지 않아도 성능이 지속해서 개선된다
4. 기능이 계속 추가 된다
5. 각 프로그래머의 코드가 많은 사람에게 낯익은 코드가 된다

메이저 릴리스마다 주목할 만한 수많은 기능이 라이브러리에 추가된다. 

```java
// transferTo 메서드를 이용해 URL의 내용 가져오기 - 자바9부터
public static void main(String[] args) throws IOException {
	try (InputStream in = new URL(args[0]).openStream()) {
		in.transferTo(System.out);
	}
}
```

모든 API 문서를 공부하기 벅차지만, 자바 프로그래머라면 java.lan, java.util, java.io, java.util.concurrent 와 하위 패키지들에 익숙해져야 한다.


**::핵심 정리::** 

> 바퀴를 다시 발명하지 말자. 특별한 기능이 아니라면 누군가 이미 라이브러리 형태로 구현해 놓았을 가능성이 크다. 그런 라이브러리가 있다면, 쓰고 있는지 모르면 찾아봐야 한다. 라이브러리 코드는 많은 개발자의 주목을 받기 때문에 코드 품질도 높아진다.



