# try-finally보다는 try-with-resources를 사용하라

- 자바 라이브러리에는 close를 통해 직접 닫아줘야 하는 메소드가 많다.
- 자원 닫기가 제대로 이루어지지 못한다면 성능 문제로 이어지며, 안전망으로 사용하는 finalizer는 위험하다.
- 대신해서 쓰이는 것이 try-finally이다.

## try-finally

```java
static String firstLineOfFile(St ring path) throws lOException {
	BufferedReader br = new BufferedReader(new FileReader(path));
	try {
		return br.readLine();
	} finally {
		br.close();
	}
}
```
- 자원이 하나일 때 try-finally 구문을 사용한 예시

```java
static void copy(String src, String dst) throws lOException {
	Inputstream in = new FilelnputStream(src);
	try {
		OutputStream out = new FileOutputStream(dst);
		try {
			byte[] buf = new byte[BUFFER_SIZE];
			int n;
			while ((n = in.read(buf)) >= 0)
				out.write(buf, 0, n);
		} finally {
			out.close();
		}
	}
	finally {
		in.close();
	}
}
```
- 하지만 만약 자원이 두개 이상이 된다면 더러워진다.
- 또한 try-finally를 사용하면 디버깅이 힘들어진다.
	- 예를 들어 위의 firstLineOfFile를 살펴보자
	- 만약 물리적인 문제가 생겨 readLine이 실패한다면, close역시도 실패할 것이다.
	- 그럴 경우 스택 추적 내역에서는 디버깅에 중요한 readLine이 아니라 close에 대한 오류만 보여준다.

## try-with-resources
- try-with-resources 구조는 java 7에서 처음 생겨났다.
- 이 구조를 사용하기 위해서는 해당 자원이 AutoClosable 인터페이스를 구현해야 한다.
	- AutoClosable는 자바에서 자원을 자동으로 해제하기 위한 인터페이스이다.

```java
static String firstLineOfFile(St ring path) throws lOException {
	try (BufferedReader br = new BufferedReader(
			new FileReader(path))) {
		return br.readLine();
	}
}
```
- 첫번째 예제를 try-with-resources를 통해 처리하는 코드이다.

```java
static void copy(String src, String dst) throws lOException {
	try (InputStream in = new FilelnputStream(src);
		OutputSt ream out = new FileOutputStream(dst)) {
			byte[] buf = new byte[BUFFER_SIZE];
			int n;
			while ((n = in.read(buf)) >= 0)
				out.write(buf, 0, n);
	}
}
```
- try-with-resources를 사용해 여러개의 자원을 처리하는 모습이다.


### 장점
- 가장 먼저 기존 코드에 비해 읽기 쉽고 이해하기 수월해지는 코드가 된다.
- 또한 문제를 진단하기도 훨씬 쉬워진다.
	- 예를 들어 첫번째 예제의 readline과 close에서 모두 문제가 발생하면, close에서의 예외는 숨겨지고 readline에서의 오류가 기록된다.
	- close에서의 오류 역시 (suppressed)를 달고 출력된다.
	- Throwable의 getSuppressed를 이용하면 코드에서 가져오는 것도 가능하다.

- catch 절을 활용하면 try문의 중첩 없이 예외를 처리하는 것이 가능하다.

```java
static String firstLineOfFile(St ring path, String defaultVal) {
	try (BufferedReader br = new BufferedReader(
			new FileReader(path))) {
		return br.readLine();
	} catch (lOException e) {
		return defaultVal;
	}
```
- 앞 예제의 firstLineofFile을 catch절과 함께 쓴 모습

## 핵심 정리

> 꼭 회수해야 하는 자원을 다룰 때는 try-finally 말고, try-with-resources를 사용하자. 예외는 없다. 코드는 더 짧고 분명해지고, 만들어지는 예외 정보도 훨씬 유용하다. try- finally로 작성하면 실용적이지 못할 만큼 코드가 지저분해지는 경우라도, try-with-resources로는 정확하고 쉽게 자원을 회수할 수 있다.