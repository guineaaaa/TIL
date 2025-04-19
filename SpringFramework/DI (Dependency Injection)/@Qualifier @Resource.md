# ✅ `@Qualifier` vs `@Resource`

스프링에서 `@Autowired`를 사용할 때 **동일한 타입의 Bean이 여러 개** 존재하면 **자동 주입이 불가능해지고 에러가 발생한다.**

이때 어떤 Bean을 주입할지 명시적으로 지정해주는 것이 바로
👉 `@Qualifier`와 `@Resource` 이다.

# 1. `@Qualifier` 어노테이션

### ✔️ 정의

- `@Autowired`와 함께 사용
- Bean의 이름 (id)를 기준으로 어떤 Bean을 주입할지 지정
    - @Autowired는 타입 기준 자동 주입을 수행한다.
    - 그런데 동일한 타입의 Bean이 2개 이상 등록되어있으면!
    
    ```java
    <bean id="print1" class="spring.MemberPrinter"/>
    <bean id="print2" class="spring.MemberPrinter"/>
    ```
    
    ```java
    @Autowired
    private MemberPrinter printer; // 에러 발생; 어떤 걸 넣을 지 모름
    ```
    
    → 이때 명확히 어떤 빈을 넣을지 지정해주는것이 @Qualifier 어노테이션
    

## 사용법

### XML

```java
<bean id="print1" class="spring.MemberPrinter">
	<qualifier value="sysout"/>
</bean>
<bean id="print2" class="spring.MemberPrinter">
```

### 자바 코드

```java
@Autowired
@Qualifier("sysout")
public void setPrinter(MemberPrinter printer){
	this.printer=printer;
}
```

- 동일한 타입의 Bean 중 하나를 명확히 지정
- @Autowired와 함께 사용한다.
- Bean 의 이름 (또는 qualifier이름)을 기준으로 선택한다.
- 위치는 필드, 세터, 생성자 파라미터 등

# 2. `@Resource` 어노테이션

### ✔️ 정의

- Java 표준 어노테이션
- @Autowired + @Qualifier를 하나의 어노테이션으로 대체 가능
- Bean 이름 기준으로 자동 주입

### 사용법

```java
<bean id="memberDao" class="spring.MemberDao"/>
```

```java
import javax.annotation.Resource;

public class MemberRegisterService{

	@Resource(name="memberDao")
	private MemberDao memberDao; //bean id가 memberDao인 객체 주입
}
```

- memberDao와 bean id와 **이름이 일치한 객체가 자동 주입됨**
- @Autowired는 타입 우선이지만, @Resource는 이름 우선 매칭

- 필드, 세터 메서드에 사용가능하다 생성자에서는 사용 불가능
- name속성을 지정 안하면 변수명으로 매칭 시도
1. name으로 매칭 → 2. 타입으로 매칭 → 3. 변수명으로 매

### 추가

- `@Autowired(required = false)`
    
    → 주입 대상이 없어도 에러 발생하지 않게 함 (널 허용)
    
- 자동 주입 우선순위
    
    ```
    1. 같은 타입의 Bean이 하나뿐이면 주입
    2. 여러 개면 → @Qualifier로 지정
    3. 그래도 없으면 → 변수명과 동일한 Bean id 찾음
    ```