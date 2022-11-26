# Stream이란?

기존 Java에서 컬렉션 데이터를 처리할 때 for, foreach 루프문을 사용하며 컬렉션 요소들을 하나씩 다뤄야 했다. 이 때 코드가 복잡해지는 문제점을 해결하기 위해 JDK 8부터 함수형 프로그래밍이 가능하도록 구현된 API로 데이터를 추상화하고 처리하는데 자주 사용되는 함수들이 정의되어 있다.

Stream API를 사용하지 않는 경우와 Stream API를 사용한 경우의 코드를 비교해보도록 하겠다.

배열과 리스트를 정렬하는 코드에서 Stream API를 사용하지 않는 경우 아래와 같이 작성할 수 있다.

### Stream API 미사용

```java
String[] fruitArr = {"banana", "apple", "grape", "orange"};
List<String> fruitList = Arrays.asList(fruitArr);

//원본의 데이터가 직접 정렬되어 변경됨
Arrays.sort(fruitArr);
Collections.sort(nameList);

for(String fruit : fruitArr) {
		System.out.println(fruit);
}

for(String fruit : fruitList) {
		System.out.println(fruit);
}
```

### Stream API 사용

Stream API를 사용하여 함수형으로 리팩토링하면 위의 코드보다 더욱 간결하고 가독성 있게 코드를 작성할 수 있으며, 원본의 데이터에 변형을 가하지 않는다.

```java
String[] fruitArr = {"banana", "apple", "grape", "orange"};
List<String> fruitList = Arrays.asList(fruitArr);

//원본의 데이터가 아닌 별도의 Stream을 생성함
Stream<String> arrayStream = Arrays.stream(fruitArr); // 배열의 스트림 생성
Stream<String> listStream = fruitList.stream(); // Collection의 스트림 생성
IntStream stream = Intstream.range(4,10); // 원시타입 스트림 생성

//복사된 데이터를 정렬하여 출력
arrayStream.sorted().forEach(System.out::println);
listStream.sorted().forEach(System.out::println);
```

# Stream API의 특징

스트림은 Stream이라는 별도 객체로 기능을 이용할 수 있으며 3가지의 큰 특징이 있다.

## 1. 원본 데이터를 변경하지 않는다.

Stream은 원본의 데이터를 조회하여 별도의 요소들을 생성하여 Stream 객체를 생성한다. 

그러므로 스트림 연산을 통해 아무리 데이터를 조작하더라도 원본 데이터는 변경되지 않는다.

```java
List<String> fruits = List.of("apple", "orange", "banana", "grape");
List<String> fruits2 = fruits.stream().sorted().collect(Collectors.toList());
//원본 객체는 변경되지 않는 값을 가지고 있는 것을 확인할 수 있다.
System.out.println(fruits);
System.out.println(fruits2);
```

## 2. Stream은 일회용이다.

Stream은 데이터를 모두 읽고나면 사라지는 일회용이기 때문에 재사용이 불가능하다.

만약 닫힌 Stream을 다시 사용한다면 IllegalStateException 런타임 에러가 발생한다.

```java
List<String> fruits = List.of("apple", "orange", "banana", "grape");
Stream<String> fruitStream = fruits.stream();

List<String> fruits2 = fruitStream.sorted().collect(Collectors.toList());

//Stream이 이미 사용된 후 닫힌 상태이므로 런타임 에러 발생
fruitStream.forEach(System.out::println);
```

## 3. 내부적으로 반복 작업을 처리한다.

기존 컬렉션 객체의 요소를 다루기 위해서는 for문이나 foreach문과 같은 반복문을 사용해야 했지만 Stream은 내부적으로 반복 작업을 하기 때문에 보다 간결한 코드 작성이 가능하다.

```java
//forEach 메서드 내부에 반복문이 숨겨져 있다.
fruitStream.forEach(System.out::println);
```

# Stream API의 연산 종류

Stream에 대한 연산은 크게 생성하기, 가공하기, 결과 만들기 3가지 단계로 나눌 수 있다.

### 1. 생성하기

- Stream 객체를 생성하는 단계
- Stream은 재사용이 불가능하므로, 닫히면 다시 생성해주어야 한다.

### 2. 가공하기

- 원본의 데이터를 별도의 데이터로 가공하기 위한 중간 연산
- 연산 결과를 Stream으로 다시 반환하기 때문에 연속해서 중간 연산을 이어갈 수 있다.

### 3. 결과 만들기

- 가공된 데이터로부터 원하는 결과를 만들기 위한 최종 연산
- Stream의 요소들을 소모하면서 연산이 수행되기 때문에 1번만 처리할 수 있다.

```java
List<String> fruits = List.of("apple", "orange", "banana", "grape");

fruits
		.stream() // 생성하기
		.map(String::toUpperCase) // 가공하기
		.sorted() // 가공하기
		.collect(Collectors.toList()); // 결과 만들기
```

위 코드에서 stream()을 통해 Stream 객체를 생성한다. 그리고 map과 sorted를 통해 중간 연산을 하고 있다. 중간 연산은 Stream 객체를 다시 반환하므로 연결해서 사용할 수 있다. 마지막으로 collect 최종 연산을 통해 결과를 List로 반환한다. 

# Stream API 중간 연산

생성한 Stream 객체에서 요소들을 가공하기 위해 중간 연산이 필요하다. 중간 연산의 파라미터로는 함수형 인터페이스들이 사용되며 여러 개의 중간 연산이 연결되도록 반환값으로 Stream을 반환한다.

## Filter

Stream의 데이터 중 설정한 조건에 맞는 데이터만 모아서 스트림을 반환한다. filter 함수의 인자로는 함수형 인터페이스 Predicate를 받고 있으므로 boolean을 반환하는 람다식을 작성하여 filter 함수를 구현할 수 있다.

```java
//1부터 5까지의 숫자 중 3이상의 숫자만 골라 리스트로 반환하는 코드
List<Integer> list = List.of(1,2,3,4,5);
List<Integer> greaterThan3 = list.stream().filter(n -> n >= 3).collect(Collectors.toList());
```

## Map

Map은 기존 Stream 요소들을 특정한 형태로 변환하여 새로운 Stream을 반환하는 연산이다.

map 함수의 인자로는 Function을 받고 있다.

```java
//각 요소에 대해 1씩 빼서 새로운 리스트로 반환
List<Integer> list = List.of(1,2,3,4,5);
List<Integer> minusOne = list.stream().map(n -> n - 1).collect(Collectors.toList());
```

## FlatMap

flatMap()은 컬렉션 안에 컬렉션이 존재하는 경우 중첩 구조를 제거하기 위해 사용하는 중간 연산이다. flatMap은 Function 인터페이스를 매개변수로 받고 있다.

예를 들어 리스트 안에 배열이 있는 경우 List에 대한 스트림과 그 안에 배열에 대한 스트림이 중첩으로 생성되어 Stream<Stream> 형태가 되므로 코드가 복잡해진다. 이 때 flatMap을 사용하여 1중 리스트로 변환하면 코드가 간결해진다.

```java
List<Integer[]> list = new ArrayList();
list.add(new Integer[]{1,5,3});
list.add(new Integer[]{5,6,7,8});

//map 사용
Set<Integer> set = list.stream()
		.map(innerArr -> Arrays.stream(innerArr).collect(Collectors.toSet()))
		.collect(HashSet::new, Set::addAll, Set::addAll);

//flatMap 사용
Stream<String[]> strStream = Stream.of(new String[] {"a", "b"}, new String[] {"c", "d"});
Stream<String> stream = strStream.flatMap(Arrays::stream);
```

## Sorted

Sorted는 Stream의 요소들을 정렬하는 함수이다. 파라미터가 없는 기본 sorted()는 오름차순 정렬이고 Comparator를 인자로 전달하여 내림차순 또는 별도 방식으로 정렬할 수 있다.

```java
//내림차순으로 정렬
List<Integer> list = List.of(1,2,3,4,5);
List<Integer> reverseList = list.stream()
							.sorted(Comparator.reverseOrder())
							.collect(Collectors.toList());
```

## Distinct

Stream 요소들의 중복된 데이터를 제거하는 함수이다. 이 때 중복된 데이터를 검사하기 위해 Object 클래스의 equals() 메서드를 사용한다. 만약 직접 생성한 클래스에 distinct를 적용하려면 equals와 hashCode를 오버라이딩해야 distinct()가 제대로 적용된다.

```java
List<Integer> list = List.of(1,1,2,2,3,3,4,4,5,5);
List<Integer> list.stream().distinct().collect(Collectors.toList());
// [1, 2, 3, 4, 5]
```

## Peek

Stream의 요소들을 대상으로 Stream에 영향을 주지 않고 특정 연산을 수행하기 위한 함수이다.

peek 함수는 Stream 각 요소들에 대해 특정 작업을 수행할 뿐 결과에 영향을 주지 않는다. 또한 peek 함수는 파라미터로 함수형 인터페이스 Consumer를 인자로 받는다.

```java
int sum = IntStream.of(1,2,3,4,5)
		.peek(System.out::println) // Stream의 각 요소를 출력
		.sum();
```

## 원시 Stream ↔ Stream

일반 Stream 객체를 원시 Stream으로 바꾸거나 그 반대로 변환하는 작업이 필요한 경우 일반적인 Stream 객체는 mapToInt(), mapToLong(), mapToDouble()의 Mapping 연산을 지원하며 그 반대로 원시 객체는 mapToObj를 통해 일반 Stream 객체로 변환할 수 있다.

```java
//IntStream -> Stream<Integer>
Stream<Integer> mapStream = IntStream.range(1,5).mapToObj(i -> i + 1);

//Stream<Integer> -> IntStream
IntStream mapStream2 = Stream.of(1,2,3,4,5).mapToInt(i -> i + 1);
```

# Stream 최종 연산으로 결과 만들기

중간 연산을 통해 가공된 Stream을 바탕으로 최종 연산하여 결과를 만들 수 있다.

## Max/Min/Sum/Average/Count

Stream의 요소들을 대상으로 최솟값, 최댓값, 총합, 평균, 개수를 구하기 위한 최종 연산이다.

max, min, average는 Stream이 비어있는 경우 값을 특정할 수 없기 때문에 Optional로 값이 반환된다.

```java
OptionalInt min = IntStream.of(1, 2, 3, 4, 5).min();
int sum = IntStream.of(1, 2, 3, 4, 5).sum();
```

## Reduce

reduce()는 최종 연산으로 누산기와 연산으로 스트림 안의 요소들을 소모하면서 연산을 수행한 후 연산의 결과 값을 반환한다. reduce 연산은 1번 데이터와 2번 데이터로 연산을 수행한 뒤 결과 값으로 3번 데이터와 같은 연산을 수행한다. 그렇게 마지막 데이터까지 연산을 수행한 후 최종 결과를 반환한다.

```java
Stream.of(1,2,3,4,5).reduce(Integer::sum).get();
```

## Collect

Stream의 요소들을 List, Set, Map 등의 컬렉션으로 만들고 싶은 경우에 사용한다.

collect 함수의 인자로는 어떤 컬렉션으로 만들지 Collector 타입을 전달한다.

List, Set, Map 등의 자주 사용하는 컬렉션은 Collectors 객체에서 static 메소드로 제공한다.

### Collectors.toList

Stream에서 작업한 결과를 List로 반환 받는다.

```java
List<String> nameList = memberList.stream().map(Member::getName).collect(Collectors.toList());
```

### Collectors.joining

Stream에서 작업한 결과를 1개의 String으로 이어붙일 수 있다. Collectors.joining에는 3개의 인자를 받을 수 있다.

- delimiter : 각 요소를 구분시켜주는 구분자
- prefix : 결과 맨 앞에 붙는 문자
- suffix : 결과 맨 뒤에 붙는 문자

```java
String joinName = memberList.stream().map(Member::getName).collect(Collectors.joining());
// 철수영희맹구

String joinName2 = memberList.stream().map(Member::getName).collect(Collectors.joining(", "));
// 철수, 영희, 맹구

String joinName3 = memberList.stream().map(Member::getName).collect(Collectors.joining(", ", "[", "]"));
// [철수, 영희, 맹구]
```

### Collectors.averagingInt / Collectors.summingInt / Collectors.summarizingInt

Stream에서 작업한 결과의 평균, 총합을 구하기 위해서는 Collectors.averagingInt()와 Collectors.summingInt()를 사용할 수 있다.

```java
Double averageAge = memberList.stream().collect(Collectors.averagingInt(Member::getAge));

Integer summingAge = memberList.stream().collect(Collectors.summingInt(Member::getAge));

Integer summingAge2 = memberList.stream().mapToInt(Member::getAge).sum();
```

만약 1개의 Stream으로부터 개수, 합계, 평균, 최댓값, 최솟값을 한번에 얻고 싶은 경우에는 Collectors.summarizingInt()를 사용한다. 이를 이용하면 IntSummaryStatistics 객체가 반환되며 필요한 값에 대해 get 메서드를 이용하여 원하는 값을 얻을 수 있다.

```java
IntSummaryStatistics statistics = memberList.stream().collect(Collectors.summarizingInt(Member::getAge));
// IntSummaryStatistics{count=3, sum=68, min=22, average=22.666667, max=24}
```

### Collectors.groupingBy()

Stream에서 작업한 결과를 특정 그룹으로 묶을 수 있다. 결과는 Map으로 반환되며 매개변수로 함수형 인터페이스 Function을 필요로 한다.

```java
Map<Integer, List<Member>> groupedMeber = memberList.stream().collect(Collectors.groupingBy(Member::getAge));
/*
	{22=[Member{age=22, name='철수'}, Member{age=22, name='영희'}],
		24=[Member{age=24, name='맹구'}]}
*/
```

### Collectors.partitioningBy()

groupingBy가 Function을 사용해서 특정 값을 기준으로 요소들을 그룹핑하였다면, partitioningBy는 Predicate를 받아 Boolean을 키 값으로 파티셔닝한다.

```java
Map<Boolean, List<Member>> mapPartitioned = memberList.stream().collect(Collectors.partitioningBy(m -> m.getAge() > 22));
/*
	{true=[Member{age=24, name='맹구'}],
	 false=[Member{age=22, name='철수'}, Member{age=22, name='영희'}]}
*/
```

## Match

Stream의 요소들이 특정 조건을 충족하는지 검사하고 싶은 경우에 match 함수를 사용한다. match는 함수형 인터페이스 Predicate를 받아 해당 조건을 만족하는지 검사하고 검사 결과를 boolean으로 반환한다.

- anyMatch : 1개의 요소라도 해당 조건을 만족하는가
- allMatch : 모든 요소가 해당 조건을 만족하는가
- nonMatch : 모든 요소가 해당 조건을 만족하지 않는가

```java
List<String> fruits = List.of("apple", "banana", "grape");

boolean anyMatch = fruits.stream().anyMatch(fruit -> fruit.contains("l")); // true
boolean allMatch = fruits.stream().allMatch(fruit -> fruit.length > 5); // false
boolean noneMatch = fruits.stream().noneMatch(fruit -> fruit.startWith("c")); // true
```

## ForEach

Stream의 요소들을 대상으로 특정한 연산을 수행하고 싶은 경우 forEach 함수를 사용할 수 있다. 비슷한 함수로 peek()과 map()이 있다. peek()은 실제 요소들에 영향을 주지 않은 채 작업을 처리하고 Stream을 반환하는 함수이며, map() 또한 Stream을 반환한다는 차이점이 있다.

```java
List<String> fruits = List.of("apple", "banana", "grape");
fruits.stream().forEach(System.out::println);
```



## 참고

[https://mangkyu.tistory.com/115](https://mangkyu.tistory.com/115)

[https://pamyferret.tistory.com/43](https://pamyferret.tistory.com/43)





