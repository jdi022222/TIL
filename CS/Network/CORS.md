# CORS

우선 **SOP**(Same Origin Policy)란?

다른 **출처**의 리소스를 사용하는 것을 제한하는 보안 방식이다.

여기서 **출처**란?

프로토콜, 호스트, 포트가 같다면 동일 출처로 판단한다.

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/6b199294-0be4-47b6-b1b5-eb87d6e20a20/a7c610c7-7822-465b-ab63-1494965c4c9c/Untitled.png)

왜 **SOP**를 사용해야 보안에 도움이 될까?

1. 사용자가 facebook에 로그인을 한다.
2. 해커가 사용자에게 해킹 링크를 보낸다.
3. 이제 해커는 사용자의 토큰을 이용해 해킹 링크로 페이스북에 포스트를 게시한다.
4. 여기서 facebook은 해커의 요청은 **다른 출**처(Cross Origin)으로 판단해 요청을 거부한다.

그렇다면 **다른 출처**의 리소스가 필요하다면 어떻게 할까?

바로 **CORS**가 필요하다.

**CORS**(Cross-Origin Resource Sharing)란?

**다른 출처**의 자원들 공유하는 것이다.

추가 HTTP 헤더를 사용해서 **다른 출처**에 접근할 권한을 부여하도록 브라우저에 알려주는 체제다.

**CORS** 접근제어 시나리오는 세 가지가 있다.

1. 단순 요청 (Simple Request)
2. 프리플라이트 요청 (Preflight Request)
3. 인증정보 포함 요청 (Credentialed Request)

1. 단순 요청 (Simple Request)

   프리플라이트와 다르게 바로 요청을 날린다.

    - 메서드 : GET, POST, HEAD
    - Content-Type : application/x-www-form-urlencoded,multipart/form-data, text
    - 헤더 : Accept, Accept-Language, Content-Language, Content-Type

      ![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/6b199294-0be4-47b6-b1b5-eb87d6e20a20/8876ffed-9253-4033-9ccb-c100a4931b2b/Untitled.png)


    그렇다면 왜 프리플라이트를 사용할까?
    
    - 서버가 CORS에 대해 모른다면 단순 요청에 대해 서버를 보호할 수 없기 때문
    - 서버가 CORS에 대해 모른다고 가정하면
        - 만약 Client에서 프리플라이트를 보내지 않았다면 브라우저는 바로 요청을 서버로 보내고 이를 처리하게 된다. (리소스 수정, 삭제가 바로 반영됨)
        - 하지만 프리플라이트를 보내면 실제 요청을 보내지 않게 됨
    
    ![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/6b199294-0be4-47b6-b1b5-eb87d6e20a20/477b33bb-068a-4aab-8199-6fdbd98b266e/Untitled.png)




1. 프리플라이트 요청 (Preflight Request)

   실제 요청 전에 OPTIONS 메서드를 통해 다른 도메인의 리소스에 요청이 가능한지 확인 작업

   ![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/6b199294-0be4-47b6-b1b5-eb87d6e20a20/a8bcd120-c139-40c9-b8f7-68455fa71bca/Untitled.png)

    1) 프리플라이트 요청의 구성은 다음과 같다.

   출처, 실제 요청 메서드, 실제 요청의 추가 헤더

   ![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/6b199294-0be4-47b6-b1b5-eb87d6e20a20/0627e5f7-b860-4177-a85a-f7ed9ab0d913/Untitled.png)

    2) 프리플라이트의 응답의 구성은 다음과 같다.

   허가 출처, 허가 메서드, 허가 헤더, 응답캐시기간

   단, 응답코드는 200대여야 하며 바디는 비어있는 것이 좋다.

   ![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/6b199294-0be4-47b6-b1b5-eb87d6e20a20/f845a135-2b91-467c-80ec-726cf10e92f3/Untitled.png)


1. 인증정보 포함 요청 (Credentialed Request)

   인증 관련 헤더를 포함할 때 사용하는 요청이다.(JWT와 같은)

   클라이언트 : credentials : include

   서버 : Access-Control-Allow-Credentials : true (*은 안됨)


**CORS** 해결방법도 세가지가 있다.

1. 프론트 프록시 서버 설정
2. 직접 헤더에 설정
3. 스프링 부트 이용

1. 프론트 프록시 서버 설정

   프록시 서버는 클라이언트의 요청을 받아 백서버로 전달하고 서버 응답을 클라이언트에게 전달한다.

   프론트 애플리케이션은 프록시 서버에 대한 요청을 보내므로 CORS 정책을 우회한다.


1. 직접 헤더에 설정
    - `Access-Control-Allow-Origin` : 다른 도메인에서의 접근을 허용할 Origin(도메인)을 설정한다.
    - `Access-Control-Allow-Methods` : 허용할 HTTP 메소드를 설정한다.
    - `Access-Control-Allow-Headers` : 허용할 HTTP 헤더를 설정한다.

1. 스프링 부트 이용

    1) CrossOrigin 어노테이션 이용

    2) WebMvcConfigurer 인터페이스 구현

    ```java
    @Configuration
    public class WebConfig implements WebMvcConfigurer {
        @Override
        public void addCorsMappings(CorsRegistry registry) {
            registry.addMapping("/**")
                    .allowedOrigins("http://localhost:8080")
                    .allowedMethods("GET", "POST", "PUT", "PATCH", "DELETE")
                    .allowedHeaders("Authorization")
                    .exposedHeaders("Custom-Header")
                    .allowCredentials(true)
                    .maxAge(3600);
        }
    }
    ```