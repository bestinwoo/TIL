# **Java Reflection**

## **리플렉션 이란?**

리플렉션은 Java의 기능으로 구체적인 클래스 타입을 알지 못하더라도 그 클래스의 메서드, 필드, 타입 등에 접근할 수 있도록 도와주는 API이다. 리플렉션은 직접 접근할 수 없는 private 인스턴스 변수와 메서드에 접근할 수 있다.

## **리플렉션의 사용처**

리플렉션은 클래스를 런타임에 동적으로 사용해야 할 때 필요하다. 컴파일 시점에는 어떤 클래스를 사용해야 할 지 모르는 경우 런타임 시점에 클래스를 가져와서 실행해야 할 때 사용한다. 대표적으로 사용되는 곳으로는 IntelliJ의 자동완성, Hibernate, Spring Framework의 BeanFactory 등의 프레임워크나 라이브러리에서 주로 사용되고 있다.

## **리플렉션의 원리**

Java에서는 모든 .class 파일 하나 당 java.lang.Class 객체가 하나씩 생성된다. Class는 모든 .class의 정보를 가지고 있으며 .class 파일에 같이 저장된다. 모든 .class들은 이 클래스를 최초로 사용하는 시점에서 동적으로 ClassLoader를 통해 JVM에 로드된다.

최초로 사용하는 시점은 해당 .class에 static을 최초로 사용할 때를 의미한다. 생성자도 static 메소드이기 때문에 new를 하면 로드되는데 이를 동적 로딩이라 한다. 이렇게 .class의 정보와 Class 객체는 JVM Run Time Data Area의 Method Area에 저장된다. 이러한 정보들을 java.lang.reflect에서 접근할 수 있게 도와준다. 

[https://www.holaxprogramming.com/2013/07/16/java-jvm-runtime-data-area/](https://www.holaxprogramming.com/2013/07/16/java-jvm-runtime-data-area/)

## **리플렉션 기능**

java.lang.reflect 패키지에는 생성자, 메서드, 필드 등의 클래스들이 있고, 생성자를 통해 새로운 객체를 생성할 수 있다.

- getter, setter 메서드를 통해 필드의 값을 읽고 수정할 수 있다.
- invoke 메서드를 통해 메서드를 호출할 수 있다.
- parameter, return type, modifier 등 클래스의 관련된 모든 정보를 알고 조작할 수 있다.
- setAccessible을 통해 private 필드, 메서드에 접근, 조작이 가능하다.

## 리플렉션 예제

### Person Class 생성

```java
public class Person {
	private String name;
	protected int age;
	public int weight;

	public Person() {
	}

	public Person(String name, int age, int weight) {
		this.name = name;
		this.age = age;
		this.weight = weight;
	}

	public void speak(String message) {
		System.out.println(message);
	}

	private void speakName() {
		System.out.println("제 이름은 " + name + "입니다.");
	}
}
```

### Class 객체 가져오기

클래스 타입의 객체를 가져오기 위한 방법으로는 3가지 방법이 있다.

- 클래스.class로 클래스를 가져온다.
- 인스턴스의 getClass() 메서드를 통해 가져온다.
- Class.forName(”패키지명.클래스명”)으로 가져온다.

```java
public class Main {
	public static void main(String[] args) throws Exception {
		//1. Person.class를 통해 클래스를 가져오는 방법
		Class<Person> personClass = Person.class;
		System.out.println(personClass.hashCode());
		//2. Person 클래스의 인스턴스를 통해 클래스를 가져오는 방법
		Person person = new Person();
		Class<? extends Person> personClass2 = person.getClass();
		System.out.println(personClass2.hashCode());
		//3. Class.forName을 통해 (패키지명).클래스명으로 클래스를 가져오는 방법
		Class<?> personClass3 = Class.forName("example.Person");
		System.out.println(personClass3.hashCode());
	}
}
```

**위 코드를 실행해보면 모두 같은 hashCode를 가지고 있는 같은 객체임을 확인할 수 있다.**

![Untitled (1)](https://user-images.githubusercontent.com/33615669/202269317-4c6c7812-6026-41cb-a813-c0bcbffe2d2b.png)

### 생성자 찾기

이제 Class를 가져왔기 때문에 해당 클래스의 생성자를 가져와 인스턴스를 생성할 수 있다.

해당 예제에서는 public 생성자만 가져오는 getConstructor 메서드를 사용했는데, 만약 private 등의 모든 생성자를 가져오고 싶다면 `personClass.getDeclaredConstructor() 메서드를 사용하면 된다.`

```java
		Class<Person> personClass = Person.class;
		//Person의 모든 생성자 출력
		Arrays.stream(personClass.getConstructors()).forEach(System.out::println);

		//Person의 기본 생성자를 통한 인스턴스 생성
		Constructor<Person> constructor = personClass.getConstructor();
		Person person = constructor.newInstance();
		System.out.println(person.weight);

		//Person의 다른 생성자를 통한 인스턴스 생성
		Constructor<Person> allArgsConstructor = personClass.getConstructor(String.class, int.class, int.class);
		Person person1 = allArgsConstructor.newInstance("김철수", 20, 82);
		System.out.println(person1.weight);
```

### 필드와 메서드 접근 및 호출

```java
Person person = new Person("김철수", 20, 82);
		Class<? extends Person> personClass = person.getClass();

		//필드 접근
		Field[] fields = personClass.getDeclaredFields();
		for (Field field : fields) {
			field.setAccessible(true); // public 외 접근제어자에 접근 가능하도록 설정
			System.out.println(field.get(person));
		}
		//person의 weight을 90으로 수정
		fields[2].set(person, 90);
		System.out.println(person.weight);

		//Person의 모든 메서드 가져오기
		Method[] methods = personClass.getDeclaredMethods();
		for (Method method : methods) {
			method.setAccessible(true);
			System.out.println(method.getName());
		}

		//메서드 호출
		Method speak = personClass.getDeclaredMethod("speak", String.class);
		speak.invoke(person, "이것이 리플렉션");

		//private 메서드 호출
		Method speakName = personClass.getDeclaredMethod("speakName");
		speakName.setAccessible(true);
		speakName.invoke(person);

//실행 결과
김철수
20
82
90
speak
speakName
이것이 리플렉션
제 이름은 김철수입니다.

```

getDeclaredFields()를 통해 클래스의 인스턴스 변수를 모두 가져올 수 있다. get()을 통해 필드의 값을 전달받을 수 있고, set()을 통해 필드 값을 수정할 수 있다.

메서드도 같은 방식으로 접근이 가능하다. 이 때, private 필드나 메서드의 경우에는 접근이 가능하도록 setAccessible(true)로 설정해주어야 한다. 메서드를 호출 할 때는 invoke() 메서드를 통해 리플렉션을 통해 얻어온 메서드를 호출할 수 있다.

### 리플렉션의 장점과 단점

**장점**

- 런타임 시점에 클래스의 인스턴스를 생성하고, 접근 제어자와 관계없이 필드와 메서드에 접근하여 작업을 수행할 수 있는 유연성을 가진다.

**단점**

- 컴파일 타임이 아닌 런타임에 동적으로 타입을 분석하고 정보를 가져오기 때문에 JVM을 최적화할 수 없어 성능 오버헤드가 발생한다.
- 직접 접근할 수 없는 private 인스턴스 변수와 메서드에 접근하기 때문에 추상화가 깨지고 캡슐화를 저해한다.
- 런타임 시점에 인스턴스를 생성하므로 구체적인 동작 흐름을 파악하기 어렵다.

## **참고**

[https://tecoble.techcourse.co.kr/post/2020-07-16-reflection-api/](https://tecoble.techcourse.co.kr/post/2020-07-16-reflection-api/) 

[https://transferhwang.tistory.com/150](https://transferhwang.tistory.com/150)

[https://steady-coding.tistory.com/609](https://steady-coding.tistory.com/609)