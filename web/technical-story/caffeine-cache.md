# Caffeine Cache 적용기

> Spring Boot 프로젝트에 Caffeine Cache를 적용해보는 글입니다.

## 📚 Cache란

**캐시란?**

데이터를 원하는 클라이언트 <-> 실제 데이터 저장소 사이에서 실제 데이터를 미리 복사해두어 조회 속도를 올려주는 임시 저장소이다.



클라이언트가 데이터 조회시 캐시 저장소를 우선적으로 조회하는데 캐시가 존재하면 **`캐시히트`,** 존재하지 않는다면 **`캐시미스`**라고 하며 **`캐시미스`**가 발생 시 실제 데이터 저장소를 조회하게 된다.



**`캐시미스`**가 자주 발생하면 데이터를 실제 데이터 저장소에서 가져오는데 필요한 시간이 추가되기 때문에 성능 저하가 발생한다. 따라서 캐시를 적용할 데이터를 신중히 결정해야 한다.



**캐시를 적용하면 좋은 경우 예시**

* 실시간성이 중요하지 않은 경우 (제품 카테고리, 공지사항)
* 매번 DB에서 갖고올 필요가 없는 경우 (조직도, 부서 코드)



Spring Boot는 **`Spring-Boot-Starter-Cache`** 의존성을 통해 간편하게 **`로컬캐시`**를 적용할 수 있다. 여기에 추가 기능을 제공하는 **`Ehcache`**나 **`Caffeine`**을 추가하여 자주 사용한다. 아래와 같은 이유로 둘 중에  Caffeine 캐시를 적용했다!





## ☕️ Caffeine Cache를 선택한 이유

* Ehcache, 단순 ConcurrentHashMap보다 월등한 성능 [링크](https://github.com/ben-manes/caffeine/wiki/Benchmarks)
* Caffeine은 spring-boot-starter-cache 에 auto-configured 되어 있음 [링크](https://github.com/spring-projects/spring-boot/blob/v3.2.2/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/cache/CaffeineCacheConfiguration.java)
  * spring-boot-starter-cache의 기본 구현체와 동일하게 concurrentHashMap을 이용하지만 caffeine은 추가적인 최적화가 보다 간편하게 가능 (크기 제한, 만료 정책 등)
  * 현직에서 사용되는 캐시에 대한 정책은 요구사항이 매우 복잡하며 기본 캐시를 이용한다면 복잡한 요구사항들을 직접 구현해야 함 (TTL, TTI, 페이지 교체 알고리즘 등)
* Xml를 사용하는 Ehcache보다 Spring 친화적임
* 캐시 히트율이 높은 **`Window TinyLFU`**를 기본 알고리즘으로 사용
* 단, 분산 환경에서는 사용 불가 (Ehcache는 가능)





## 👋 Caffeine Cache 사용 방법

프로젝트에 적용하는 방법은 잘 설명된 레퍼런스가 많이 때문에 아주아주 간단하게 적었다.



**build.gradle에 Cache 의존성 추가**

```java
// cache
implementation 'org.springframework.boot:spring-boot-starter-cache'
implementation 'com.github.ben-manes.caffeine:caffeine'
```



**CacheConfig 생성**

* @EnableCaching을 통해  Cache 사용을 명시한다
* Enum으로 등록한 CaffeineCache들을 리스트로 빈에 등록한다

```java
@Configuration
@EnableCaching
public class CacheConfig {

	@Bean
	public List<CaffeineCache> caffeineCaches() {
		return Arrays.stream(CacheType.values())
			.map(cache -> new CaffeineCache(cache.getCacheName(),
				Caffeine.newBuilder().recordStats()
					.expireAfterWrite(cache.getExpiredAfterWrite(), TimeUnit.SECONDS)
					.maximumSize(cache.getMaximumSize())
					.build()))
			.toList();
	}

	@Bean
	public CacheManager cacheManager(List<CaffeineCache> caffeineCaches) {
		SimpleCacheManager cacheManager = new SimpleCacheManager();
		cacheManager.setCaches(caffeineCaches);
		return cacheManager;
	}
}
```



**Cache Enum 생성**

* 사용할 캐시들을 명시한다

```java
public enum CacheType {

	NOTICE("notice", 60 * 60, 10000),

	GPT_GUIDE("gpt_guide", 60 * 60, 10000),

	GPT_GUIDE_DETAIL("gpt_guide_detail", 60 * 60, 10000);

	private final String cacheName;
	private final int expiredAfterWrite;
	private final int maximumSize;

	CacheType(String cacheName, int expiredAfterWrite, int maximumSize) {
		this.cacheName = cacheName;
		this.expiredAfterWrite = expiredAfterWrite;
		this.maximumSize = maximumSize;
	}

	public String getCacheName() {
		return cacheName;
	}

	public int getExpiredAfterWrite() {
		return expiredAfterWrite;
	}

	public int getMaximumSize() {
		return maximumSize;
	}
}
```



**캐시를 적용할 로직에 @Cacheable 어노테이션 명시**

```java
@Override
@Cacheable(cacheNames = "notice")
public NoticeResponses findNotices() {
	List<Notice> notices = noticeRepository.findAllByOrderByCreatedAtDesc();
	return NoticeResponses.from(notices);
}
```





## 1️⃣ 적용 1 : 공지사항 데이터

우선 공지사항에 캐시를 적용해보았다.

현재 프로젝트에서 제공하는 공지사항 데이터는 업데이트 주기가 1주일 이상이므로 해당 데이터에 일주일의 캐시를 적용했다. 캐시 사용 전후로 부하테스트를 진행하고 Scouter로 Heap, TPS, 응답시간 등을 모니터링했다.



### 1. 캐시 적용 전 부하테스트

* Vuser(가상 사용자) : 10
* Duration : 1분

<figure><img src="../../.gitbook/assets/image (12).png" alt=""><figcaption></figcaption></figure>



### 2. 캐시 적용 후 부하테스트

* Vuser(가상 사용자) : 10
* Duration : 1분

<figure><img src="../../.gitbook/assets/image (2) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>



### 3. Scouter로 모니터링

<figure><img src="../../.gitbook/assets/image (7) (1) (1).png" alt=""><figcaption></figcaption></figure>

빨간 색이 캐시 적용 후, 파란 색이 캐시 적용 전이다.

캐시를 적용하면서 TPS는 높게 치솟았지만 GC Count 횟수와 Heap 사용량도 증가함을 확인할 수 있다.



### 4. 수치 변화

* **`수치 변화`**
  * 평균 TPS : { 5800 } → { 12800 } **약 2배 개선**
  * Peek TPS : { 7000 } → { 14800 }
  * Mean Test Time : { 1.59 } ms → { 0.7 } ms **약 2배 단축**
  * Exected Tests : { 340000 } → { 750000 } **약 2배 실행**
* **`지표 변화`**
  * 요청이 더 많아져서 GC가 더 많이 발생
  * 그만큼 Heap 영역도 많이 사용



## 2️⃣ 적용 2 : 가이드 리스트 데이터

프로젝트에서 제공하는 chatGPT 가이드 리스트 기능은 JWT 토큰 인증 과정이 추가된다.

따라서 각 테스트 전에 Vuser별로 JWT 토큰을 발급받아 헤더에 추가하는 과정이 필요하다.



Ngrinder Script의 **`@BeforeProcess`**에 각 프로세스별로 토큰을 헤더에 추가했다.

```
@BeforeProcess
public static void beforeProcess() {
	HTTPRequestControl.setConnectionTimeout(300000)
	test = new GTest(1, "127.0.0.1")
	request = new HTTPRequest()
	grinder.logger.info("before process.")
	headers.put("Authorization", "Bearer eyJhbGciOiJIUzUxMiJ9.eyJpZCI6MSwiaWF0IjoxNzA3ODM2OTQzLCJleHAiOjE3MDg0NDE1NDN9.OqJQCItfErjA7lKIVDPbJ79PwKigmwscG0g5wDE3jYwLxCbw_mrVdAG0odjxDlP9t-bf7-e_YrI3xAth28MLMw")
}
```



### 1. 캐시 적용 전 부하테스트

* Vuser(가상 사용자) : 10
* Duration : 1분

<figure><img src="../../.gitbook/assets/image (3) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>



### 2. 캐시 적용 후 부하테스트

* Vuser(가상 사용자) : 10
* Duration : 1분

<figure><img src="../../.gitbook/assets/image (4) (1) (1).png" alt=""><figcaption></figcaption></figure>



### 3. Scouter로 모니터링

<figure><img src="../../.gitbook/assets/image (6) (1) (1).png" alt=""><figcaption></figcaption></figure>

이번엔 왼쪽이 캐시 적용 전, 오른쪽이 캐시 적용 후 이다.

캐시를 적용하면서 TPS는 높게 치솟았지만 요청을 많이 처리하게 되면서 Heap 사용량이 증가했으며, 그에 따라 GC Count 횟수도 비례해서 증가함을 확인할 수 있다.



### 4. 수치 변화

* **`수치 변화`**
  * 평균 TPS : { 2700 } → { 4800 } **약 1.8배 개선**
  * Peek TPS : { 3622 } → { 5419 }
  * Mean Test Time : { 3.48 } ms → { 2.00 } ms **약 1.5배 단축**
  * Exected Tests : { 158600 } → { 278100 } **약 1.8배 실행**
* **`지표 변화`**
  * 요청이 더 많아져서 GC가 더 많이 발생
  * 그만큼 Heap 영역도 많이 사용



## 🎬 결론

> 캐시를 적용할 데이터의 특성과 트레이드 오프를 따져가며 캐시의 적용 여부를 결정하자!\
> 만약, 캐시를 적용할지 말지 고민된다면 부하테스트와 모니터링을 통해 캐시 적용 전후 차이에 대해 확인해보자!

