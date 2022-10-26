# BCryptPasswordEncoder란?

BCryptPasswordEncoder는 Spring Security 프레임워크에서 제공하는 클래스로 비밀번호를 암호화(해시)하는 데에 사용한다.

해시 함수에는 MD5나 SHA 등의 종류가 있지만 BCrypt는 단순히 입력을 1회 해시시키는 것이 아니라 솔트(salt)를 부여하여 여러번 해싱하므로 더 안전하게 암호를 관리할 수 있다.

BCrypt는 같은 비밀번호를 암호화하더라도 해시 값은 다른 값이 도출된다.

따라서 BCryptPasswordEncoder에서는 사용자가 제출된 비밀번호와 암호화되어 저장된 비밀번호의 일치 여부를 확인하는 메소드가 제공된다.

BCryptPasswordEncoder는 BCrypt의 로그 라운드라고도 하는 강도(strength)를 설정할 수 있는데, strength가 클 수록 암호를 해시하기 위해 더 많은 작업을(기하급수적으로) 수행해야 한다. 기본값은 10이며 4에서 31 사이의 값을 설정할 수 있다.

BCryptPasswordEncoder에서 제공하는 메소드는 부모클래스인 Object의 메소드를 제외하고 총 3개가 있다.

### 1. String encode(CharSequence rawPassword)

패스워드를 암호화해주는 메소드로, 8바이트 이상의 무작위로 생성된 솔트와 결합된 SHA-1 이상의 해시를 적용한다.

java.lang.CharSequence 타입의 패스워드를 매개변수로 입력해주면 암호화 된 패스워드를 String 타입으로 반환해준다. 해시시킬 때 무작위로 생성한 salt가 포함되므로 같은 비밀번호를 인코딩해도 매번 다른 결과값이 반환된다.

### 2. boolean matches(CharSequence rawPassword, String encodedPassword)

제출된 인코딩 되지 않은 암호와 저장소에 있는 인코딩된 암호가 일치하는지 확인한다. 비밀번호가 일치하면 true를 반환하고 일치하지 않으면 false를 반환한다. 저장된 비밀번호 자체는 절대로 디코딩되지 않는다.

encode 메소드로 인코딩 시킬 때마다 같은 암호도 매번 다른 결과가 나오는 특성 때문에 로그인 등에서 사용자가 입력한 암호가 일치하는지 확인할 때 이 메소드를 사용해야 한다.

### 3. boolean upgradeEncoding(String encodedPassword)

더 나은 보안을 위해 인코딩된 암호를 다시 인코딩해야 하는 경우 true를 반환하고, 그렇지 않으면 false를 반환한다. 기본 구현은 항상 false를 반환한다.

해당 메소드는 기본적으로 항상 false를 반환하므로 필요하다면 오버라이딩 하여 encode된 암호의 안전성을 체크하는 로직을 구현해서 사용하면 될 것 같다.

### 구현 방법

### 1. build.gradle에 spring-security 추가

```groovy
dependencies {
	implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
	**implementation 'org.springframework.boot:spring-boot-starter-security'**
	implementation 'org.springframework.boot:spring-boot-starter-web'
	implementation 'org.springframework.boot:spring-boot-starter-websocket'
	implementation 'org.springframework.boot:spring-boot-starter-data-redis'
```

build.gradle에 **'org.springframework.boot:spring-boot-starter-security'** 를 ****implementation 해준다.

### 2. SecurityConfig에 BCryptPasswordEncdoer Bean으로 등록

```java
@Configuration
@EnableWebSecurity
@RequiredArgsConstructor
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
}
```

SecurityConfig에서 BCryptPasswordEncoder를 Bean으로 등록하여 스프링 컨테이너에 등록한다.

### 3. BCryptPasswordEncoder 사용

```groovy
@Service
@RequiredArgsConstructor
public class TestService {
//Bean으로 등록된 BCryptPasswordEncoder 의존성 주입
	private final PasswordEncoder passwordEncoder;

	public void register(String id, String password) {
			//사용자가 입력한 암호를 encode하여 저장
			String encodedPassword = passwordEncoder.encode(password);
			Member member = new Member(id, encodedPassword);

			TestRepository.save(member);
	}	

	public boolean login(String id, String password) {
			Member member = TestRepository.find(id);
			if(passwordEncoder.matches(password, member.getPassword()) {
					return true; // 입력한 비밀번호와 저장소의 비밀번호가 일치
			} else {
					return false;
			}
	}
}
```

간단하게 encode와 matches의 사용방법을 설명하기 위해 예시 코드를 만들었다.

register 메소드에서는 사용자가 가입할 때 입력한 패스워드를 인코딩하여 저장소에 저장하는 것을 볼 수 있다.

그리고 login 메소드에서는 사용자가 로그인 할 때 입력한 패스워드와 회원가입 때 인코딩되어 저장소에 저장된 패스워드가 일치하는지를 matches 메소드를 통해 체크한다.

이처럼 Spring Security에서 제공하는 BCryptPasswordEncoder를 사용하면, 사용자의 비밀번호를 안전하게 관리할 수 있다.