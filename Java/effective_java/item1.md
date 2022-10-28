# 생성자 대신 정적 팩터리 메서드를 고려하라

클래스는 클라이언트에 public 생성자 대신 (혹은 생성자와 함께) 정적 팩터리 메서드를 제공할 수 있다. 이 방식에는 장점과 단점이 모두 존재한다.

## 장점

### 1. 이름을 가질 수 있다.

```java
class BigInteger {
	//public 생성자 방식
	public BigInteger(int bitLength, int certainty, Random rnd) {
		BigInteger prime;

		if (bitLength < 2)
			throw new ArithmeticException("bitLength < 2");
		prime = (bitLength < SMALL_PRIME_THRESHOLD
			? smallPrime(bitLength, certainty, rnd)
			: largePrime(bitLength, certainty, rnd));
		signum = 1;
		mag = prime.mag;
	}

	//정적 팩터리 메서드									
	public static BigInteger probablePrime(int bitLength, Random rnd) {
		if (bitLength < 2)
			throw new ArithmeticException("bitLength < 2");

		return (bitLength < SMALL_PRIME_THRESHOLD ?
			smallPrime(bitLength, DEFAULT_PRIME_CERTAINTY, rnd) :
			largePrime(bitLength, DEFAULT_PRIME_CERTAINTY, rnd));
	}
}
```
BigInteger 클래스의 public 생성자와 정적 팩터리 메서드를 비교해보면 어느 쪽이 값이 소수인 
BigInteger를 반환한다는 의미를 잘 설명하는지 이해할 수 있다.

### 2. 호출될 때마다 인스턴스를 새로 생성 하지는 않아도 된다.

```java
class Boolean {
	@HotSpotIntrinsicCandidate
	public static Boolean valueOf(boolean b) {
		return (b ? TRUE : FALSE);
	}
}
```
Boolean.valueOf(boolean) 메서드는 객체를 아예 생성하지 않으면서 boolean 값을 박싱할 수 있다.
따라서 생성 비용이 큰 객체가 자주 요청되는 상황에서 성능을 상당히 끌어올려 준다.

반복되는 요청에 같은 객체를 반환하는 클래스를 인스턴스 통제 클래스라 한다. 인스턴스를 통제하면
클래스를 싱글턴, 인스턴스화 불가로 만들 수 있고 불변 값 클래스에서 동치인 인스턴스가 단 하나뿐임을 보장할 수
있다. 인스턴스 통제는 플라이웨이트 패턴의 근간이 되며, 열거 타입은 인스턴스가 하나만 만들어짐을 보장한다.

### 3. 반환 타입의 하위 타입 객체를 반환할 수 있는 능력이 있다.
이 유연성을 응용하면 구현 클래스를 공개하지 않고도 그 객체를 반환할 수 있어 API를 작게
유지할 수 있다. 프로그래머는 명시한 인터페이스대로 동작하는 객체를 얻을 것임을 알기에
굳이 별도 문서를 찾아가며 실제 구현 클래스가 무엇인지 알아보지 않아도 된다.

### 4. 입력 매개변수에 따라 매번 다른 클래스의 객체를 반환할 수 있다. 

```java
class OAuth2Provider {
	public static OAuth2Provider valueOf(String provider) {
            switch (provider) {
                case "kakao": 
                    return new KakaoProvider();
                    break;
                case "naver":
                    return new NaverProvider();
                    break;
                default:
                    return new GoogleProvider();
            }
    }
}
```
OAuth2Provider 클래스는 정적 팩토리 메서드의 매개변수 값에 따라 OAuth2Provider의 하위 객체인 kakao, naver, google Provider를 반환한다.
클라이언트는 팩터리가 건네주는 객체가 어느 클래스의 인스턴스인지 알 수도 없고 알 필요도 없다.
OAuth2Provider의 하위 클래스이기만 하면 되는 것이다.

### 5. 정적 팩터리 메서드를 작성하는 시점에는 반환할 객체의 클래스가 존재하지 않아도 된다.
이런 유연함은 서비스 제공자 프레임워크를 만드는 근간이 된다.
구현체들을 클라이언트에 제공하는 역할을 프레임워크가 통제하여 **클라이언트를 구현체로부터 분리해준다.**

서비스 제공자 프레임워크 패턴에는 여러 변형이 있는데, 의존 객체 주입(dependency injection, DI) 프레임워크도
강력한 서비스 제공자라고 생각할 수 있다.

이제 단점을 알아볼 차례다.
### 1. 상속을 하려면 public이나 protected 생성자가 필요하다.
public이나 protected 생성자 없이 정적 팩터리 메서드만 제공하면 하위 클래스를 만들 수 없다.
이 제약은 상속보다 컴포지션을 사용하도록 유도하고 불변 타입으로 만들려면 이 제약을 지켜야 한다는 점에서
오히려 장점으로 받아들일 수도 있다.

### 2. 정적 팩터리 메서드는 프로그래머가 찾기 어렵다.
생성자처럼 API 설명에 명확히 드러나지 않으니 사용자는 정적 팩터리 메서드 방식 클래스를
인스턴스화할 방법을 알아내야 한다.

다음은 정적 팩터리 메서드에서 흔히 사용하는 명명 방식들이다.
- from: 매개변수를 하나 받아서 해당 타입의 인스턴스를 반환하는 형변환 메서드
```java
Date d = Date.from(instance);
```
- of: 여러 매개변수를 받아 적합한 타입의 인스턴스를 반환하는 집계 메서드
```java
Set<Rank> faceCards = EnumSet.of(JACK, QUEEN, KING);
```
- valueOf: from과 of의 더 자세한 버전
```java
BigInteger prime = BigInteger.valueOf(Integer.MAX_VALUE);
```
- instance 또는 getInstance: 매개변수로 명시한 인스턴스를 반환하지만, 같은 인스턴스임을 보장하지는 않는다.
```java
StackWalker luke = StackWalker.getInstance(options);
```
- create 또는 newInstance: instance와 같지만, 매번 새로운 인스턴스를 생성해 반환함을 보장한다.
```java
Object newArray = Array.newInstance(classObject, arrayLen);
```
- getType: getInstance와 같으나, 생성할 클래스가 아닌 다른 클래스에 팩터리 메서드를 정의할 때 쓴다. Type은 반환할 객체의 타입이다.
```java
FileStore fs = Files.getFileStore(path);
```
- newType: newInstance와 같으나, 생성할 클래스가 아닌 다른 클래스에 팩터리 메서드를 정의할 때 쓴다.
```java
BufferedReader br = Files.newBufferedReader(path);
```
- type: getType과 newType의 간결한 버전이다.
```java
List<Complaint> litany = Collections.list(legacyLitany);
```

### 핵심 정리
정적 팩터리 메서드와 public 생성자는 각자 쓰임새가 있으니 장단점을 이해하고 사용하는 것이 좋다.
그래도 정적 팩터리를 사용하는게 유리한 경우가 더 많으므로 무작정 public 생성자를 제공하던
습관이 있다면 고치는 것이 좋을 것 같다.

