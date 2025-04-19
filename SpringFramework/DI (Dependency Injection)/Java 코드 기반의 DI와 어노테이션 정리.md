# 1. 자바 코드 기반 DI 설정

### XML 설정 vs Java 설정

| 구분      | 설명                                             |
| --------- | ------------------------------------------------ |
| XML 설정  | <bean> 태그로 DI 설정                            |
| Java 설정 | @Configuration 클래스에서 @Bean 메서드로 DI 구성 |
| 장점      | 타입 안정성, IDE 지원, 복잡한 처리 설정에 유리   |

# 2. JavaConfig 예시

```java
@Configuration
public class JavaConfig{
	@Bean
	public MemberDao memberDao(){
		return new MemberDao();
	}

	@Bean
	public MemberRegisterService memberRegSvc(){
		return nrew MemberRegisterService(memberDao());
	}

	@Bean
	public MemberPrinter printer(){
		return new MemberPrinter();
	}

	@Bean
	public MemberInfoPrinter infoPrinter(){
		MemberInfoPrinter p=new MemberInfoPrinter();
		p.setMemberDao(memberDao());
		p.setPrinter(printer());
		return p;
	}
}
```

- BeanId는 메서드 이름
- 클래스는 리턴 타입
- DI는 직접 메서드 호출로 연결

# 3. @Autowired와 자바 설정

```java
@Component
public class MemberInfoPrinter{
	private MemberDao memDao;

	@Autowired
	public void setMemberDao(MemberDao memberDao){
		this.memDao=memberDao;
	}
}
```

# 4. 자바 설정 + ComponentScan

```java
@Configuration
@ComponentScan(basePackages={"spring"})
public class JavaConfig{
}
```

- @Component, @Service, @Repository 등 자동 탐지 및 등록
- JavaConfig에서 별도 @Bean 등록 없이도 가능

# 5. ComponentScan 제외하기

```java
@Configuration
@ComponentScan(basePackages = {"spring"},
    excludeFilters = {
        @ComponentScan.Filter(type = FilterType.REGEX, pattern = "spring\\..*Dao"),
        @ComponentScan.Filter(type = FilterType.ASSIGNABLE_TYPE, classes = Operand.class)
    })
public class JavaConfig {
}

```

# 6. Java 설정 다중 구성

```java
@Configuration
@Import(ConfigPart2.class)
public class ConfigPart1 {
    ...
}
```

- 여러 JavaConfig 클래스 조합 가능
- @Import로 설정 포함 가능
- 또는 `new AnnotationConfigApplicationContext(Config1.class, Config2.class)` 형태로도 사용 가능

# 7. Java 설정에서 XML 설정 포함하기

```java
@Configuration
@ImportResource("classpath:appCtx.xml")
public class JavaConfig{
}
```

# 8. Bean 의 생명주기 (LifeCycle)

### ✔ 단계

1. **Bean 생성**
2. **의존 주입 (Setter 또는 생성자)**
3. **초기화 메서드 실행**
4. **사용**
5. **소멸 (컨테이너 종료 시)**

### ✔ 방법 1: 인터페이스 사용

```java
public class Client implements InitializingBean, DisposableBean{
	public void afterPropertiesSet(){ ... }
	public void destroy() { ... }
}
```

### ✔ 방법 2: 메서드 이름 직접 지정

```java
@Bean (initMethod="connect", destroyMethod="close")
public Client2 client2(){ ...}
```

```java
<bean id="client2" class="spring.Client2"
	init-method="connect" destroy-method="close">
</bean>
```

# 9. Singleton vs Prototype

- Spring Bean은 기본적으로 Singleton

### prototype 지정

```java
@Bean
@Scope("prototype")
public MemberDao memberDao(){
	return new MemberDao();
}
```

- getBean() 호출할 때마다 새 객체 생성

# 핵심 요약표

| 항목                                  | 설명                                                        |
| ------------------------------------- | ----------------------------------------------------------- |
| JavaConfig                            | @Configuration, @Bean으로 DI 구성@                          |
| ComponentScan                         | @Component 자동 탐지                                        |
| @Autowired                            | 자동 의존성 주입                                            |
| @Import                               | 자바 설정 병합                                              |
| @ImportResource                       | XML 설정 포함                                               |
| Singleton                             | 기본 Bean 스코프                                            |
| Prototype                             | Bean 마다 새 인스턴스 생성                                  |
| LifeCylcle                            | InitializingBean, DisposableBean, initMethod, destroyMethod |
| @Configuration                        | 해당 클래스가 스프링 설정 클래스임을 나타냄                 |
| 내부에 정의된 메서드들이 @Bean을 생성 |

### @Configuration

```java
@Configuration
public class AppConfig{

	@Bean
	public MemberService memberService(){
		return new MemberService();
	}
}
```

⇒ 즉, @Configuration은 “여기 안에 Bean들을 설정할거야” 라고 스프링에게 알려주는 표시이다.

### @Bean

- Configuration 클래스 내부 메서드에 붙여서 해당 메서드의 리턴 객체를 SpringBean으로 등록

```java
@Bean
public MemberDao memberDao(){
	return new MemberDao();
}
```

⇒ @Bean은 이 객체를 스프링 컨테이너에 넣어줘 라는 뜻

### @Component

- 클래스 자체에 붙여서 자동으로 Bean으로 등록되게 함
- ComponentScan이 설정된 패키지 안에서만 동작

```java
@Component
public class MemberDao{
	...
}
```

그리고 JavaConfig에 이렇게 설정해줘야함

```java
@Configuartion
@ComponentScan(basePackages="spring")
public class Appconfig{
}
```

⇒ @Component는 자동 등록, @Bean은 수동 등록!

## `@Bean` vs `@Component` 비교표

| 항목      | `@Bean`                           | `@Component`                          |
| --------- | --------------------------------- | ------------------------------------- |
| 선언 위치 | 메서드                            | 클래스                                |
| 사용 위치 | `@Configuration` 클래스 내부      | ComponentScan 대상 패키지 클래스      |
| 등록 방식 | 수동 등록                         | 자동 등록                             |
| 등록 이름 | 메서드명으로 등록                 | 클래스명(lowerCamelCase)으로 등록     |
| 주 사용처 | 외부 라이브러리, 직접 생성할 객체 | 내 프로젝트의 일반 클래스, Service 등 |

## 언제 어떤 걸 써야 할까?

| 상황                                            | 추천 방식                      |
| ----------------------------------------------- | ------------------------------ |
| 직접 클래스를 new로 생성해야 하는 경우          | `@Bean`                        |
| 우리가 만든 컴포넌트 클래스들 (Service, Dao 등) | `@Component`                   |
| 외부 라이브러리 클래스가 Bean으로 필요할 때     | `@Bean`                        |
| Service/Controller/Repository 구분이 필요할 때  | `@Component` + 세부 어노테이션 |
