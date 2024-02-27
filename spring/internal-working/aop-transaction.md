# AOP, Transaction

> Spring AOP, 선언적 Transaction의 개념에 대해 간단하게 설명하고, 내부 동작에 대해 정리한 글입니다.

## 📚 AOP란

**Aspect Oriented Programming** : 관점 지향 프로그래밍

&#x20;공통된 부가 기능을 하나의 모듈로 분리하여 횡단 관심사를 처리하는 프로그래밍 패러다임이다.



### 1. 개념

* 핵심 기능과 부가 기능을 분리해 **`횡단 관심사`**를 처리한다.
* 부가 기능을 한 곳에서 관리해 하나의 모듈로 관리한다.
* 하나의 모듈이 **`Aspect(관점)`**이 된다.



### 2. 사용 예시

* 부가 기능을 Aspect로 모듈화 해서 핵심 기능에 별도의 코드 침투 없이 적용
* 모든 패키지에 실행시간, 로그, 모니터링 적용
* 동기화나 오류 검사 등 기존 코드에 기능 추가



### 3. AspectJ

* AOP의 대표적인 구현체이다.
* Spring AOP보다 다양한 문법을 지원한다.



### 4. Spring AOP

* Spring이 제공하는 AOP이다.
* AspectJ의 문법, 인터페이스만을 따른다.
* Spring AOP는 런타임 시점에 **`프록시`**를 이용해 AOP를 적용한다.
* 단, 메서드에만 적용 가능하다.
* **어드바이스** + **포인트컷**을 모듈화한 @Aspect이용한다.

```java
@Aspect
public class AspectSample {

	@Around("execution(* *(..))")
	public Object doSomething(ProceedingJoinPoint joinPoint) throws Throwable {
	    	// doSomething before origin method proceed
		return joinPoint.proceed();
	}
}
```



Spring AOP는 위에서 언급했듯이, 런타임 시점에 프록시를 이용해 AOP를 적용한다.

동적으로 객체의 정보를 가져와 프록시를 적용하기 위해 **`JDK Dynamic Proxy`** 혹은 **`CGLIB`**을 이용한다.



두 방식의 차이점에 대해서 보기 전에 Reflection에 대해 알고 가야한다.





## 1️⃣ Reflection API

**`Reflection`**은 런타임 시점에 클래스나 메서드의 메타 정보를 런타임에 동적으로 획득하고 바인딩하는 API이다.

Spring AOP는 런타임 시점에 동적으로 프록시 객체를 생성해 AOP를 적용한다.

런타임 시점에 클래스 정보와 메서드 정보를 가져와 프록시 객체를 생성해야 하므로 **`Reflection API`**를 사용해야 한다.



앞으로 나올 JDK Dynamic Proxy와 CGLIB는 모두 런타임 시점에 프록시 객체를 만드는 라이브러리 이므로 이 **`Reflection API`**를 이용한다.



## 2️⃣ JDK Dynamic Proxy

개발자가 직접 프록시 클래스를 만들지 않아도 런타임 시점에 동적으로 대신 만들어 준다.

인터페이스를 기반으로 프록시를 동적으로 만들어준다. 따라서 **`Interface`**가 필수이며 구현체로는 만들어주지 않는다.



## 3️⃣ CGLIB

[Code Generator Library](https://github.com/cglib/cglib)

JDK Dynamic Proxy와는 다르게 **`Concrete Class`**를 기반으로 프록시를 동적으로 만들어준다.&#x20;

기존에는 외부 라이브러리였지만 Spring이 내부 소스 코드에 포함했으며 최신 Spring, Spring Boot는 CGLIB를 기본으로 사용한다. ~~그런데 CGLIB에서는 ByteBuddy와 같은 것으로 마이그레이션 하는 것을 추천하고 있다..~~

Spring은 **`ProxyFactory`**를 통해 CGLIB을 편하게 사용하도록 도와준다.





## 🌱 AopAutoConfiguration

Spring Boot AOP도 마찬가지로 자동 설정을 지원한다.

AOP관련 의존성을 추가했다면 아래와 같이 AOP 라이브러리 패키지가 추가된다.

<figure><img src="../../.gitbook/assets/image (11).png" alt=""><figcaption></figcaption></figure>

Spring Boot가 자동 주입해주는 AOP 관련 Bean들을 살펴보겠다.



Spring Boot의 Auto Configure 설정 중 **`AopAutoConfiguration`**에는 두 개의 설정 클래스가 존재한다.

**`AutoConfiguration`**에 대한 내용은 \[[링크](01\_-springbootapplication\_auto\_configure\_.md)]를 참고하면 된다.

<figure><img src="../../.gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>

### 1. AspectJAutoProxyConfiguration

**`Advice`** 클래스가 클래스 패스에 존재할 때만 실행되도록 설정되어 있다. AspectJ를 이용한 AOP 프록시 설정을 담당한다.



**`JDK Dynamic Proxy`**와 **`CGLIB Bean`**이 모두 등록되지만 havingValue Option을 통해 **`CGLIB`**이 default로 사용되는 것을 확인할 수 있다.



### 2. ClassProxyConfiguration

`org.aspectj.weaver.Advice` 클래스가 클래스 패스에 존재하지 않을 때 실행되도록 설정되어 있다.&#x20;

&#x20;이 경우, **`BeanFactoryPostProccessor`**라는 후처리기가 Bean으로 등록된다.



> 위에서 알 수 있듯이, Spring Boot 3.2.x기준으로 **`CGLIB`**가 기본 프록시 생성 전략으로 사용되는 것을 확인할 수 있다.



## 💻 Proxy Factory

스프링은 동적 프록시를 통합해서 편리하게 만들어주는 **프록시 팩토리 기능**을 제공한다.

앞서 Spring은 **`ProxyFactory`**를 통해 CGLIB을 편하게 사용하도록 도와준다고 했는데 어떻게 사용되는지  **`ProxyFactory`**의 상속 구조부터 살펴보겠다.



### 1. 상속구조

<figure><img src="../../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

* Proxy를 생성하는데 필요한 설정값을 가진 superclass인 **ProxyConfig**
* AOP에 대한 설정 정보를 담고 있는 **Advised interface**

ProxyFactory에서 위에 대한 것과 Proxy의 생명주기 관리에 대해 구현하고 있다.&#x20;



### 2. ProxyFactory 동작 과정

스프링은 동적 프록시를 통합하여 생성해 주는 프록시 팩토리를 기능을 제공한다.&#x20;

이전에는 인터페이스의 유무에 따라 JDK 동적 프록시나 CGLIB 을 선택해야 했다면, 최신 Spring 버전을 이용한다면 ProxyFactory 하나로 편리하게 동적 프록시를 생성할 수 있다.

<figure><img src="../../.gitbook/assets/image (9).png" alt=""><figcaption></figcaption></figure>



스프링이 제공하는 빈 후처리기의 구현체 중 하나인 **`DefaultAdvisorAutoProxyCreator`**같은 경우 어드바이저를 통해 자동으로 프록시를 생성하는 후처리기이다.

<figure><img src="../../.gitbook/assets/image (8).png" alt=""><figcaption></figcaption></figure>

자동 프록시 생성 빈 후처리기는 Bean의 일부를 프록시를 통해 포장하고, 프록시를 대신 Bean으로 등록하는 역할을 한다.



아래에 프록시 객체 생성 및 반환 과정을 요약했다.

1. 빈으로 등록된 모든 Advisor의 포인트컷을 이용해 프록시 적용 대상 Bean을 찾는다.
2. 적용 대상일 경우 프록시 생성기(default CGLIB)를 통해 현재 빈의 프록시 객체를 만들게 하고 어드바이저를 연결해 로직을 추가한다.
3. 빈 후처리기는 생성된 프록시를 컨테이너의 Bean 대신 프록시 객체를 컨테이너에게 돌려준다.





## 🔍 Transaction

Spring에서는 트랜잭션 처리도 AOP를 이용한다.&#x20;

우선 Transaction을 적용하는 두 가지 방법이 존재한다.

### 1. 프로그래밍을 통한 트랜잭션 관리

프로그래밍을 통해 트랜잭션을 수동으로 관리하는 방법이다.

```
transactionManager.begin();
transactionManager.commit();
transactionManager.rollback();
```

소스 코드 내에 트랜잭션을 제어하는 코드도 포함되기 때문에, 버그가 발생할 여지가 있다.



참고로, DB Access API별 TxManager는 아래와 같다.

<table><thead><tr><th width="172">DB Access API</th><th>Transaction Manager</th></tr></thead><tbody><tr><td>JDBC</td><td> org.springframework.jdbc.datasource.DataSourceTransactionManager</td></tr><tr><td>JDO</td><td>org.springframework.orm.jdo.JdoTransactionManager</td></tr><tr><td>JTA</td><td>org.springframework.transaction.jta.JtaTransactionManager</td></tr><tr><td>Hibernate</td><td>org.springframework.orm.hibernate.HibernateTransactionManager</td></tr></tbody></table>



### 2. 선언적 트랜잭션 관리

메서드 혹은 클래스에 **`@Transactional`**어노테이션을 부여하는 것으로 구현된다.

```
public class Service {

        @Transactional
        public void create(Dto dto) {
        }
      
}
```

소스 코드 내에 트랜잭션 관련 코드가 침투하지 않으므로 자주 사용하게 된다.



메소드나 클래스에 **`@Transactional`** 어노테이션을 부여하는 것만으로 트랜잭션의 시작, 커밋, 롤백은 자동적으로 행해진다. 다만, **Unchecked 예외(RuntimeException 및 그 서브 클래스)**가 발생했을 경우는 롤백되지만, **Checked 예외(Exception 및 그 서브 클래스로 RuntimeException의 서브 클래스가 아닌 것)**가 발생했을 경우는 롤백되지 않고 커밋된다.



## ✚ AOP + Transcation

선언적 트랜잭션을 이용할 경우 Spring은 AOP를 이용해 프록시 객체를 통해 트랜잭션을 적용한다.



우선, **`ProxyTransactionManagementConfiguration`**은 선언적 트랜잭션을 통해 어노테이션 기반으로 트랜잭션을 하도록 도화주는 설정 클래스다. Prfix에서 확인 가능하듯이 Proxy 기반으로 트랜잭션을 적용함을 알 수 있다.

<figure><img src="../../.gitbook/assets/image (25).png" alt=""><figcaption></figcaption></figure>

해당 설정 클래스에서 관리되는 Bean과 이를 통해 선언적 트랜잭션이 어떻게 동작하는지 살펴보았다.



### 1. **TransactionAttributeSourcePointcut**

```java
@Bean(name = TransactionManagementConfigUtils.TRANSACTION_ADVISOR_BEAN_NAME)
@Role(BeanDefinition.ROLE_INFRASTRUCTURE)
public BeanFactoryTransactionAttributeSourceAdvisor transactionAdvisor(
		TransactionAttributeSource transactionAttributeSource, TransactionInterceptor transactionInterceptor) {

	BeanFactoryTransactionAttributeSourceAdvisor advisor = new BeanFactoryTransactionAttributeSourceAdvisor();
	advisor.setTransactionAttributeSource(transactionAttributeSource);
	advisor.setAdvice(transactionInterceptor);
	if (this.enableTx != null) {
		advisor.setOrder(this.enableTx.<Integer>getNumber("order"));
	}
	return advisor;
}
```

해당 Bean의 내부를 보면 TransactionAttributeSourcePointcut이 정의되어 있다.

<figure><img src="../../.gitbook/assets/image (26).png" alt=""><figcaption></figcaption></figure>

Spring은 @Transactional이 부여된 모든 메서드, 클래스를 자동으로 target 객체로 인식한다.

이 경우 **`TransactionAttributeSourcePointcut`**라는 포인트컷을 이용한다.

<figure><img src="../../.gitbook/assets/image (10).png" alt=""><figcaption></figcaption></figure>

하지만 해당 포인트컷에는 AspectJ와 같은 표현식이 존재하지 않는다. 대신, @Transactional이 명시된 Bean을 모두 찾아 포인트 컷의 필터링 결과로 반환한다. 이를 통해 @Transactional은 단순히 트랜잭션 속성을 정의하는 것 뿐만 아니라, 포인트컷 대상에도 등록이 된다.



### 2. TransactionAttributeSource

해당 빈은 어노테이션에서 Attrubute를 가져오기 위한 Bean이다.

```java
@Transactional(readOnly=true)
```

선언적 트랜잭션을 읽기 전용 속성으로 작성할 때 위와 같이 속성 값을 명시한다. 여기서 속성을 가져오도록하는 Bean이 TransactionAttributeSource이다.



```java
@Bean
@Role(BeanDefinition.ROLE_INFRASTRUCTURE)
public TransactionAttributeSource transactionAttributeSource() {
	// Accept protected @Transactional methods on CGLIB proxies, as of 6.0.
	return new AnnotationTransactionAttributeSource(false);
}
```

**`AnnotationTransactionAttributeSource`** 객체를 반환하는 것을 확인할 수 있다.&#x20;

해당 클래스를 통해 트랜잭션을 적용할 target 객체에서 @Transactional의 속성값을 불러온다. (트랜잭션 전파 속성, 읽기 전용 속성 등등..) 이를 통해 메서드별로 세밀한 트랜잭션 속성 전략을 선택할 수 있다.

<figure><img src="../../.gitbook/assets/image (23).png" alt=""><figcaption></figcaption></figure>

> Spring은 프록시 방식으로 트랜잭션을 적용할 수 있도록 하며 개발자는 선언적 트랜잭션을 통해 여러 옵션에 대해 각각의 프록시 생성 방식을 고려하지 않아도 된다.





## 참고

[https://spring.io/blog/2012/05/23/transactions-caching-and-aop-understanding-proxy-usage-in-spring](https://spring.io/blog/2012/05/23/transactions-caching-and-aop-understanding-proxy-usage-in-spring)

[https://qiita.com/NagaokaKenichi/items/a279857cc2d22a35d0dd](https://qiita.com/NagaokaKenichi/items/a279857cc2d22a35d0dd)

