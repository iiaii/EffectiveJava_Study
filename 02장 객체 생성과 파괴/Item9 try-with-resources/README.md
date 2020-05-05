# #EffectiveJava/2장_객체생성과파괴/9try-with-resources

## 9. try-finally보다는 try-with-resources를 사용하라

자바 라이브러리에는 close 메서드를 호출해 직접 닫아줘야 하는 자원이 많다
-> InputStream, OutputStream, java.sql.Connection 등

전통적으로 try-finally가 쓰였음
```java
static String firstLineOfFile(String path) throws IOException {
	BufferedReader br = new BufferedReader(new FileReader(path));
	try {
		return br.readLine();
	} finally {
		br.close();
	}
}
// 두개 이상이라면 try-finally가 중첩되기 시작함
// 중첩되면 스택 추적 내역으로 버그를 파악하기도 힘들어 진다
```

자바7이 투척한 try-with-resources로 해결함

```java
static String copy(String src, String dest) throws IOException {
	try(InputStream in = new FileInputStream(src);
		OutputStream out = new FileOutputStream(dest)) {
		byte[] but = new byte[BUFFER_SIZE];
		int n;
		while((n = in.read(bud)) >= 0)
			out.write(bud, 0, n);
	}
}
```

catch 나 finally도 계속 사용할 수 있다.


::핵심 정리::

> 꼭 회수해야 하는 자원을 다룰 때는 try-with-resources를 사용하자. 코드는 더 짧고 분명해지고, 만들어지는 예외 정보도 훨씬 유용하다. 정확하고 쉽게 자원을 회수할 수 있다.


