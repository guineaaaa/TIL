# 1. Dependencu (의존성) 이란?

객체 A가 객체 B를 사용해야 하면 A는 B에 의존한다고 말한다.
예를 들어 `Customer`클래스가 `Address` 객체를 사용해야 한다면,
`Customer`는 `Address`에 의존한다는 것이다.

```java
class Customer{
    Address address;
}
```

# 2. 의존성 설정 방법

객체 A가 객체 B를 사용하는 방법에는 2가지가 있다.

## 1) 내부에서 직접 생성 (내부 설정)

```java
class Customer{
    Address address;

    public Customer(){
        address=new Address(); // 직접 객체 생성
    }
}
```

이 방법은 Customer 클래스 내부에서 Address 객체를 직접 생성하는 방식이다.
**문제점**: `Customer`클래스가 `Address` 객체를 스스로 생성하기 때문에, `Address`가 변경되면 `Customer`도 수정해야 한다.
즉, 유연성이 부족하다는 말 이다.

## 2) 외부에서 주입 (외부 설정)

```java
public static void main(String[] args){
    Customer cust=new Customer(new Address()); //생성자로 Address 주입
}
```

이 방법은 `main` 함수에서 `Address` 객체를 생성한 후, `Customer` 생성자에 넘겨준다.

이렇게 하면 `Customer`는 `Address`가 어떻게 만들어지는지 몰라도 된다.
➡️ 이것이 바로 의존성 주입의 개념이다

## 3. Spring에서의 DI(의존성 주입)

Spring에서는 객체 간의 의존성을 직접 코드에서 설정하는 것이 아니라 Spring IoC 컨테이너 (ApplicationContext)가 대신 설정해준다.

### DI 적용 전 (기존 방식)

```java
public class HelloApp{
    public static void main(String[] args){
        MessageBean bean=new MessageBean(); //직접 객체 생성
        bean.sayHello("Spring");
    }
}
```

- HelloApp이 MessageBean을 직접 생성
- MessageBean이 변경되면 HelloApp도 변경해야 한다

### DI 적용 후 🌿(Spring의 방식!)

```java
public class HelloApp{
    public static void main(String[] args){
        ApplicationContext ctx=new GenericXmlApplicationContext("applicationContext.xml");
        MessageBean bean=(MessageBean) ctx.getBean("messageBean"); //Spring이 객체 생성
        bean.sayHello("Spring");
    }
}
```

- applicatioContext.xml 설정 파일에서 MessageBean 객체를 생성하고 관리한다.
- HelloApp에서는 ctx.getBean("messageBean")을 호출해서 Spring 컨테이너에서 객체를 받아오기만 하면 된다.

## 4. Spring에서의 DI 방식식

Spring에서 DI를 구현하는 방식은 크게게 2가지가 있다.

### 1) 생성자 주입 (Constructor Injection)

**💡 XML 설정 방식**

```xml
<bean id="messageBean" class="sample3.MessageBean">
    <constructor-arg value="Spring">
</bean>
```

- MessageBean 객체를 만들 떄, 생성자에 "Spring"을 넣어준다.

**💡 Java 코드 (Spring @Configuration 사용)**

```java
public class AppConfig{
    @Bean
    public MessageBean messageBean(){
        return new MessageBean("Spring");
    }
}
```

**💡 @Autowired 어노테이션 방식**

```java
@Component
public class MessageBean{
    private final String greeting;

    @Autowired
    public Messagebean(String greeting){
        this.greeting=greeting;
    }
}
```

- 생성자 주입은 순환 참조 문제를 방지하고, 필드가 final일 경우 변경할 수 없도록 보장해줘서 안정성이 높다.

### 2) 세터 주입 (Setter Injection)

<bean id="messageBean" class="sample3.MessageBean">
    <property name="greeting" value="Hello"/>
</bean>
```
- property 태그를 이용해 MessageBean의 setGretting("Hello") 메서드를 호출해서 값 주입

**💡 Java코드 (Spring @Configuration 사용)**

```java
public class AppConfig{
    @Bean
    public MessageBean messageBean(){
        MessageBean bean=new MessageBean();
        bean.setGreeting("Hello");
        return bean;
    }
}
```

**💡 @Autowired 어노테이션 방식**

```java
@Component
public class MessageBean{
    private String greeting;

    @Autowired
    public void setGreeting(String greeting){
        this.greeting=greeting;
    }
}
```

- 세터 주입 방식은 선택적 의존성 주입이 가능하지만, 객체가 완전히 생성된 후에야 의존성이 설정되기 때문에 불변성이 필요한 경우 적합하지 않다.

### 3) 필드 주입 (Field Injection)

```java
public class MessageBean{
    @Autowired //필드 주입
    private String greeting
}
```

- 필드 주입은 필드에 직접 @Autowired를 붙여서 주입하는 방식
- 하지만 필드 주입은 테스트하기 어렵고, 객체의 상태가 명확하지 않기 때문에 지양하는것이 좋다.

⭐ 결론

1. 생성자 주입을 기본적으로 사용 (불변성 보장, 순환 참조 방지)
2. 세터 주입은 선택적 의존성이 필요한 경우에만 사용
3. 필드 주입은 지양 (테스트 어려움, 명확한 객체 상태 보장 불가)

## 5. **실습 예제**

### **1) Java 코드 (MessageBean.java)**

```java
package sample3;

public class MessageBean {
    private String greeting;

    public void setGreeting(String greeting) {
        this.greeting = greeting;
    }

    public void sayHello(String name) {
        System.out.println(greeting + name);
    }
}

```

### **2) XML 설정 (applicationContext.xml)**

```xml
<beans xmlns="http://www.springframework.org/schema/beans">
    <bean id="messageBean" class="sample3.MessageBean">
        <property name="greeting" value="Hello, " />
    </bean>
</beans>

```

### **3) 실행 코드 (HelloApp.java)**

```java
package sample3;

import org.springframework.context.ApplicationContext;
import org.springframework.context.support.GenericXmlApplicationContext;

public class HelloApp {
    public static void main(String[] args) {
        ApplicationContext ctx = new GenericXmlApplicationContext("applicationContext.xml");

        MessageBean bean = (MessageBean) ctx.getBean("messageBean");
        bean.sayHello("Spring");
    }
}

```

**실행 결과:**

```java
Hello, Spring
```
