**김영한님의 자바 ORM 표준 프로그래밍을 읽고 정리한 내용입니다.**

## 영속성 컨텍스트란?

JPA를 이해하는 데 가장 중요한 용어는 영속성 컨텍스트로 해석하자면 **엔티티를 영구 저장하는 환경**이라는 뜻이다. 

```java
em.persist(member);
```

엔티티 매니저를 사용해서 위와 같은 코드를 실행했을 때, 단순히 회원 엔티티를 저장한다고 생각하지만 정확히는 엔티티 매니저를 사용해서 회원 엔티티를 **영속성 컨텍스트에 저장**하는 것이다.

영속성 컨텍스트는 엔티티 매니저를 생성할 때 하나가 만들어지고 엔티티 매니저를 통해 영속성 컨텍스트에 접근하고 관리할 수 있다.

## 엔티티의 생명주기

엔티티에는 4가지 상태가 존재한다.

1. 비영속(new/transient): 영속성 컨텍스트와 전혀 관계가 없는 상태
2. 영속(managed): 영속성 컨텍스트에 저장된 상태
3. 준영속(detached): 영속성 컨텍스트에 저장되었다가 분리된 상태
4. 삭제(removed): 삭제된 상태

![https://user-images.githubusercontent.com/33615669/211193721-428b96e6-aeca-40db-95d8-9e1f263d67e3.png](https://user-images.githubusercontent.com/33615669/211193721-428b96e6-aeca-40db-95d8-9e1f263d67e3.png)

### 비영속(new/transient)

엔티티 객체를 생성한 시점에서 엔티티는 순수한 객체 상태이며 아직 저장하지 않았기 때문에 영속성 컨텍스트나 데이터베이스와는 전혀 관련이 없다. 이 상태를 비영속 상태라고 한다.

```java
//비영속 상태
Member member = new Member();
member.setId("member1");
member.setUsername("회원1");
```

### 영속(managed)

엔티티 매니저를 통해서 엔티티를 영속성 컨텍스트에 저장한 상태를 영속 상태라고 한다. 영속된 엔티티는 영속성 컨텍스트에 의해 관리된다. em.find()나 JPQL을 사용해서 조회한 엔티티도 영속성 컨텍스트가 관리하는 영속 상태를 갖는다.

```java
//객체를 저장한 상태(영속)
em.persist(member);
```

### 준영속(detached)

영속성 컨텍스트가 관리하던 영속 상태의 엔티티를 영속성 컨텍스트가 관리하지 않으면 준영속 상태가 된다. 특정 엔티티를 준영속 상태로 만들려면 em.detach() 메서드를 호출하면 된다. em.close() 메서드로 영속성 컨텍스트를 닫거나 em.clear() 메서드로 영속성 컨텍스트를 초기화해도 영속성 컨텍스트가 관리하던 영속 상태의 엔티티는 준영속 상태가 된다.

```java
//회원 엔티티를 영속성 컨텍스트에서 분리하여 준영속 상태가 된다.
em.detach(member);
```

### 삭제(removed)

엔티티를 영속성 컨텍스트와 데이터베이스에서 모두 삭제한다.

```java
//객체를 영속성 컨텍스트와 데이터베이스에서 삭제
em.remove(member);
```

## 영속성 컨텍스트의 특징

- 영속성 컨텍스트와 식별자 값
    - 영속 상태의 엔티티는 식별자 값(@Id로 기본키와 매핑한 값)이 반드시 있어야 한다. 식별자 값이 없으면 예외가 발생한다.
- 영속성 컨텍스트와 데이터베이스 저장
    - 영속성 컨텍스트에 저장한 엔티티는 트랜잭션을 커밋하는 순간에 데이터베이스에 반영된다. 이것을 플러시(flush)라 한다.
- 영속성 컨텍스트의 장점
    - 1차 캐시
    - 동일성 보장
    - 트랜잭션을 지원하는 쓰기 지연
    - 변경 감지
    - 지연 로딩