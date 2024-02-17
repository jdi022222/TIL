# @SpringBootApplication

> @SpringBootApplication이 담당하는 기능과 Spring Boot가 제공하는 AutoConfiguration의 내부 동작을 정리한 글입니다.

## ⚙️ Spring Boot와 자동 환경 설정

Spring은 설정에 대한 과정이 복잡하다.  이를 해결하기 위해 Spring Boot에서는 `AutoConfigure`를 통해 자동 환경 설정을 제공한다. `AutoConfigure`는 spring-boot-starter의 하위 모듈이다.



만약 프로젝트에 Redis를 연결하고 싶다면

```java
implementation'org.springframework.boot:spring-boot-starter-data-redis'
```

위와 같은 의존성을 추가하게 되고 바로 Redis의 RedisTemplate 등등을 이용할 수 있다. 이 뿐만 아니라 Spring Data JPA를 사용한다면 Transaction Manager, datasource와 같은 빈도 자동으로 생성된다.&#x20;



Spring Boot가 아닌 Spring을 이용했다면 해당 빈들을 개발자가 직접 Configuraion에 빈들을 정의해줘야한다. 이런 간편할 설정이 어떻게 가능한지 Spring Boot 애플리케이션의 실행을 살펴보았다.





## 1️⃣ @SpringBootApplication

```java

@SpringBootApplication
public class MeommuApiApplication {

	public static void main(String[] args) {
		SpringApplication.run(MeommuApiApplication.class, args);
	}

}
```

Spring Boot는 Root Class에 @SpringBootApplication 어노테이션을 이용해 Spring Boot의 시작점으로 사용한다.

루트 클래스에 해당 어노테이션만 추가하면, 컴포넌트 스캔을 통한 Bean 등록 및 의존성 주입 등이 자동으로 일어나게 된다.

@SpringBootApplication의 내부 구조를 보면 여러 어노테이션이 존재한다.

![image](https://github.com/jdi022222/TIL/assets/97517890/3d6267db-b196-408d-bf42-0d6e67e6c19c)

가장 중요한 세 가지만 보면

1. @ComponentScan : @Component와 이를 상속한 @Controller, @Service, @Repository 등을 Bean으로 등록한다.
2. @EnableAutoConfiguration : 자동 설정 및 구성을 담당한다. 메타 파일에 정의된 설정들을 필터링하고 자동으로 구성하는 역할을 한다.
3. @SpringBootConfiguration : 컨텍스트나 추가 설정에 대한 import를 진행한다. @Configuration와 동일한 효과를 가진다. @Configuration과 다르게 오직 한 개만 선언 가능하다.

여기서 DataSource나 TransactionManager, RedisTemplate등 빈들을 자동으로 등록해주고 Application 구동을 위한 설정의 책임을 가진 것이 `@EnableAutoConfiguration`이다.



## 2️⃣ @EnableAutoConfiguration

![image](https://github.com/jdi022222/TIL/assets/97517890/9dba61bb-eef4-4098-8527-22e3c54e3095)

1. @AutoConfigurationPackage : 해당 Annotation이 등록되어있는 Class를 base package로 설정
2. @Import(AutoConfigurationImportSelector.class) : Import 할 자동 설정 정보들 관리하는 클래스(AutoConfigurationImportSelector)를 이용한다.

AutoConfigurationImportSelector의 내부 동작을 통해 초기 설정, 빈 주입에 대한 부분이 진행된다.



## 3️⃣ AutoConfigurationImportSelector.class

AutoConfigurationImportSelector는 자동 설정 관련 클래스들을 가져오고 필터링을 하는 클래스다.

내부에 getAutoConfigurationEntry라는 @AutoConfiguration이 붙은 클래스 목록 (Entry)를 가져오는 메서드가 구현되어 있다.

```java
       protected AutoConfigurationEntry getAutoConfigurationEntry(AnnotationMetadata annotationMetadata){
        if(!isEnabled(annotationMetadata)){ // annotationMetadata 설정 활성화 확인
            return EMPTY_ENTRY;
        }
		
        AnnotationAttributes attributes = getAttributes(annotationMetadata); // 애너테이션 속성 import
        List<String> configurations = getCandidateConfigurations(annotationMetadata,attributes); // 후보 구성 클래스 목록
        configurations=removeDuplicates(configurations);
        Set<String> exclusions = getExclusions(annotationMetadata,attributes); // 제외할 클래스 목록
        checkExcludedClasses(configurations,exclusions);
        configurations.removeAll(exclusions);
        configurations = getConfigurationClassFilter().filter(configurations); // 필터링을 수행
        fireAutoConfigurationImportEvents(configurations,exclusions); // 자동 구성 가져오기 이벤트 수행
        return new AutoConfigurationEntry(configurations,exclusions);  // 
        }
```

중요 동작 흐름을 하나씩 살펴보겠다.



### 1. getCandidateConfigurations를 통해 import할 클래스 목록 조회

```java
protected List<String> getCandidateConfigurations(AnnotationMetadata metadata,AnnotationAttributes attributes){
        List<String> configurations=ImportCandidates.load(AutoConfiguration.class,getBeanClassLoader())
        .getCandidates();
        Assert.notEmpty(configurations,
        "No auto configuration classes found in "
        +"META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports. If you "
        +"are using a custom packaging, make sure that file is correct.");
        return configurations;
        }
```

ImportCadidates의 load라는 메서드를 호출한다.

```java
public final class ImportCandidates implements Iterable<String> {

   private static final String LOCATION = "META-INF/spring/%s.imports";
   
   ...
```

LOCATION에 해당하는 경로의 파일을 조회해 @AutoConfiguration 어노테이션이 붙은 클래스들을 모두 가져온다.

External Libraries 패키지 내부를 보면 방금 본 Auto Connfigure를 위한 imports 파일이 존재한다.

![image](https://github.com/jdi022222/TIL/assets/97517890/d1f588c3-f048-48f9-b834-83cc7cbf370c)

해당 imports 파일에는 각 의존성별 AutoConfiguration 설정 정보가 담긴 클래스들이 존재한다.

![image](https://github.com/jdi022222/TIL/assets/97517890/4a439d9a-fe47-4853-b31a-9bccb2a68a46)

해당 클래스들은 모두 @AutoConfiguration 어노테이션이 붙어있어 자동 설정의 대상이 된다.

예시로 Redis 관련 Configure를 살펴보겠다.

![image](https://github.com/jdi022222/TIL/assets/97517890/518be68e-c64e-4bbc-ad13-efd1f91b52d1)

![image](https://github.com/jdi022222/TIL/assets/97517890/df85099e-b77f-4c19-ac9b-fe8a8e42e41e)

@Bean이 붙은 클래스가 3가지 있으며 Redis의 AutoConfiguration이 import될 때 해당 Bean들이 자동으로 등록된다.



```java
@Bean
@ConditionalOnMissingBean(name = "redisTemplate")
@ConditionalOnSingleCandidate(RedisConnectionFactory.class)
public RedisTemplate<Object, Object> redisTemplate(RedisConnectionFactory redisConnectionFactory){
        RedisTemplate<Object, Object> template=new RedisTemplate<>();
        template.setConnectionFactory(redisConnectionFactory);
        return template;
        }
```

@ConditionalOnMissingBean(name = "redisTemplate") 처럼 이미 redisTemplate이라는 Bean이 등록이 됐다면 (@ComponentScan 시점에 등록) 해당 Bean은 자동 등록되지 않는다.



### 2. getExclusions를 통해 필터링할 후보들 선정

제외할 클래스 목록을 가져오기 위해 getExclusions 메서드를 호출한다.

getExclusions 메서드는 EnableAutoConfiguration 어노테이션의 exclude, excludeName 속성 및 spring.autoconfigure.exclude 프로퍼티로부터 제외할 클래스 목록을 가져온다.

제외할 클래스 목록에 포함된 클래스가 후보 구성 목록에 있는지 확인하고, 있는 경우 해당 클래스를 후보 목록에서 제외한다.

> AnnotationMetadata를 사용하여 특정 클래스에 EnableAutoConfiguration 속성을 확인하고 자동으로 빈을 등록하거나 설정을 적용할지 여부를 결정할 수 있다. 만약, 후보 클래스 중에 이미 Component Scan을 통해 생성된 빈이 존재한다면 해당 후보 클래스를 제외해야한다.



### 3. 자동 구성된 Entry 반환

AutoConfigure를 통해 필터링된 빈들이 반환되고 해당 빈들이 빈 컨테이너에 등록되게 된다.

따라서 DataSource나 RedisTemplate\<Object, Object>를 빈으로 등록하지 않아도 Spring Boot가 등록해준 빈을 사용할 수 있다.



> 주의할 점은 RedisTemplate\<Object, Object>만 빈으로 정의되어 있다는 점이다. 만약 제네릭 타입을 String으로 변경하거나 직렬화 방식을 변경하고 싶다면 AutoConfiguration이 만든 빈을 사용하지 않고 개발자가 직접 Configuration에 빈을 등록해야한다.

