# equals와 hashCode

Java의 모든 클래스는 Object 클래스를 암시적으로 상속받고 있다. 모든 클래스의 조상인 Object 클래스에서는 모든 클래스가 공통적으로 포함하고 있어야 하는 기능을 제공한다.

그 중 오늘은 equals와 hashCode 메서드에 대해서 알아보려고 한다.

그 전에 객체의 동일성과 동등성에 대한 개념부터 짚고 넘어가야 한다.

# 객체의 동일성과 동등성

### 동일성(Identity)

동일성은 두 객체가 같은 메모리 주소 값을 가지는 경우를 의미한다. 같은 주소 값을 가지고 있기 때문에 두 변수가 모두 같은 객체를 가리킨다.

두 변수의 동일성은 **== 연산자**를 통해 확인할 수 있다. 또한 후술할 equals 메서드의 **기본 구현**도 ==을 통해 이루어지기 때문에 동일성을 비교한다.

### **동등성(Equality)**

동등성은 두 객체가 같은 내용을 가지고 있는 경우를 의미한다. 객체의 주소가 다르더라도 내용이 같으면 두 객체는 동등하다.

**두 객체가 동일하면 동등하지만, 동등하다고 동일한 것은 아니다.**



# equals()

equlas 메서드는 기본적으로 2개의 객체의 참조가 같은지 동일성을 확인하도록 구현되어 있어 == 연산자와 같은 동작을 하게 된다.

```java
//Object 클래스의 기본 equals 구현
public boolean equals(Object obj) {
        return (this == obj);
}
```



객체의 동등성을 비교하기 위해서는 equals 메서드를 재정의 해줘야 한다. 대표적으로 String이나 Integer, Long과 같은 래퍼 클래스에서도 equals 메서드가 오버라이딩되어 있다.

```java
//Long 클래스의 equals
public boolean equals(Object obj) {
        if (obj instanceof Long) {
            return value == ((Long)obj).longValue();
        }
        return false;
    }
```

위의 Long 클래스에서 equals()를 재정의하여 longValue()로 값을 비교하여 동등성을 비교하고 있다.

Member 클래스를 만들어서 직접 equals를 오버라이딩하는 과정을 진행해보도록 하자

Member는 각 email을 고유한 값으로 가진다고 가정하면 같은 email을 가진 멤버는 동등하다고 판단할 수 있다.

```java
public class Member {
	private final String email;
	private final int age;
	private final String name;

	public Member(String email, int age, String name) {
		this.email = email;
		this.age = age;
		this.name = name;
	}
}
```



우선 equals()를 재정의하지 않은 상태에서 결과를 찍어보면, 같은 이메일을 가진 멤버임에도 다른 메모리 주소를 가지고 있기 때문에 equals()의 결과가 false로 출력되는 것을 확인할 수 있다.

```java
public class EqualsAndHashCode {
	public static void main(String[] args) {
		Member member1 = new Member("test@gmail.com", 22, "홍길동");
		Member member2 = new Member("test@gmail.com", 22, "홍길동");
		Member member3 = new Member("test2@gmail.com", 28, "홍길동");

		System.out.println(member1.equals(member2)); //false
	}
}
```



이번에는 Member에 equals를 동등성을 비교하도록 오버라이드 한 상태에서 다시 같은 코드를 실행해보면 같은 객체로 취급된다.

```java
public class Member {
	private final String email;
	private final int age;
	private final String name;

	@Override
	public boolean equals(Object o) {
		if (this == o) // 동일성 검사
			return true;
		if (o == null || getClass() != o.getClass())
			return false;
		Member member = (Member)o;
		return email.equals(member.email);
	}
}

public class EqualsAndHashCode {
	public static void main(String[] args) {
		Member member1 = new Member("test@gmail.com", 22, "홍길동");
		Member member2 = new Member("test@gmail.com", 22, "홍길동");
		Member member3 = new Member("test2@gmail.com", 28, "홍길동");

		System.out.println(member1.equals(member2)); //true
	}
}
```



## hashCode()

hashCode 메서드는 **각 객체의 유일한 정수값을 반환한다.** Object 클래스에서는 heap에 저장된 객체의 메모리 주소를 반환하도록 구현되어 있지만 항상 그런 것은 아니다.

```java
//Object 클래스의 hashCode()
public native int hashCode();
```

Object 클래스의 hashCode는 native로 선언되어 있다. native는 메서드가 JNI(Java Native Interface)라는 native code로 구현되었음을 의미하는데, native는 메서드에만 적용 가능한 제어자로 C나 C++ 등 Java가 아닌 언어로 구현된 부분을 Java에서 이용하고자 할 때 사용된다. 



### Long의 hashCode()

Long에서는 hashCode 메서드를 재정의하여 static 메서드에서 hashCode 값을 계산하도록 구현되어 있다. 이렇게 재정의하면 메모리 주소를 기반으로 생성하던 hashCode와 달리 value 값으로 hashCode가 생성되므로 동등성을 비교할 수 있게 된다.

```java
@Override
    public int hashCode() {
        return Long.hashCode(value);
    }

    public static int hashCode(long value) {
        return (int)(value ^ (value >>> 32));
    }
```





## equals()와 hashCode()를 같이 재정의해야 하는 이유

equals()만 재정의하더라도 객체끼리의 동등성을 확인하는 데에는 문제가 없다. 그러나 hashCode를 재정의 하지 않는 경우 **hash 값을 사용하는 Collection(HashSet, HashMap, HashTable)을 사용할 때 문제가 발생한다.**

```java
public class EqualsAndHashCode {
	public static void main(String[] args) {
		Member member1 = new Member("test@gmail.com", 22, "홍길동");
		Member member2 = new Member("test@gmail.com", 22, "홍길동");
		Member member3 = new Member("test2@gmail.com", 28, "홍길동");

		Set<Member> members = new HashSet<>();
		members.add(member1);
		members.add(member2);
		members.add(member3);
		
		System.out.println(members.size()); // 3
	}
}
```

위 코드에서 member1과 member2는 동등하기 때문에 중복이 허용되지 않는 Set에 삽입했을 경우 size가 2가 되는 것을 기대할 수 있다. 

하지만 **hashCode를 재정의하지 않으면 메모리 주소로 객체를 비교**하기 때문에 같은 값을 가진 객체여도 다른 해시값을 반환하여 Set에 모두 저장되므로 size는 3이 출력된다.



이러한 문제를 해결하기 위해 Member 클래스에 hashCode 메서드도 오버라이드하여 재정의 해야 한다.

```java
public class Member {

	@Override
	public int hashCode() {
		return Objects.hash(email);
	}
}
```

Objects.hash 메서드는 hashCode 메서드를 재정의하기 위해 간편하게 사용할 수 있는 메서드지만 속도가 느리다는 단점이 있다. 성능에 아주 민감하지 않은 대부분의 프로그램은 사용해도 문제 없지만 민감한 경우에는 직접 정의해주는 것이 좋다.



hashCode를 오버라이드 한 이후 위의 코드를 다시 실행하면 HashSet의 Size가 기대했던 대로 2가 출력되는 것을 확인할 수 있다.





## 참고

[https://mangkyu.tistory.com/101](https://mangkyu.tistory.com/101)

[https://tecoble.techcourse.co.kr/post/2020-07-29-equals-and-hashCode/](https://tecoble.techcourse.co.kr/post/2020-07-29-equals-and-hashCode/)

[https://creampuffy.tistory.com/140](https://creampuffy.tistory.com/140)