# Optional

상태: 완료

## Optional이란?

Java 8에 도입된 타입으로 NullPointException 예외를 방지하기 위해 등장했다.

Optional 객체는 변수 또는 함수 호출의 반환 값을 래핑하며 다음 상태 중 하나일 수 있다.

- present - 래핑하는 데이터가 null이 아닌 경우
- empty - 데이터가 null 값인 경우 래핑하는데 사용

## Optional 클래스의 메서드

Optional 메서드를 코드 예제와 함께 알아보도록 하겠다. 아래 예제에서 Member에는 ‘Inwoo’라는 이름을 가진 Member는 존재하지만 ‘Donghun’이라는 멤버는 존재하지 않는다고 가정한다.

### 1. isPresent()

isPresent()를 사용하여 옵셔널 객체의 상태를 확인할 수 있다. 옵셔널 객체가 비어 있으면 false를 반환하고 값을 가지고 있다면 true를 반환한다.

```java
Optional<Member> member = MemberRepository.findByName("Inwoo");
if(member.isPresent()) {
		System.out.println(member.getName + " 회원을 찾았습니다.");
}
```

### 2. isEmpty()

위와 반대로 isEmpty()는 옵셔널 객체가 null 값을 래핑하고 있는지 확인할 수 있다.

```java
Optional<Member> member = MemberRepository.findByName("Donghun");
if(member.isEmpty()) {
		System.out.println(member.getName + " 회원을 찾을 수 없습니다.");
}
```

### 3. get()

get()은 Optional 내부의 데이터에 액세스하는 데 사용할 수 있다. 옵셔널 값이 존재하면 데이터를 반환하지만, 옵셔널이 비어 있으면 NoSuchElementException이 발생한다.

```java
Optional<Member> member = MemberRepository.findByName("Inwoo");
System.out.println(member.get().getName()); // Inwoo 출력

Optional<Member> member1 = MemberRepository.findByName("Donghun");
System.out.println(member1.get().getName()); // NoSuchElementException 예외 발생
```

### 4. orElseThrow()

orElseThrow()는 get()과 똑같은 동작을 한다. 데이터가 있으면 데이터를 반환하고 비어 있으면 NoSuchElementException 예외를 throw 한다.

get 메서드를 통해 예외를 발생시키는 것은 다소 직관적이지 않기 때문에 orElseThrow를 사용하여 데이터가 없는 경우 예외가 발생함을 암시하는 것이 좋다.

```java
Optional<Member> member = MemberRepository.findByName("Inwoo");
System.out.println(member.orElseThrow().getName()); // Inwoo 출력

Optional<Member> member1 = MemberRepository.findByName("Donghun");
System.out.println(member1.orElseThrow().getName()); // NoSuchElementException 예외 발생
```

### 5. orElseThrow(exceptionSupplier)

해당 메서드는 orElseThrow()를 오버로딩한 메서드로 데이터가 비어있는 경우 발생하는 예외를 선택하여 사용자 지정 예외를 발생시키거나 특정 예외 메시지를 제공할 수 있다.

```java
Optional<Member> member = MemberRepository.findByName("Inwoo");
System.out.println(member.orElseThrow(IllegalStateException::new).getName()); // Inwoo 출력

Optional<Member> member1 = MemberRepository.findByName("Donghun");
System.out.println(member1.orElseThrow(() -> new CustomException("멤버를 찾을 수 없습니다.").getName()); // CusotmException 발생
```

### 6. orElse(defaultValue)

orElse() 메서드를 사용하여 기본값을 지정할 수 있다. 데이터가 존재하면 데이터가 반환되고 존재하지 않는다면 지정된 기본값이 대신 반환된다.

```java
Optional<Member> member = MemberRepository.findByName("Inwoo");
System.out.println(member.orElse(new Member("Donghun"))); // Inwoo 객체 출력

Optional<Member> member1 = MemberRepository.findByName("Donghun");
System.out.println(member1.orElse(new Member("Donghun"))); // 새로운 Member 객체 출력
```

### 7. orElseGet(defaultValueSupplier)

orElseGet 메서드는 orElse와 매우 유사한 메서드로 유일한 차이점은 supplier로 default 객체를 반환한다는 점이다. 이 방식의 장점은 supplier 함수가 lazy하게 실행된다는 점이다.

Optional 객체에 값이 존재하면 supplier는 실행되지 않는다. 이를 통해 불필요하게 객체를 생성하거나, 최악의 경우 외부 호출이나 DB 작업을 방지할 수 있다. 따라서 orElse보다 orElseGet을 사용하는 것이 좋다.

```java
Optional<Member> member = MemberRepository.findByName("Inwoo");
System.out.println(member.orElse(new Member("Donghun"))); // 새로운 Member 인스턴스를 생성하지만 Inwoo 멤버를 출력한다.

Optional<Member> member1 = MemberRepository.findByName("Inwoo");
System.out.println(member1.orElseGet(() -> new Member("Donghun"))); // orElseGet()은 새로운 객체를 생성하지 않고 Inwoo 멤버를 출력한다.
```

### 8. ifPresent(consumer)

ifPresent는 Consumer 인터페이스를 매개변수로 구현해야 한다. 옵셔널 객체에 값이 존재하면 consumer가 적용되고 그렇지 않으면 무시된다.

```java
Optional<Member> member1 = MemberRepository.findByName("Inwoo");
member.ifPresent(member -> System.out.println(member.getName() + "회원을 찾았습니다.");
// Inwoo 회원을 찾았습니다. 출력
Optional<Member> member2 = MemberRepository.findByName("Donghun");
member2.ifPresent(member -> System.out.println(member.getName() + "회원을 찾았습니다.");
//아무 동작이 일어나지 않는다.
```

### 9. ifPresentOrElse(consumer, runnable)

데이터가 있는 경우 실행할 동작과 Optional이 비어있는 경우 실행될 동작을 제공하려는 경우 ifPresentOrElse를 사용할 수 있다.

```java
//검색된 회원 이름을 출력한다.
Optional<Member> member1 = MemberRepository.findByName("Inwoo");
member1.ifPresentOrElse(
		member -> System.out.println(member.getName() + "회원을 찾았습니다."),
		() -> System.out.println("회원을 찾을 수 없습니다.");
);

//"회원을 찾을 수 없습니다." 출력
Optional<Member> member2 = MemberRepository.findByName("Donghun");
member2.ifPresentOrElse(
		member -> System.out.println(member.getName() + "회원을 찾았습니다."),
		() -> System.out.println("회원을 찾을 수 없습니다.");
);
```

### 10. filter(filterPredicate)

filter는 Predicate를 수신하여 다른 Optional 객체를 반환한다. 여기에는 3가지 가능한 시나리오가 있다.

- 원래 Optional 객체가 비어 있으면 filter()는 빈 옵셔널을 반환한다.
- 원래 Optional 객체의 데이터가 존재하면 Predicate가 평가되고 ‘false’를 반환하면 filter()는 빈 Optional 객체를 반환한다.
- 원래 Optional 객체의 데이터가 존재하면 Predicate가 평가되고 ‘true’를 반환하면 Optional 래핑 데이터가 반환된다.

```java
//원래 Optional이 비어 있었기 떄문에 false를 출력한다.
Optional<Member> member1 = MemberRepository.findByName("Donghun");
member1 = member1.filter(member -> member.getAge() > 20);
System.out.println(member1.isPresent());

//Inwoo Member의 나이가 30을 초과하지 않기 때문에 빈 옵셔널 객체가 반환되어 false 출력
Optional<Member> member2 = MemberRepository.findByName("Inwoo");
member2 = member2.filter(member -> member.getAge() > 30);
System.out.println(member2.isPresent());

//Inwoo의 나이가 20을 초과하기 때문에 true가 출력된다.
Optional<Member> member3 = MemberRepository.findByName("Inwoo");
member3 = member3.filter(member -> member.getAge() > 20);
System.out.println(member3.isPresent());
```

### 11. map(mappingFunction)

map() 메서드는 함수를 매개변수로 받고 데이터가 있는 경우 이를 데이터에 적용한다.

마지막으로 map은 다른 타입의 또 다른 Optional 객체를 반환한다. 예를 들면 Member의 특정한 값에 매핑하면 Optional<Member>를 Optional<Integer>로 변환할 수 있다.

```java
//member1의 age 값인 25가 age1 객체에 래핑된다.
Optional<Member> member1 = MemberRepository.findByName("Inwoo");
Optional<Integer> age1 = member1.map(Member::getAge);

//member2가 empty이므로 빈 Optional 객체가 반환된다.
Optional<Member> member2 = MemberRepository.findByName("Donghun");
Optional<Integer> age2 = member21.map(member -> member.getAge());
```

### 12. flatMap(mappingFunction)

map의 함수가 옵셔널을 반환하는 경우 중첩된 옵셔널 구조가 되기 때문에 flatMap을 사용해야 한다.

```java
//findByName은 Optional<Member>를 리턴하기 때문에 중첩된 옵셔널 구조가 생성된다.
Optional<String> memberName = Optional.of("Inwoo");
Optional<Optional<Member>> member1 = memberName.map(name -> MemberRepository.findByName(name));

//flatMap은 map을 처리하고 중첩된 Optional 구조를 처리한다.
Optional<String> memberName2 = Optional.of("Inwoo");
Optional<Member> member2 = memberName2.flatMap(name -> MemberRepository.findByName(name));
```

### 13. Optional.ofNullable(nullableObject)

Optional의 정적 메소드인 ofNullable은 전달된 데이터를 기반으로 옵셔널을 생성한다. 데이터가 null이면 빈 옵셔널을 반환하고 데이터가 존재하면 Optional에 데이터를 래핑해서 반환한다.

```java
Optional<String> optional1 = Optional.ofNullable("some String");
System.out.println(optional1.isPresent()); // true

Optional<String> optional1 = Optional.ofNullable(null);
System.out.println(optional1.isPresent()); // false
```

### 14. Optional.empty()

정적 메소드 empty()는 빈 옵셔널을 생성한다.

```java
Optional<String> optional = Optional.empty();
System.out.println(optional.isPresent()); // false;
```

### 15. Optional.of()

옵셔널을 생성하는 세 번째이자 마지막 방법은 정적 메소드 of()를 사용하는 것이다.

이 메서드는 null이 아닌 값에만 사용해야 한다. null일 경우 NullPointerException이 발생한다.

```java
Optional<String> optional1 = Optional.of("some value");
System.out.println(optional1.isPresent()); // true

Optional<String> optional2 = Optional.of(null); // throw NullPointerException
```

## 참고

[https://medium.com/codex/each-of-the-15-methods-of-java-optionals-api-explained-in-less-than-20-seconds-c5711233d209#id_token=eyJhbGciOiJSUzI1NiIsImtpZCI6IjI3Yjg2ZGM2OTM4ZGMzMjdiMjA0MzMzYTI1MGViYjQzYjMyZTRiM2MiLCJ0eXAiOiJKV1QifQ.eyJpc3MiOiJodHRwczovL2FjY291bnRzLmdvb2dsZS5jb20iLCJuYmYiOjE2NjkxOTAwODgsImF1ZCI6IjIxNjI5NjAzNTgzNC1rMWs2cWUwNjBzMnRwMmEyamFtNGxqZGNtczAwc3R0Zy5hcHBzLmdvb2dsZXVzZXJjb250ZW50LmNvbSIsInN1YiI6IjEwMDk2MDQ0NjkzMDcyODYwNzQ0OSIsImVtYWlsIjoiYmVzdGlud29vQGdtYWlsLmNvbSIsImVtYWlsX3ZlcmlmaWVkIjp0cnVlLCJhenAiOiIyMTYyOTYwMzU4MzQtazFrNnFlMDYwczJ0cDJhMmphbTRsamRjbXMwMHN0dGcuYXBwcy5nb29nbGV1c2VyY29udGVudC5jb20iLCJuYW1lIjoiaW51IFBhcmsiLCJwaWN0dXJlIjoiaHR0cHM6Ly9saDMuZ29vZ2xldXNlcmNvbnRlbnQuY29tL2EvQUxtNXd1MjVrR3FPcTM3Y1VXYnI1Y2liZHZXVjVkajY1UUZQLXc2M0Z3cWs9czk2LWMiLCJnaXZlbl9uYW1lIjoiaW51IiwiZmFtaWx5X25hbWUiOiJQYXJrIiwiaWF0IjoxNjY5MTkwMzg4LCJleHAiOjE2NjkxOTM5ODgsImp0aSI6ImM2MTYzZGI3ZjgxZDJmYzEzODM1MDE3ZDg5NjA5ZjFmNzU0MTVmNjMifQ.cCmecd_MTwyEURBp_caqhkTpRTnShytwbFqE584fxWKcPjB7ttUKpn4o5N2Uo6v5dDqRcXElM6O9tQcIG5r1BkBkTK6ySo_dk0L4CLGcIDwkKCOYMD0dmVYfZryoq-Ud-3WrqzfVREc85hlJCqO7q81CIuxuz3DyTWJf6vzQqo1Am977i8C_ZWU1cD3kBlA8rxmjYoY71h_tIUNaRQz9r05eW9f-E7iSwylkw-p50qjnViIEhnlfqAusBzhhFJMT6i8y4eLobOu8llcAbumhE2T9tnnGAYNrHL6kBkiT74Dnlngh8y9E74khsnwhTY0yzL03P49BMy64rgZ2sN73eQ](https://medium.com/codex/each-of-the-15-methods-of-java-optionals-api-explained-in-less-than-20-seconds-c5711233d209#id_token=eyJhbGciOiJSUzI1NiIsImtpZCI6IjI3Yjg2ZGM2OTM4ZGMzMjdiMjA0MzMzYTI1MGViYjQzYjMyZTRiM2MiLCJ0eXAiOiJKV1QifQ.eyJpc3MiOiJodHRwczovL2FjY291bnRzLmdvb2dsZS5jb20iLCJuYmYiOjE2NjkxOTAwODgsImF1ZCI6IjIxNjI5NjAzNTgzNC1rMWs2cWUwNjBzMnRwMmEyamFtNGxqZGNtczAwc3R0Zy5hcHBzLmdvb2dsZXVzZXJjb250ZW50LmNvbSIsInN1YiI6IjEwMDk2MDQ0NjkzMDcyODYwNzQ0OSIsImVtYWlsIjoiYmVzdGlud29vQGdtYWlsLmNvbSIsImVtYWlsX3ZlcmlmaWVkIjp0cnVlLCJhenAiOiIyMTYyOTYwMzU4MzQtazFrNnFlMDYwczJ0cDJhMmphbTRsamRjbXMwMHN0dGcuYXBwcy5nb29nbGV1c2VyY29udGVudC5jb20iLCJuYW1lIjoiaW51IFBhcmsiLCJwaWN0dXJlIjoiaHR0cHM6Ly9saDMuZ29vZ2xldXNlcmNvbnRlbnQuY29tL2EvQUxtNXd1MjVrR3FPcTM3Y1VXYnI1Y2liZHZXVjVkajY1UUZQLXc2M0Z3cWs9czk2LWMiLCJnaXZlbl9uYW1lIjoiaW51IiwiZmFtaWx5X25hbWUiOiJQYXJrIiwiaWF0IjoxNjY5MTkwMzg4LCJleHAiOjE2NjkxOTM5ODgsImp0aSI6ImM2MTYzZGI3ZjgxZDJmYzEzODM1MDE3ZDg5NjA5ZjFmNzU0MTVmNjMifQ.cCmecd_MTwyEURBp_caqhkTpRTnShytwbFqE584fxWKcPjB7ttUKpn4o5N2Uo6v5dDqRcXElM6O9tQcIG5r1BkBkTK6ySo_dk0L4CLGcIDwkKCOYMD0dmVYfZryoq-Ud-3WrqzfVREc85hlJCqO7q81CIuxuz3DyTWJf6vzQqo1Am977i8C_ZWU1cD3kBlA8rxmjYoY71h_tIUNaRQz9r05eW9f-E7iSwylkw-p50qjnViIEhnlfqAusBzhhFJMT6i8y4eLobOu8llcAbumhE2T9tnnGAYNrHL6kBkiT74Dnlngh8y9E74khsnwhTY0yzL03P49BMy64rgZ2sN73eQ)