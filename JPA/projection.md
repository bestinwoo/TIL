## ****Interface-Based Projections****

인터페이스 기반 프로젝션은 DTO 클래스를 만들지 않고 인터페이스를 사용하게 되는데, Spring이 프로젝션 인터페이스의 프록시 인스턴스를 생성해주게 되어 값을 조회할 수 있게 된다.

### SchoolRankingDto Inteface

우선 Projection한 결과 값을 저장할 인터페이스를 생성한다. 

```java
public interface SchoolRankingDto {
	String getUniversity();
	Long getPoint();
	Long getPersonnel();
}
```

### MemberRepository

```java
@Query(value =
		"select m.university as university, sum(m.point) as point, count(m.university) as personnel "
			+ "from Member m "
			+ "group by m.university "
			+ "order by point desc")
	Page<SchoolRankingDto> findGroupByUniversityOrderByPointDesc(Pageable pageable);
```

위의 메서드는 학교별로 Group by하여 합산 Point로 정렬하는 메서드인데, Group by와 집계 함수 등이 필요하므로 JPQL을 사용했다.

인터페이스의 메서드명과 쿼리에서 가져오는 필드명이 같아야 매핑이 가능하다.

JPQL 쿼리를 보면, sum(m.point) 등의 집계 함수로 가져온 필드의 이름을 인터페이스의 이름과 맞춰주고 있다. m.university 같은 경우 인터페이스와 Entity의 이름이 모두 university라 문제 없을 줄 알았는데 값이 안가져와져서 별칭을 설정해주었다.

만약 엔티티 속성과 일치하지 않는 값을 값을 프로젝션 해야 하는 경우

**Open Projections**을 이용해 가져올 수 있다.

```java
public interface PersonView {
    // Open Projections

    @Value("#{target.firstName + ' ' + target.lastName}")
    String getFullName();
}
```

@Value 어노테이션에 SpEL 표현식을 사용해서 가져올 값을 런타임에 계산하도록 만들 수 있다. 하지만 이 방법에는 단점이 있는데, Spring이 어떤 속성이 사용될지 미리 알지 못하기 때문에 쿼리를 최적화 해줄 수 없다.

## ****Class-Based Projections****

클래스 기반 프로젝션은 프로젝션 인터페이스에서 생성하는 프록시를 사용하는 대신 자체 프로젝션 클래스(DTO)를 정의하는 방법이다.

### PersonalRankingDto Class

프로젝션 클래스의 생성자 매개변수 이름이 엔티티 클래스의 속성과 반드시 일치해야 한다. 또한 equals 및 hashCode 구현을 정의해야 한다고 하는데 없어도 작동은 되는걸로 보아 다른 문제가 있는 것 같다.

```java
@Getter
@AllArgsConstructor
@EqualsAndHashCode // 없어도 작동은 잘 됨..
public class PersonalRankingDto {
	private Long id;
	private String nickname;
	private Long point;
	private String university;
}
```

### MemberRepository

반환 객체를 위에서 생성한 PersonalRankingDto로 지정하면 값이 잘 매핑되어 들어오는 것을 확인할 수 있다.

그러나 클래스 프로젝션은 중첩 프로젝션을 사용할 수 없다.

```java
Page<PersonalRankingDto> findAllByOrderByPointDesc(Pageable pageable);
```

사실 위의 인터페이스 프로젝션의 예시도 클래스 프로젝션이 가능한데 굳이 인터페이스를 생성해서 사용하게 된 이유가 있다.

1. JPQL을 사용하면 Inner Class에는 프로젝션이 안됨 (원래 RankingDto 클래스에 Personal, School Class를 정의했었음)
2. JPQL에서 Class Projection을 사용하려면 아래와 같이 클래스 패키지명을 다 써줘야 해서 보기에 안좋음

```java
@Query(value = "select new inhatc.capstone.baro.ranking.SchoolRankingDto(m.university, sum(m.point) as point, count(m.university) as personnel) from Member m group by m.university")
	Page<SchoolRankingDto> findGroupByUniversityOrderByPointDesc(Pageable pageable);
```

## Dynamic Projections

동적 프로젝션은 Repository에 정의된 하나의 메서드를 Entity, Dto, Inteface 등 여러 객체로 반환받고 싶을 수 있다. 이 때 별도의 메서드를 정의하는 것은 번거롭기 때문에 제네릭을 사용해서 동적 프로젝션을 적용할 수 있다.

아래 예제를 보면 하나의 메서드지만 Class 매개변수를 사용해서 Person, PersonView, PersonDto 모두 하나의 메서드를 사용할 수 있다.

```java
public interface PersonRepository extends Repository<Person, Long> {
    // ...

    <T> T findByLastName(String lastName, Class<T> type);
}

@Test
public void whenUsingDynamicProjections_thenObjectWithRequiredPropertiesIsReturned() {
    Person person = personRepository.findByLastName("Doe", Person.class);
    PersonView personView = personRepository.findByLastName("Doe", PersonView.class);
    PersonDto personDto = personRepository.findByLastName("Doe", PersonDto.class);

    assertThat(person.getFirstName()).isEqualTo("John");
    assertThat(personView.getFirstName()).isEqualTo("John");
    assertThat(personDto.getFirstName()).isEqualTo("John");
}
```

이렇게 Inteface Projection과 Class Projection을 모두 사용해보았는데, 전에도 인터페이스 프로젝션을 사용해본 적은 있지만 이 기능에 대해 자세히 알아보고 쓴 것은 이번이 처음이라 앞으론 자주 사용할 수 있을 것 같다. 

## 참고

[](https://www.baeldung.com/spring-data-jpa-projections)

https://docs.spring.io/spring-data/jpa/docs/current/reference/html/