# DI 설정 개선하기

**수동 DI(@Bean)-> @Autowired -> @ComponentScan 적용**
JavaConfig.java 파일로 기존의 ApplicationContext.xml로 의존성을 주입했던 방식을 개선할 수 있다.

# 수동 DI

```java
@Configuration
public class JavaConfig {
	/**
	 * <bean id="operand1" class="com.example.demo.Operand" scope="prototype">
       		<property name="value" value="30"/>
       </bean>

        <bean id="operand2" class="com.example.demo.Operand" scope="prototype">
       		<property name="value" value="20"/>
       </bean>
	 */

	// @Bean
	@Bean(name = "operand1")
	@Scope("prototype")
	public Operand op1() {
		Operand op1=new Operand();
		op1.setValue(30);
		return op1;
	}

	// @Bean
	@Bean(name = "operand2")
	@Scope("prototype")
	public Operand op2() {
		Operand op2=new Operand();
		op2.setValue(20);
		return op2;
	}

    @Bean
    public OperatorBean operatorBean() {
    	// return new PlusOp(); -> @Autowired를 사용하는 방법
		OperatorBean op=new PlusOp();
		op.setOperand1(op1()); // @Autowired 대신 직접 호출
		op.setOperand2(op2());

		return op;
	}

}
```

- **핵심 특징**:
  - `@Bean`을 사용하여 `Operand` 객체를 직접 생성하고 `PlusOp`에 주입.
  - `operatorBean()` 메서드에서 `PlusOp` 객체를 수동으로 생성 후 `setOperand1()`, `setOperand2()`를 호출하여 의존성을 주입.
  - **`@Autowired` 없이 수동으로 객체를 연결하는 방식**.
- **장점**:
  - `@Autowired` 없이 직접 객체를 주입하므로 Spring이 자동 주입을 못하는 경우에도 동작 가능.
- **단점**:
  - `PlusOp` 객체를 `new`로 직접 생성하므로, Spring의 관리 대상이 아님.
  - `Operand` 객체도 `op1()` 및 `op2()` 메서드를 직접 호출하여 주입하므로 스프링의 DI 기능을 100% 활용하지 못함.


# `@Autowired` 사용 DI

```java
@Configuration
public class JavaConfig2 {
	// 이름 같은것을 연결시켜준다
	@Bean(name = "operand1")
	@Scope("prototype")
	public Operand op1() {
		Operand op1=new Operand();
		op1.setValue(30);
		return op1;
	}

	// @Bean
	@Bean(name = "operand2")
	@Scope("prototype")
	public Operand op2() {
		Operand op2=new Operand();
		op2.setValue(20);
		return op2;
	}

    @Bean
    public OperatorBean operatorBean() {
    	return new PlusOp(); // @Autowired를 사용하는 방법

	}

}
```

- **핵심 특징**:
  - `@Bean`을 사용하여 `Operand` 객체를 생성.
  - `operatorBean()` 메서드에서 `new PlusOp();` 만 반환하여 `OperatorBean`의 구현체를 Spring이 관리하도록 함.
  - **`@Autowired`를 활용하는 방식**.
- **장점**:
  - `PlusOp` 객체를 직접 조작하지 않고, `@Autowired`를 활용하여 Spring이 알아서 주입하도록 함.
  - `OperatorBean`의 구체적 구현체(`PlusOp`)를 쉽게 변경 가능.
- **단점**:
  - `@Autowired`에 의존하므로, `PlusOp`가 `@Component`로 선언되지 않으면 주입이 실패할 수 있음.
  - 명확한 `@Qualifier` 지정이 없으면 여러 개의 `OperatorBean`이 있을 때 충돌 가능.


# `@ComponentScan` 사용 DI

```java
@Configuration
@ComponentScan(basePackages= {"ex5_5"})
public class JavaConfig3 {
    @Bean
    public OperatorBean operatorBean(PlusOp op) {
    	return op;
	}

}
```

- **핵심 특징**:
  - `@ComponentScan`을 사용하여 `ex5_5` 패키지의 모든 `@Component`가 붙은 클래스를 자동으로 스캔.
  - `Operand`를 수동으로 `@Bean`으로 등록하지 않고, `@Component`로 선언된 객체들을 자동으로 Spring이 관리.
  - `operatorBean()`에서 `PlusOp`를 인자로 받아 반환하는 방식.
- **장점**:
  - `@ComponentScan`을 사용하여 **수동 설정 없이 자동으로 의존성 주입이 가능**.
  - `operatorBean()`에서 `PlusOp`를 매개변수로 받아서 주입하므로, 스프링이 자동으로 `PlusOp`를 찾아서 제공.
  - 코드가 가장 간결하며, 유지보수성이 좋음.
- **단점**:
  - `ex5_5` 패키지 내 `@Component`가 없는 경우 객체를 생성하지 못함.
  - 자동 주입 방식이므로 명확한 제어가 필요한 경우에는 다소 불편할 수 있음.

# 결론

| 설정 파일       | `@Bean` 방식                                   | `@Autowired` 사용 | `@ComponentScan` 사용 | 주요 특징                             |
| --------------- | ---------------------------------------------- | ----------------- | --------------------- | ------------------------------------- |
| **JavaConfig**  | `@Bean`을 사용해서 직접 수동 객체 생성 및 주입 | ❌                | ❌                    | 직접 객체를 생성하고 setter로 주입    |
| **JavaConfig2** | `@Bean`으로 등록                               | ✅                | ❌                    | `@Autowired`를 활용한 Spring DI       |
| **JavaConfig3** | `@Bean` + `@ComponentScan`                     | ✅                | ✅                    | 가장 자동화된 방식, 유지보수성이 좋음 |

- **Spring의 DI 기능을 최대한 활용하려면 `JavaConfig3` 방식이 가장 추천됨.**
  (`@ComponentScan`과 `@Autowired`를 조합하여 설정을 최소화)
