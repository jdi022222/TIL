# SSE + Stream 방식 문제점

> 멈무일기 프로젝트에 SSE + Stream 방식을 사용하게 되면서 발생한 문제와 해결 과정을 정리한 글입니다.

## ✅ 개발 완료 모습

<figure><img src="../../.gitbook/assets/멈무일기 stream (1).gif" alt=""><figcaption></figcaption></figure>



리팩토링 전, chatGPT API를 호출하는 시간은 최소 5초에서 최대 30초까지 굉장히 긴 시간이 소요됐다. 사용자가 애플리케이션을 쓰면서 로딩 화면을 5초이상 본다는 것은 UX 측면에서 좋지 않다고 생각했다. 프로젝트 흐름상, chatGPT가 생성한 데이터가 존재해야 다음 Flow로 진행이 가능했기 때문에 요청 시간동안 loading circle을 띄워서 요청 처리 중이라는 것을 사용자에게 알려줬다.



사용자 경험을 향상시키기 위해 API 호출이 완료되고 스크립트를 한 번에 응답하는 것이 아닌, 위 사진처럼 API호출 중 생성된 한 글자마다 즉시 응답하도록 구현했다.



## 💻 SSE + Stream방식 이용

단발성 응답이 아닌, 클라이언트와 연결을 유지한 채 계속해서 데이터를 전달하는 방식은 Polling과 Long Polling이 있다. Long polling은 응답마다 다시 요청해야하기 때문에 Stream 방식에는 어울리지 않다고 판단했다.



그리고 WebSocket 방식도 고려했었는데 WebSocket의 양방향 통신이 필요 없었고 결과값을 응답하는 단방향 방식이 더 적절하다고 판단했다.



검색해보니 SSE(Server Sent Events)를 이용하면 될 것 같았다.

> SSE란, 클라이언트가 서버로부터 전송되는 데이터를 지속적으로 수신 가능한 단방향(서버 -> 클라) 기술

SSE를 이용해 한 번 요청을 하면 커넥션이 종료될 때까지 서버가 지속적인 응답을 할 수 있다.



## 🚀 개발 과정

SSE로 정했으니 바로 개발에 들어갔다.

우선 외부 API를 Stream 방식으로 받아와야 한다. 다행히도 chatGPT는 stream옵션을 이용하면 간편하게 stream 형식으로 받아올 수 있었다.

<figure><img src="../../.gitbook/assets/image (8).png" alt=""><figcaption></figcaption></figure>



그리고 Webflux의 Flux를 이용해 Stream을 처리했다.  Stream으로 여러 데이터를 emit해야하기 때문에 Mono가 아닌 Flux 타입으로 반환을 했다.

<figure><img src="../../.gitbook/assets/image (3) (1).png" alt=""><figcaption></figcaption></figure>

개발 후 로컬에서 테스트는 잘 동작했으나 배포환경에서는 몇가지 문제가 발생했다.



## 1️⃣ ISSUE 1 - DB connection pool

개발 서버에 배포 후 프론트에서 테스트를 진행했다.

ChatGPT를 통해 결과 생성 요청을 10개정도 연속으로 보내 보았는데 서버가 터지는 문제가 발생했다.



로그를 확인해보니 커넥션풀이 터지는 문제가 발생했다.

<figure><img src="../../.gitbook/assets/image (2) (1).png" alt=""><figcaption></figcaption></figure>



Jmeter를 통해 부하테스트를 진행해봤다.&#x20;

다른 API 요청은 여러건 보내면 정상처리 됐으나 외부 API를 호출하는 요청만 문제가 발생하는 것을 확인했다.



찾아보니 `OSIV(Open-Session-In-View)` 문제였다.

**SSE 통신이 진행되는 동안에는 클라이언트와 서버가 계속해서 HTTP 연결이 맺어져있는 상태여야 한다**. 현재 프로젝트에서 JPA를 이용했고 **HTTP 연결이 유지되는 동안 영속성 컨텍스트도 유지되게 된다**. API를 호출하는 비즈니스 코드의 트랜잭션이 시작되면 영속성 컨텍스트가 DB 커넥션과 함께 생성되는데, `OSIV`가 true로 설정되어 있을 경우 SSE가 진행되는 동안 커넥션도 동시간 점유하게 된다.



사실... `OSIV`의 존재는 알고 있었지만 기본값이 true인 것은 몰랐다. false로 변경해서 테스트를 진행해보니 커넥션도 바로 반납되어 커넥션 풀도 터지지 않았다.



> 결론 : SSE를 사용할 경우 OSIV 옵션은 false로 하자.



## 2️⃣ ISSUE 2 - Nginx proxy buffering setting

로컬에서 테스트 했을 때는 문제 없었지만 온프레미스 서버에 배포 후에 stream 데이터가 한 글자씩 반환되지 않고 덩어리로 가는 문제가 발생했다.



server의 로그를 확인해 봤을 때는 한 글자씩 끊어서 보냈으나 Client에게는 뭉쳐져서 보내졌기 때문에 반쪽짜리 Stream이 되었다.



서버의 배포 환경은 아래와 같았다.

<figure><img src="../../.gitbook/assets/image (22).png" alt=""><figcaption></figcaption></figure>



Docker container로 배포되어 있는 Spring이 Nginx를 거쳐 외부로 응답을 보내는 구조다.

배포 환경을 생각해보니 Nginx의 문제인 것 같아서 default 설정을 찾아봤다.



[https://docs.nginx.com/nginx/admin-guide/web-server/reverse-proxy/](https://docs.nginx.com/nginx/admin-guide/web-server/reverse-proxy/)

Nginx의 공식문서를 보면 응답시 buffering이 있는 것을 확인할 수 있었다. Nginx는 응답의 데이터를 버퍼에 저장해두고 설정된 크기의 버퍼가 차게되거나 서버의 응답이 종료되면 그제서야 클라이언트로 보내게 된다.



Nginx의 버퍼링은 성능을 최적화하는 데 도움이 되기 때문에 기본적으로 버퍼를 이용하는 것이 default이다.



따라서 Stream방식이 적용된 URL만 buffuer를 이용하지 않도록 Nginx 설정을 수정했다.

```bash
# /etc/nginx/\nginx.conf



# 특정 URL에 대한 설정
if ($request_uri = '/api/v1/gpt-stream') {
    proxy_buffering off;
}
```



> 결론 : Stream 방식을 이용할 경우 Nginx의 버퍼 설정을 확인하자.



