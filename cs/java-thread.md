# JAVA Thread



## ？ 동시 접근 문제

### 1. 경쟁 상태

여러 프로세스/스레드가 **동시에 하나의 데이터**를 접근한다면? -> 해당 하는 값의 무결성이 깨지거나 의도치 않은 결과가 일어날 수 있다.&#x20;

<figure><img src="../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

각 프로세스는 다른 프로세스의 존재 여부를 모르고 있다는 가정하에 **프로세스1**과 **프로세스2**가 **동시에** 변수 a를 1씩 증가시키고 있다고 가정해보자.&#x20;



<figure><img src="../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

프로세스1의 입장에서는 a는 1이 되어야 하지만 처리하는 과정에서 프로세스2 또한 해당 값을 증가시키기 때문에 2가 반환될 수 있다.&#x20;

만약, 위처럼 단순한 변수 값이 아니라 결제 금액이나 포인트 같은 민감한 데이터일 경우에는 큰 문제가 발생할 수 있다.



<figure><img src="../.gitbook/assets/image (36).png" alt=""><figcaption></figcaption></figure>

위 예시처럼 여러 프로세스/스레드가 공유 자원을 접근하는 영역을 코드 영역을 <mark style="color:red;">**임계 영역**</mark>이라고 하며 접근 타이밍이나 접근 순서에 따라 결과가 달라질 수 있는 상황을 <mark style="color:red;">**경쟁 조건**</mark>이라고 한다.



> 임계 영역 critical section
>
> 여러 프로세스/스레드가 공유 데이터를 접근하는 프로그램 코드 부분을 의미한다.



> 경쟁 조건 race condition
>
> 여러 프로세스/스레드가 공유 데이터를 접근할 때 타이밍이나 접근 순서에 따라 결과가 달라질 수 있는 상황을 의미한다.



### 2. 동기화

위처럼 경쟁 조건 때문에 의도치 않은 동작을 예방하기 위해 프로세스/스레드 간의 **동기화**를 해야한다. **동기화**는 공유 자원 및 임계 영역에 대해 여러 프로세스/스레드가 동시에 접근하지 못하도록 접근 순서를 제어하는 방법이다.



### 3. 상호 배제

동기화는 **상호 배제**(Mutual Exclution) 매커니즘을 통해 구현된다.

**상호 배제** 매커니즘을 활용하면 여러 프로세스/스레드를 동시에 실행해도 공유 데이터의 일관성을 유지할 수 있다.





## 👍 동기화 방법

동기화는 기본적으로 **락**(Lock)을 이용한다. 하나의 프로세스/스레드가 자원에 접근하면 해당 자원을 잠금(= 락)을 하고 사용이 끝나면 잠금을 해제하는 방식으로 구현된다.&#x20;



### 1. 스핀락

> 락을 가질 수 있을 때까지 반복해서 시도하는 방법



#### 1-1. 바쁜 대기 문제

하지만 스핀락은 프로세스/스레드가 락을 기다리는 동안 CPU를 계속해서 낭비한다는 단점이 있다.&#x20;

단순히 락을 획득하기 위해 지속해서 CPU 자원이 소모되기 때문에 이를 **바쁜 대기**(Busy Waiting)문제라고 한다.





### **3. 세마포어 Semaphore**

동기화 대상이 1개 이상일 때 사용한다.&#x20;

**세마포어**라는 **정수 변수**를 이용해 임계 영역에 몇 개의 프로세스가 진입 가능한지 나타낸다.

P 연산과 V연산으로 해당 **정수 변수**를 이용한다.

* P 연산 : 프로세스가 임계 영역에 진입하는 연산. 세마포어 값을 1 감소한다.
* V 연산 : 프로세스가 임계 영역을 빠져나가는 연산. 세마포어 값을 1 증가한다.



#### 3-1. 바이너리 세마포어 (vs 뮤텍스)





#### 3-2. ?? 세마포어





### **2. 뮤텍스 MUTual EXclusion**

동기화 대상이 1개일 때 사용한다.

스레드는 Key를 가지며 해당 Key를 가진 스레드만이 공유 자원에 접근할 수 있다.

다른 스레드는 sleep 상태로 자원 점유를 위해 대기한다.



### **3. 모니터 Monitor**

동시성을 보다 간편하고 안전하게 다루기 위해 만들어진 **추상화**된 동기화 도구이다.

뮤텍스와 마찬가지로 동기화 대상이 1개일 때 사용한다.



**뮤텍스와의 다른 점**은 뮤텍스는 다른 프로세스간의 동기화 시 이용되며 모니터는 하나의 프로세스 내에서 다른 스레드 간의 동기화에 사용된다.&#x20;



**세마포어와의 다른 점**은 세마포어는 카운터라는 변수값으로 상호배제를 적용하지만 모니터는 이러한 기능이 캡슐화되어 있는 기술이기 때문에 `synchronized`, `wait` 등의 키워드로 상호 배제를 적용할 수 있다.



### 4. 정리

| 특성       | 스핀락 (Spinlock)       | 뮤텍스 (Mutex)          | 세마포어 (Semaphore)     | 모니터 (Monitor)        |
| -------- | -------------------- | -------------------- | -------------------- | -------------------- |
| 동기화 메커니즘 | 상호배제                 | 상호배제                 | 상호배제 및 조건 변수         | 상호배제 및 조건 변수         |
| 대기 방식    | 바쁜 대기 (Busy waiting) | 대기 큐 (Waiting queue) | 대기 큐 (Waiting queue) | 대기 큐 (Waiting queue) |
| 복잡성      | 낮음                   | 중간                   | 중간                   | 중간                   |
| 사용 편의성   | 높음                   | 높음                   | 중간                   | 높음                   |
| 대상       | 프로세스/스레드             | 프로세스/스레드             | 프로세스/스레드             | 모듈/클래스               |
| 자원 낭비    | 높음                   | 낮음                   | 낮음                   | 낮음                   |
| 경쟁 조건 방지 | O                    | O                    | O                    | O                    |
| 재진입 가능   | X                    | O                    | O                    | O                    |





## 🖥️ 모니터

기능

* 상호 배제(mutual exclusion)를 보장
* 조건에 따라 Thread가 대기 상태로 전환 가능



언제 사용?

* 한 번에 하나의 스레드만 실행해야 할 때
* 여러 스레드와 협업이 필요할 때&#x20;



구성 요소

1. mutex : critical section에서 mutual exclusion을 보장하는 장치, mutex lock을 취득하지 못한 스레드는 큐에 들어간 후 대기 상태로 전환(waiting queue). 즉, mutext lock을 쥐고 있는 스레드가 lock을 반환하면 락을 기다리며 큐에 대기 상태로 있던 스레드 중 하나가 실행됨
2.  condition variable : waiting queue를 가짐. 조건이 충족되길 기다리는 스레드들이 대기 상태로 머무는 곳





condition variable 주요 동작

wait : thread가 자기 자신을 condition variable의 waiting queue에 넣고, 대기 상태로 전환

signal : waiting queue에서 대기 중인 스레드 중 **하나**를 깨움

broadcast : waiting queue에서 대기 중인 스레드 중 **전부**를 깨움



<figure><img src="../.gitbook/assets/image (37).png" alt=""><figcaption></figcaption></figure>

mutex lock을 취득하지 못한 스레드는 entry queue로 들어가서 대기하게 된다.

노란색 부분 -> 모니터의 condition variable과 관련된 부분





### 큐 두가지

1. entry queue : mutex에 의해 관리. critical section에 진입을 기다리는 큐
2. waiting queue : condition variable에 의해 관리. 조건이 충족되길 기다리는 큐





## 🚀 JAVA에서 모니터

모니터는 보통 프로그래밍 언어 레벨에서 지원되기 때문에 직접 구현할 일은 없다.

개발자는 지원되는 모니터 기능을 이용하면 된다.



JAVA에서 모든 객체는 내부적으로 모니터를 **하나**만 가진다.



모니터에서 제공하는 mutual exclusion 기능은 synchronized 키워드로 사용한다.

또만, 자바의 모니터는 condition variable을 **하나**만 가진다.



자바 모니터의 세 가지 동작

1. wait
2. notify == signal
3. notify all == broadcast







## 🔍 참고

[https://www.youtube.com/watch?v=gTkvX2Awj6g\&ab\_channel=%EC%89%AC%EC%9A%B4%EC%BD%94%EB%93%9C](https://www.youtube.com/watch?v=gTkvX2Awj6g\&ab\_channel=%EC%89%AC%EC%9A%B4%EC%BD%94%EB%93%9C)

[https://www.youtube.com/watch?v=Dms1oBmRAlo\&ab\_channel=%EC%89%AC%EC%9A%B4%EC%BD%94%EB%93%9C](https://www.youtube.com/watch?v=Dms1oBmRAlo\&ab\_channel=%EC%89%AC%EC%9A%B4%EC%BD%94%EB%93%9C)



