# ApplicationContext

> IOC 컨테이너, ApplicationContext에 대해 정리한 글입니다.

## Spring과 IOC 컨테이너

객체지향적으로 코딩하다보면 객체간의 의존성이 커지는 문제를 마주하게 된다. 어느 한 객체가 다른 객체의 생성을 담당하거나 한 객체의 수정사항이 다른 객체에 영향을 줄 경우 의존성이 발생한다. 이로 인해 각 객체간 결합도가 높아지고 유지보수성이 떨어지는 문제가 생길 수 있다.



SOLID 원칙 중 하나인 **`IOC(Inversion of Control)`**는 객체 호출 흐름의 제어를 개발자(내부)가 아닌 외부에 맡기는 원칙이다. 외부에서 객체간의 의존성을 해결하게 되고 이를 통해 코드의 유연성이 높아지고 결합도는 낮아지게 된다.



Spring framework에서는 개발자가 직접 객체를 생성하지 않고 컨테이너가 생성해주고 필요한 곳에 주입을 한다. IOC 원칙을 이용한 IOC 컨테이너가 객체를 생성하고 객체간 **`의존성 주입(Dependency Injection)`**도 해준다.



생각해보면 Spring을 통해 개발했을 때 Controller나 Service와 같은 클래스를 new라는 키워드를 통해 인스턴스를 생성한 적이 없다. Controller의 생성자에 주입되는 객체도 인스턴스를 생성한 적이 없다. 이 또한 객체의 생성을 개발자가 아닌 IOC 컨테이너가 담당했기 때문이다.



이러한 컨테이너가 만들고 관리하는 객체를 **`Bean`**이라고 한다. 이러한 Bean을 관리하기 때문에 IOC 컨테이너를 **`Bean Factory(빈 팩토리)`**라고도 부른다. Spring framework에서는 이 BeanFactory를 인터페이스로 정의해놓았다.

\


## ApplicationContext 상속 구조

<figure><img src="../../.gitbook/assets/image (1) (1).png" alt=""><figcaption><p>ApplicationContext인터페이스의 구조</p></figcaption></figure>

Spring에서 BeanFactory를 생성하기 위한 인터페이스들의 다이어그램이다.최상단에 BeanFactory가 있고 최하단에 `ApplicationContext`가 존재한다.



BeanFactory는 `getBean()`, `containsBean()`, `isSingleton()`과 같이 Bean에 대한 라이프사이클을 관리하는 기능을 담고있다.



BeanFactory는 Bean의 라이프사이클을 관리하는 기능만 있다면, ApplicationContext는 BeanFactory 기능과 함께 아래와 같은 추가적인 기능을 제공한다.&#x20;

<figure><img src="../../.gitbook/assets/image (3) (1).png" alt=""><figcaption><p><a href="https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/beans/factory/BeanFactory.html">https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/beans/factory/BeanFactory.html</a></p></figcaption></figure>



메세지 소스 관리, ResouceLoader, EventPublisher 기능 등 웹 개발에 자주 사용되는 기능이 제공되므로  `ApplicationContext` 인터페이스를 이용하면 된다.



> 여담으로, Event를 생성하여 Pub/Sub 방식으로 통신할 때, ApplicationContext는 ApplicationEventPublisher를 포함하고 제공한다. ApplicationEventPublisher는 이벤트를 발행하고 구독하는 기능을 제공하는 인터페이스로, ApplicationContext를 통해 사용되며 이벤트를 통해 도메인간 의존성을 없애고 비동기적인 호출을 할 때 사용된다.





## ApplicationContext의 구현체

그럼, ApplicationContext의 구현체들을 살펴보자.

<figure><img src="../../.gitbook/assets/image (1) (1) (1).png" alt=""><figcaption><p>ApplicationContext의 다양한 구현체</p></figcaption></figure>

다양하다. 실제로 사용해 본 구현체만 간단하게 설명하겠다.



### 1. WebApplicationContext

<figure><img src="../../.gitbook/assets/image (10).png" alt=""><figcaption></figcaption></figure>

WebApplication은 구현체는 아니고 인터페이스이다. 이름 그대로 웹 환경에서 사용할 때 필요한 기능이 추가된 인터페이스이다.



### 2. AnnotationConfigWebApplicationContext&#x20;

<figure><img src="../../.gitbook/assets/image (11).png" alt=""><figcaption><p>너무 복잡해서 가지치기를 했다</p></figcaption></figure>

Annotation 설정을 기반으로 빈을 등록하는 WebApplication의 구현체이다.



### 3. XmlWebApplicationContext

<figure><img src="../../.gitbook/assets/image (12).png" alt=""><figcaption><p>너무 복잡해서 가지치기를 했다</p></figcaption></figure>

Web.xml의 설정을 기반으로 빈을 등록하는 WebApplication의 구현체이다.





### 어떤 것이 사용될까?

@SpringBootApplication 어노테이션이 붙은 root 클래스에서 `SpringApplication.run();`을 할 때 해당 애플리케이션의 타입에 따라 사용되는 ApplicationContext가 다르다.



ApplicationType은 크게 세 가지가 Enum으로 제공된다.

```
/**
 * The application should not run as a web application and should not start an
 * embedded web server.
 */
NONE,

/**
 * The application should run as a servlet-based web application and should start an
 * embedded servlet web server.
 */
SERVLET,

/**
 * The application should run as a reactive web application and should start an
 * embedded reactive web server.
 */
REACTIVE;
```



ClassLoader를 통해 클래스 목록을 확인하여 해당 애플리케이션의 타입을 할당하고 적절한 ApplicationContext 구현체를 사용한다.

<figure><img src="../../.gitbook/assets/image (13).png" alt=""><figcaption></figcaption></figure>

일반적인 Spring Boot Web 어플리케이션은 `AnnotationConfigServletWebServerApplicationContext`을 이용하게 된다.&#x20;





## ApplicationContext 동작 과정

1. XML or 어노테이션 기반으로 설정 파일 로드 후 Bean 생성
2. Bean간의 의존성 주입
3. Bean 초기화
4. 애플리케이션 실행
5. 실행 중 빈 들의 라이프사이클 관리
6. 애플레케이션 실행 중 ApplicationEvent 발생 시 등록된 EventListener에게 위임
7. 애플리케이션 종료시 Bean 소멸



컨테이너에서 빈을 꺼내오고 싶다면 BeanFactory(를 구현한 구현체)의 `getBean`메서드를 요청하게 되면 ApplicationContext 빈 컨테이너에서 요청한 이름이 있는지 확인하고 반환한다.&#x20;



## DispatcherServlet과의 관계

그렇다면 웹 애플리케이션을 만들 때 ApplicationContext가 어떻게 사용될까?



웹 애플리케이션에서 사용되는 ApplicationContext를 상속한 인터페이스인 `WebApplicationContext`가 `DispatcherServlet`과 함께 사용된다.

<figure><img src="../../.gitbook/assets/image (9).png" alt=""><figcaption><p>DispatcherServlet이 생성될 때 WebApplicationContext를 파라미터로 받는 생성자</p></figcaption></figure>



<figure><img src="../../.gitbook/assets/image (2) (1).png" alt=""><figcaption><p>WebApplicationContext는 ApplicationContext를 상속받은 인터페이스 중 하나이다.</p></figcaption></figure>



DispatcherServlet은 Spring MVC가 웹 요청을 처리할 때 사용되는 핵심 컴포넌트다. DispatcherServlet은 내부적으로 WebApplicationContext의 구현체를 사용해 빈을 관리한다. ~~관련된 내용은 더 공부해서 다음 글에 이어서 작성하겠다.~~



## 실습 - 컨테이너가 관리하는 모든 Bean 조회

참고 : [https://www.youtube.com/watch?v=qEzRBmw6YlI](https://www.youtube.com/watch?v=qEzRBmw6YlI)



1. 프로젝트에 spring actuator 의존성을 추가한다.

```
implementation 'org.springframework.boot:spring-boot-starter-actuator'
```



2. application.yml에 Bean 목록 조회 엔드포인트를 열어준다.

```
# Actuator
management:
  endpoint:
    beans:
      enabled: true
  endpoints:
    web:
      exposure:
        include: beans
```



3. 애플리케이션 구동 후 {project URL}/actuator/beans로 접속한다.

<figure><img src="../../.gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>



JSON Viewer를 설치하면 아래처럼 정렬된 상태로 볼 수 있다.

<figure><img src="../../.gitbook/assets/image (7).png" alt=""><figcaption></figcaption></figure>

프로덕트 코드에서 등록된 authController라는 Bean이 등록된 것을 확인할 수 있다. 추가적으로 singleton으로 관리되며 해당 Bean과 의존성을 가지는 Bean도 확인할 수 있다.



<figure><img src="../../.gitbook/assets/image (8).png" alt=""><figcaption></figcaption></figure>

실제 authController 코드는 위와 같다. 생성자를 통한 의존성 주입만 명시되어 있다. 컨테이너가 해당 Controller Component를 개발자 대신 싱글톤 빈으로 등록해준다. 추가적으로 authService는 Interface인데 해당 인터페이스의 구현체인 authServiceImpl도 찾아서 주입해주는 것을 확인할 수 있다.



코드상 어디에도 AuthController를 인스턴스화하는 부분은 없지만 ComponentScan 과정을 거쳐 Spring이 알아서 의존성을 주입하고 빈을 컨테이너에 등록하는 것을 확인해보았다.





