## JPA 및 Hibernate의 1차 캐시

JPA(Hibernate)에서 엔티티 매니저를 사용해서 persist, merge, remove를 했을 때 엔티티의 상태는 변경(New, Managed, Removed, ..)되지만, flush나 트랜잭션이 커밋될 때 까지 데이터베이스에 동기화되지는 않는다.

트랜잭션이 커밋되었을 때 변경을 한번에 데이터베이스에 반영하기 위해

JPA(Hibernate)의 영속성 컨텍스트 내부에는 엔티티를 저장하는 1차 캐시가 존재한다. 

1차 캐시는 **트랜잭션이 시작되고 종료될 때 까지만 유효한 트랜잭션 단위의 캐시**다. 애플리케이션 단위의 캐시를 사용하고 싶다면 2차 캐시를 활성화해야 한다.

## Hibernate의 1차 캐시 구현

내부적으로 Hibernate는 엔티티를 다음과 같은 Map에 저장한다.

```java
//StatefulPersistenceContext.java
@Override
	public void addEntity(EntityUniqueKey euk, Object entity) {
		if ( entitiesByUniqueKey == null ) {
			entitiesByUniqueKey = new HashMap<>( INIT_COLL_SIZE );
		}
		entitiesByUniqueKey.put( euk, entity );
	}
```

그리고 EntityUniqueKey는 다음과 같이 정의되어 있다.

```java
public class EntityUniqueKey implements Serializable {
	private final String uniqueKeyName;
	private final String entityName;
	private final Object key;
	private final Type keyType;
	private final EntityMode entityMode;
	private final int hashCode;

	@Override
	public boolean equals(Object other) {
		EntityUniqueKey that = (EntityUniqueKey) other;
		return that != null && that.entityName.equals( entityName )
				&& that.uniqueKeyName.equals( uniqueKeyName )
				&& keyType.isEqual( that.key, key );
	}
	
...
}
```

엔티티의 상태가 Managed라는 것은 엔티티가 entitiesByUniqueKey에 저장되어 있음을 의미한다.

JPA 및 Hibernate에서 1차 캐시는 HashMap이며 Map의 Key는 식별자를 캡슐화한 객체이며 Value는 엔티티 자체이다.

따라서 JPA EntityManager에서는 동일한 식별자 및 엔티티 클래스 타입을 사용하여 저장된 단 하나의 엔티티만 존재할 수 있다.

1차 캐시에서 동일한 엔티티를 하나만 가질 수 있는 이유는 그렇지 않으면 어떤 것이 동기화되어야 하는 올바른 버전인지 알 수 없어 동일한 데이터베이스 행에 대해 서로 다른 표현을 갖게 될 수 있기 때문이다.

## 읽기 작업에서의 1차 캐시 동작

JPA에서 entitiy를 가져올 때 다음과 같은 작업을 수행한다.

```java
Member member = entityManager.find(Member.class, 1L);
```

그럼 하이버네이트에서 아래와 같이 LoadEntityEvent가 트리거된다. 엔티티를 로드하는 LoadEntityEvent는 아래 그림과 같이 DefaultLoadEventListener에 의해 처리된다.

![https://vladmihalcea.com/wp-content/uploads/2020/10/EntityLoadingControlFlow-1024x262.png](https://vladmihalcea.com/wp-content/uploads/2020/10/EntityLoadingControlFlow-1024x262.png)

그림을 보면 loadFromSessionCache()를 통해 엔티티가 이미 1차 캐시에 저장되어 있는지 확인한다. 저장되어 있을 경우 1차 캐시에 저장된 엔티티를 반환한다.

엔티티가 1차 캐시에서 발견되지 않고 2차 캐시가 활성화된 경우 Hibernate는 2차 캐시를 확인한다.

엔티티가 1차 캐시와 2차 캐시 모두에서 발견되지 않으면 Hibernate는 SQL 쿼리를 사용해 데이터베이스에서 엔티티를 로드한다.

1차 캐시는 엔티티가 영속성 컨텍스트에서 로드되는 횟수와 관계없이 동일한 엔티티 참조가 호출자에게 반환되기 때문에 [엔티티에 대한 애플리케이션 수준의 반복 가능한 읽기 보장](https://vladmihalcea.com/hibernate-application-level-repeatable-reads/)을 제공한다.

데이터베이스에서 엔티티를 불러오면 Hibernate에서 JDBC ResultSet을 가져와서 Java Object 배열로 변환한다. 로드된 상태는 아래 그림처럼 엔티티와 함께 1차 캐시에 저장된다.

![https://vladmihalcea.com/wp-content/uploads/2020/10/EntityLoadedState-1024x435.png](https://vladmihalcea.com/wp-content/uploads/2020/10/EntityLoadedState-1024x435.png)

위 다이어그램에서 볼 수 있듯이 2차 캐시는 로드된 상태를 저장하므로 2차 캐시에 저장된 엔티티를 로드할 때 SQL 쿼리를 실행하지 않고도 엔티티의 상태를 획득할 수 있다.

이러한 이유로 엔티티 로드의 메모리는 엔티티의 상태도 저장해야 하므로 엔티티 객체 자체보다 크다. JPA 영속성 컨텍스트가 flush 될 때 로드된 상태는 **dirty check(변경 감지)를 위해 사용**된다. 

따라서 **엔티티를 수정할 계획이 없다면 엔티티 객체를 인스턴스화한 이후 로드된 상태가 삭제되므로 읽기 전용으로 로드하는 것이 더 효율적이다.**

```java
//조회 목적인 경우 readOnly 활성화
@Transactional(readOnly = true)
	public Page<Summary> getProjectList(Pageable pageable, Request request) {
		return projectRepository.findByCondition(request, pageable).map(ProjectMapper.INSTANCE::toSummary);
	}
```

## 결론

1차 캐시는 JPA 및 Hibernate의 필수 구성으로 활성/비활성화를 선택할 수 있는 옵션이 없다. 1차 캐시는 현재 실행중인 스레드에 바인딩되어 있어 여러 사용자 간에 공유할 수 없다. 따라서 1차 캐시는 스레드 안전성이 보장되지 않는다.

위에서 살펴본 반복 가능한 읽기를 제공하는 것 외에도 1차 캐시는 flush할 때 여러 SQL문을 일괄 처리 할 수 있으므로 읽기-쓰기 트랜잭션 응답시간이 향상된다.

## 참조

[https://vladmihalcea.com/jpa-hibernate-first-level-cache/](https://vladmihalcea.com/jpa-hibernate-first-level-cache/)