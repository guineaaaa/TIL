# @Autowired

- **스프링 컨테이너가 자동으로 해당 타입의 Bean을 찾아서 주입해주는 어노테이션이다.**
- **즉, 개발자가 직접 new 키워드로 객체를 만들지 않아도 스프링이 자동으로 의존 객체를 연결 (DI: Dependency Injection) 해준다.**

## ✅ Spring DI 방식 3가지 (자바 코드 기반)

| 구분           | 설명                                                               |
| -------------- | ------------------------------------------------------------------ |
| 1️⃣ 필드 주입   | `@Autowired`를 필드에 바로 붙이는 방식 (권장 ❌)                   |
| 2️⃣ 세터 주입   | `@Autowired`를 세터 메서드에 붙여서 주입하는 방식                  |
| 3️⃣ 생성자 주입 | 생성자에 `@Autowired`를 붙여서 생성 시점에 주입하는 방식 (⭐ 추천) |

# 1. 필드 주입 (Field Injection)

### ✔ 특징

- 가장 간편하지만 단점 많다.
- 테스트 어려움
- 의존관계 명확하지 않음

### ✅ 예시

```java
@Component
public class MemberService{
	@Autowired
	private MemberDao memberDao;
	...
}
```

간편하지만 객체가 어떻게 생성됐는지 알기 어려워서 테스트와 유지보수에 불

# 2. 세터 주입 (Setter Injection) #선택적사용

### ✔ 특징

- **세터 메서드를 통해서 의존 객체를 주입**
- **의존성이 선택적(optional)일 때 사용**
- **객체 생성 이후 주입되므로 주입 누락 가능성** 있음
- 테스트 용이성은 낮음 (Mock 주입 어려움)

### ✅ 예시

```java
@Component
public class MemberService{
	private MemberDao memberDao;

	@Autowired
	public void setMemberDao(MemberDao memberDao){
		this.memberDao=memberDao;
	}

	public void doSomething(){
		memberDao.save();
	}
}
```

- memberDao가 없어도 일단 객체 생성됨

# 3. 생성자 주입 (Constructor Injection)

### ✔ 특징

- **생성자를 통해 의존 객체 주입**
- **의존성이 불변이고, 반드시 필요한 경우 사용**
- **객체 생성 시점에 주입이 완료되어 Null이 될 가능성이 없다.**
- 테스트와 유지 보수에 유리 (final 필드 사용)

### ✅ 예시

```java
@Component
public class MemberService{
	private final MemberDao memberDao;

	@Autowired
	public MemberService(MemberDao memberDao){
		this.memberDao=memberDao;
	}

	public void doSomething(){
		memberDao.save();
	}
}
```

- `memberDao`는 반드시 필요하므로 생성자에서 주입
- `memberDao`는 다른 클래스에서 `@Component`로 Bean등록이 되어있어야 함
- 스프링이 자동으로 `MemberDao` 객체를 찾아서 생성자에 넣어줌
- final로 선언 가능해서 주입이 안되면 컴파일러가 알려줌

---

# XML 기반 DI 설정 방법 - Setter 주입

### 예시 클래스

```java
public class MemberDao{
	public void save(){
		System.out.println("saving member");
	}
}
public class MemberService{
	private MemberDao memberDao;

	public void setMemberDao(MemberDao memberDao){
		this.memberDao=memberDao;
	}
}
```

### XML 설정 (applicationContext.xml)

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="
           http://www.springframework.org/schema/beans
           http://www.springframework.org/schema/beans/spring-beans.xsd">

           <bean id="memberDao" class="spring.MemberDao"/>

           <bean id="memberService" class="spring.MemberService">
	           <property name="memberDao" ref="memberDao" />
	          </bean>
</beans>
```

- property name=”memberDao” ——→ 세터 메서드 (setMemberDao)를 자동 호출,
  - MemberService 클래스에서 setMemberDao()라는 세터 메서드를 찾음 (set+memberDao의 이름의 메서드를 자동으로 호출)
- Sptring 컨테이너에 이미 등록된 id=”memberDao”인 Bean을 찾아서 setMemberDao(그 객체)로 주입하라는 뜻이다.

### main

```java
ApplicationContext ctx=new GenericXmlApplicationContext("classpath:applicatoinContext.xml");

MemberService service=ctx.getBean("memberService", Memberservice.class");
service.doSomething();
```

# XML 기반 DI - 생성자 주입

- <constructor-arg> 태그를 이용해서 생성자 매개변수에 값을 주입하는 방법
- 이때 매개변수의 순서, 타입 또는 인덱스를 기준으로 어떤 값을 주입할 지 지정할 수 있다.

### 예제 클래스

```java
public class MemberService{
	private MemberDao memberDao;

	// 생성자 주입 용 생성자
	public MemberService(MemberDao memberDao){
		this.memberDao=memberDao;
	}

	public void doSomething(){
		memberDao.save();
	}
}
```

### XML 설정 예시 (applicationContext.xml)

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="
           http://www.springframework.org/schema/beans
           http://www.springframework.org/schema/beans/spring-beans.xsd">

           <bean id="memberDao" class="spring.MemberDao"/>

           <bean id="memberService" class="spring.MemberService">
	           <constructor-arg ref="memberDao"/>
	          </bean>
</bean>
```

- <constructor-arg>는 생성자 파라미터에 주입할 값을 지정
- ref=”memberDao”는 id=”memberDao”인 Bean을 생성자 매개 변수로 넘긴다.
- 생성자 파라미터 순서와 constructor-arg의 순서가 일치해야 한다.

### main

```java
ApplicationContext ctx=new GenericXmlApplicationContext("classpath:applicationContext.xml");

MemberService service=ctx.getBean("memberService");
service.doSomething();
```
