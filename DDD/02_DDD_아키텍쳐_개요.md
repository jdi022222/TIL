# 02 아키텍쳐 개요

## 아키텍쳐의 네 가지 영역

표현, 응용, 도메인, 인프라스트럭쳐는 DDD 아키텍쳐를 설계할 때의 대표적인 네 가지 영역이다.

* **`표현`** : 사용자의 요청을 응용에 전달하고 처리 결과를 다시 사용자에게 보여준다. spring MVC가 표현 영역을 위한 기술이다.
* **`응용`** : 실제 기능 구현이다. 기능을 구현하기 위해 도메인 용역의 도메인 모델을 이용한다. 응용 서비스는 로직을 직접 수행하기보다 도메인 모델에 로직 수행을 위임한다.
* **`도메인`** : 도메인 모델을 구현한다. 도메인의 핵심 로직을 가진다.
* **`인프라스트럭쳐`** : 구현 기술에 대한 것을 다룬다. 논리적인 개념보다는 실제 구현을 다룬다. 도메인, 응용, 표현 영역은 구현 기술을 사용한 코드를 직접 만들지 않으며 인프라에서 제공하는 기능을 사용해서 필요한 기능을 만든다.

각 계층 구조는 상위 계층에서 하위 계층으로 단방향 의존하게 된다. 구현의 편리함을 위해 응용에서 인프라를 의존하기도 한다. 하지만 이런 방식은 테스트가 어려워진다. 응용 계층을 테스트 하기 위해서는 인프라가 완벽하게 구현이 되어있고 동작해야 비로소 테스트가 가능해지기 때문이다. 결국, 테스트와 기능 확장이 어려워지게 된다. 이를 해결하기 위해 DIP를 이용한다.



### DIP

DIP : 의존 역전 원칙이다. 저수준 모듈을 추상화한 인터페이스로 만들어 구현 객체를 생성자를 통해 전달한다. DIP를 통해 대역 객체를 통해 인프라가 구현되어 있지 않더라도 응용 객체를 테스트할 수 있게 된다. 다만, 단순히 저수준 모듈에서 인터페이스를 추출하면, 도메인 영역은 여전히 인프라 영역을 의존하게 될 수도 있다. 따라서 추상화한 인터페이스는 고수준 모듈에 위치해야 한다.



## 도메인 영역

다음은 도메인 영역의 주요 구성요소이다.

1. 엔티티
2. 밸류
3. 애그리거트
4. 리포지터리
5. 도메인 서비스



### 1. 엔티티

도메인 모델과 DB 테이블은 같은 것이 아니다. 두 모델의 가장 큰 차이점은 도메인 모델의 엔티티는 데이터와 함께 도메인 기능을 제공한다.

```java
public class Order {
	private OrderNo number;
	private Orderer orderer;
	private ShippingInfo shippingInfo;
	...

	public void changeShippingInfo(ShippinInfo shippingInfo) {
        ...
	}
}
```

추가적으로, 두 개 이상의 데이터를 밸류 타입을 이용해 표현 가능하다.

RDBMS와 같은 RDB는 밸류 타입을 제대로 표현하기가 어렵다. 별도 테이블로 분리해 외래키로 관리해야 한다.



### 2. 밸류

밸류 타입은 불변으로 관리해야한다. 만약, 기존의 밸류 타입 데이터를 변경하고자 할 때는 새로운 객체를 생성해서 필드에 할당한다.

```java
public class Order {
	private OrderNo number;
	private Orderer orderer;
	private ShippingInfo shippingInfo;
	...

	public void changeShippingInfo(ShippinInfo newShippingInfo) {
		if (newShippingInfo == null) {
		}
		// 밸류 타입의 데이터는 새 객체로 교체
		this.shippingInfo = newShippingInfo;
	}
}
```



### 3. 애그리거트

애그리거트는 관련 객체를 하나로 묶은 군집이다. 애그리거트를 이용해 개별 객체가 아닌 관련있는 여러 객체를 모아 군집 단위로 모델을 관리할 수 있게 된다.

애그리거트는 군집에 속한 객체들을 관리하는 루트 엔티티를 갖는다.애그리거트를 사용하는 코드는 애그리거트 루트가 제공하는 기능을 실행하고, 루트를 통해 간접적으로 애그리거트 내의 다른 엔티티나 밸류 객체에 접근한다. -> 애그리거트 단위로 구현을 캡슐화 가능

```java
public class Order {
	private OrderNo number;
	private Orderer orderer;
	private ShippingInfo shippingInfo;
	...

	public void changeShippingInfo(ShippinInfo newShippingInfo) {
		if (newShippingInfo == null) {
		}
		checkShippingInfoChangeable();
		this.shippingInfo = newShippingInfo;
	}

	// 배송지 정보를 변경 가능한지에 대한 도메인 규칙 구현부
	private void checkShippingInfoChangeable() {}
}
```

애그리거트를 구현할 때는 구현 복잡도와 트랜잭션 범위, 구현 기술에 따른 애그리거트 제약 등을 고려해야한다.



### 4. 리포지터리

리포지터리는 애그리거트 단위로 도메인 객체를 저장하고 조회하는 기능을 제공한다. 예를 들어 주문 애그리거트 리포지터리는 아래와 같다.

```java
public interface OrderRepository {
	Order findByNumber(OrderNumber number);
	void save(Order order);
	void delete(Order order);
}
```

메서드를 보면 CRUD의 단위가 루트 애그리거트인 Order임을 알 수 있다. Order 객체는 애그리거트에 속한 모든 객체를 포함하므로 결과적으로 애그리거트 단위로 CRUD작업을 수행할 수 있다. OrderRepository는 영속화하는데 필요한 기능을 추상화해놓은 고수준 모듈므로 저수준 모듈로 구현해서 이용해야 한다.



### 5. 도메인 서비스

특정 엔티티에 속하지 않은 도메인 로직이다.

예를 들어 할인 금액 계산은 다양한 조건이 필요한데, 도메인 로직이 여러 엔티티와 밸류를 필요로 할 경우 도메인 서비스에서 로직을 구현한다.



## 인프라스트럭쳐

인프라스트럭쳐는 표현, 응용, 도메인 영역을 지원하며 도메인 영역과 응용 영역에서 정의한 인터페이스를 구현한다. 이를 통해 시스템을 **변경에 유연**하고 **테스트하기 쉽게** 만들어준다.

하지만 무조건 인프라트스럭처에 대한 의존을 없앨 필요는 없다. Spring에서는 응용 서비스에서 @Transcational을 사용하는 것이 편리하며 JPA를 사용할 경우 @Entity나 @Table과 같은 애너테이션을 도메인 모델에 적용하는 것이 XML 매핑보다 편리하다.

```java
public class OrderService {
	
	@Autowired
	OrderRepository orderRepository;

	@Transactional
	public void Save() {
	}
}
```

```java
@Entity
@Table
public class Order {
}
```

**구현의 편리함**은 DIP가 주는 다른 장점만큼 중요하기 때문에 DIP를 해치지 않는 선에서 상위 계층에서 구현 기술에 대한 의존을 가져가는 것도 나쁘지 않다.
