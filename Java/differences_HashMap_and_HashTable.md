# HashMap과 HashTable의 차이점

상태: 완료

Java에서 Key-Value 쌍을 저장하는 자료구조로 HashTable과 HashMap이 있다.

HashTable 또는 HashMap을 사용할 때 키와 키에 연결하려는 값을 지정하면 키가 해시되고 결과 해시코드가 테이블 내에서 값이 저장되는 인덱스로 사용된다.

그렇다면 기본적인 구조는 비슷한 두 자료구조의 차이점은 무엇인지 알아보도록 하자

# HashMap vs HashTable

- HashMap은 동기화되지 않기 때문에 thread-safe 하지 않아 싱글 스레드 환경에서 사용하는 것이 좋다. HashTable은 동기화를 보장하여 멀티 스레드 환경에서 thread-safe 하다.
- HashMap은 하나 또는 여러개의 null 키를 허용하는 반면 HashTable은 키 또는 값에 null 값을 허용하지 않는다.
- 멀티 스레드 환경이 아닌 경우 HashMap이 성능적으로 HashTable보다 빠르기 때문에 더 선호된다. (HashTable은 스레드의 대기 시간이 증가하여 성능이 저하되기 때문이다.)

## HashTable이 null을 허용하지 않는 이유

위의 비교를 보면 HashMap에서는 Key나 Value에 null 값을 허용하는 반면 HashTable은 허용하지 않는다고 하는데 이유가 무엇일까?

HashTable에서 객체를 저장하고 검색하려면 키로 사용되는 객체가 hashCode 메서드와 equals 메서드를 구현해야 하기 때문이다. null은 객체가 아니므로 이러한 메서드를 구현할 수 없다. 

## 결론

멀티 스레드 환경에서 legacy한 HashTable을 사용하는 것보단 HashMap의 동기화 문제를 보완하기 위해 구현된 **ConcurrentHashMap**을 사용하는 것이 좋다.

 HashTable은 스레드간 락을 거는 반면에 CouncurrentHashMap은 Entry 아이템별로 락을 걸어 상대적으로 속도가 빠르다. 

## 참고

[https://www.geeksforgeeks.org/differences-between-hashmap-and-hashtable-in-java/](https://www.geeksforgeeks.org/differences-between-hashmap-and-hashtable-in-java/)

[https://tecoble.techcourse.co.kr/post/2021-11-26-hashmap-hashtable-concurrenthashmap/](https://tecoble.techcourse.co.kr/post/2021-11-26-hashmap-hashtable-concurrenthashmap/)