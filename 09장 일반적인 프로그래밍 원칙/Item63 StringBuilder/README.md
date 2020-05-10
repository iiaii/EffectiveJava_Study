# #EffectiveJava/9장_일반적인프로그래밍원칙/63StringBuilder


## 63. 문자열 연결은 느리니 주의하라

문자열 연결 연산자로 문자열 n 개를 잇는 시간은 n^2에 비례한다. 문자열은 불변이라서 양쪽의 내용 모두를 복사해야 하므로 성능 저하를 피할 수 없다.

```java
// 문자열 연결 - 느리다!
public String statement() {
	String result = “”;
	for(int i=0 ; i< numItems() ; i++)
		result += lineForItem(i); // 문자열 연결
	return result;
}

// StringBuilder 사용하면 성능이 크게 개선됨! 위에 비해 6.5배 빠름!
public String statement2() {
	StringBuilder sb = new StringBuilder(numItems()*LINE_WIDTH);
	for(int i=0 ; i<numItems() ; i++)
		b.append(lineForItem(i));
	return b.toString();
}
```


**::핵심 정리::** 

> 많은 문자열을 연결할때는 문자열 연결 연산자를 피하고 StringBuilder의 append를 사용하라. (혹은 연결하지 않고 처리할 것)



