# Spring Security를 통한 외부 Oauth2.0 Redirect URI와 로그인 URI이 만들어지는 방식

## 도대체 Redirect URI랑 로그인 URI는 어케 만들어지나...

최근에 소셜로그인을 JWT 방식으로 전환을 하면서 한가지 궁금한 점이 있었다. 그게 뭐냐하면, 도대체 Redirect URI랑 로그인 URI는 Controller도 없이 알아서 다 만들어주는데 그걸  도대체 어디서 확인할 수 있느냐 이런 문제였다. 어차피 URI 생성과 같은 문제는 Controller에서 처리하는 문제인데, Spring  Security는 그런 것도 없이 URI를 생성해주곤 하니 그 발생 진원지가 궁금했다. 

어떻게 만들어지는 질 알아야지 제가 Debug를 할질 알아서 궁금해서 찾아보기 시작했다. 그전에 미리 알고 가야할 것들이 있다. 간단하게 Spring Security에서 Oauth2.0을 어떤식으로 처리하는지에 대해서 생각해보자.

## Spring Security에서 Oauth2.0은...

![Security GIF | Gfycat](https://thumbs.gfycat.com/PoliteFluidGannet-size_restricted.gif)

Oauth의 개념을 일일히 짚고 넘어가는건 여기서 다룰 내용은 아니라고 생각한다. 차후에 개념 정리를 위해서 블로그에 적어두겠지만. 어쨌든 Oauth2.0는 외부 사이트의 인증절차를 간편하게 해주고 통칭 "**소셜로그인**"을 하기위해서는 필수적이다.

물론 Spring Security가 Django처럼 미리 세팅이 되있어서 바로 줍줍해서 먹으면 좋겠으나, 실제로 뭐 미국내에서 가장 유명한 사이트들을 제외한 다른 부분들은 각자 알아서 설정해놓게끔 해둔다. 

![제목 없음-1](/Users/LeeChnagSup/Desktop/My-CODE/제목 없음-1.png)

대충 이런것들..

이런 친구들은 굳이 뭐... application.yml나, application.properties에서 설정해줄 부분이 적다. Client-id나 Client-secret과 같이 그 범위가 매우 좁아 설정할 부분이 적다.



## 하지만 우리가 로그인 하고 싶은거는 이런건데...

<img src="https://t1.kakaocdn.net/kakaocorp/corp_thumbnail/Kakao.png" alt="카카오" style="zoom:33%;" />

![네이버 고객센터](https://ssl.pstatic.net/static/help/img/img_logo_naver_200X200.png)

실제로 우리가 하고싶은건 뭐 구글도 괜찮지만... 한국에 사는 이상 네이버와 카카오 로그인은 필수적인건 둘째치고, 이외(디스코드같은 것들)의 oauth 소셜 로그인이 필요할 수 도 때문에 Spring Security로 직접 구현할 필요가 있을 것이다.  그것을 위해서는 좀 미리 알아둬야하는 것들이 있다.

### Redirect URI?

우리가 타사 로그인을 한뒤에 그 로그인을 바탕으로 최종 인증을 다 맞췄을때 다시 우리 페이지로 돌아와하는 주소를 알려줘야한다. **즉, 인증 받고 우리집으로 돌아올 경로를 알려주는 주소이다.** 그래서 대부분 소셜로그인의 Redirect URI는 소셜로그인을 허용하는 페이지에서 등록을 하게끔 되어있다. 입력해야하는 주소가 대부분 이 주소이다.

![스크린샷 2020-09-10 오전 9.26.50](/Users/LeeChnagSup/Desktop/스크린샷 2020-09-10 오전 9.25.48.png)

![스크린샷 2020-09-10 오전 9.26.50](/Users/LeeChnagSup/Desktop/스크린샷 2020-09-10 오전 9.26.50.png)

스프링부트에서는 다음과 같이 지정을 하고 있다.

http://localhost:8080/login/oauth2/code/google 과 같이 특정 패턴을 가지고 있는데, 비슷하게 네이버나 카카오도 이런 폼을 그대로 살리면서  Redirect URI를 만들 수 있다. 

`{baseUrl}/login/oauth2/code/{registrationId} ` 처럼 URI도 특정한 폼을 지정한 상태로 다른 타사이름만 붙혀주면 계속 생성할 수 있다.



### Spring Boot 2.x Property Mappings

스프링 부트에서는 Application.yml을 통한 데이터를 일정 양식만 지켜주면 알아서 잘 매핑을 하게된다. 

```yaml
spring:
  security:
    oauth2:
      client:
        registration:
          okta: // registrationId
            client-id: okta-client-id
            client-secret: okta-client-secret
        provider: 
          okta: //providerId 
            authorization-uri: https://your-subdomain.oktapreview.com/oauth2/v1/authorize
            token-uri: https://your-subdomain.oktapreview.com/oauth2/v1/token
            user-info-uri: https://your-subdomain.oktapreview.com/oauth2/v1/userinfo
            user-name-attribute: sub
            jwk-set-uri: https://your-subdomain.oktapreview.com/oauth2/v1/keys
```

| Spring Boot 2.x                                              | ClientRegistration                                      |
| :----------------------------------------------------------- | :------------------------------------------------------ |
| `spring.security.oauth2.client.registration.*[registrationId]*` | `registrationId`                                        |
| `spring.security.oauth2.client.registration.*[registrationId]*.client-id` | `clientId`                                              |
| `spring.security.oauth2.client.registration.*[registrationId]*.client-secret` | `clientSecret`                                          |
| `spring.security.oauth2.client.registration.*[registrationId]*.client-authentication-method` | `clientAuthenticationMethod`                            |
| `spring.security.oauth2.client.registration.*[registrationId]*.authorization-grant-type` | `authorizationGrantType`                                |
| `spring.security.oauth2.client.registration.*[registrationId]*.redirect-uri` | `redirectUriTemplate`                                   |
| `spring.security.oauth2.client.registration.*[registrationId]*.scope` | `scopes`                                                |
| `spring.security.oauth2.client.registration.*[registrationId]*.client-name` | `clientName`                                            |
| `spring.security.oauth2.client.provider.*[providerId]*.authorization-uri` | `providerDetails.authorizationUri`                      |
| `spring.security.oauth2.client.provider.*[providerId]*.token-uri` | `providerDetails.tokenUri`                              |
| `spring.security.oauth2.client.provider.*[providerId]*.jwk-set-uri` | `providerDetails.jwkSetUri`                             |
| `spring.security.oauth2.client.provider.*[providerId]*.user-info-uri` | `providerDetails.userInfoEndpoint.uri`                  |
| `spring.security.oauth2.client.provider.*[providerId]*.user-info-authentication-method` | `providerDetails.userInfoEndpoint.authenticationMethod` |
| `spring.security.oauth2.client.provider.*[providerId]*.user-name-attribute` | `providerDetails.userInfoEndpoint.userNameAttributeNa`  |

즉, 일정형식만 잘 지켜진다면 대부분의 모든 정보를 클래스를 만들어서 생성하는 방식이 아니더라도 알아서 Spring Boot가 만들어준다.

물론 직접 AutoConfig해서 일일히 Code화 시키는 방법도 있긴한데, 일정 양식만 잘 지켜주면 Spring Boot가 만들어주기때문에, 자세하게 필요한 경우는 Spring Security를 참고 하자.



### PROVIDER???  그게 뭔데요.

여기서 말하는 Provider는 무슨 역할을 하는가? 하느냐면 Oauth에 대한 꽤나 디테일한 정보들이 들어가야해서 구체적으로 설명하기는 어렵고 쉽게 이야기하면 인증 정보를 위한 모든 URI 주소를 여기서 관리하는 것이다. 

Oauth를 위한 토큰 URI, JWT 토큰 URI, User정보를 받아오는 URI등을 Provider로 설정할 수 있다. 이전에 말했듯, Spring Boot에서 이런 Provider를 제공하는 업체는 딱 4군데 정도 밖에 없다. 그래서 우리가 타사의 Oauth2.0을 이용하기 위해서는 반드시 다음과 같이 Provider를 설정해줘야한다.

네이버나 카카오는 다음과 같이 지정해두곤 한다.

![스크린샷 2020-09-10 오전 9.32.33](/Users/LeeChnagSup/Desktop/스크린샷 2020-09-10 오전 9.32.33.png)

![스크린샷 2020-09-10 오전 9.33.53](/Users/LeeChnagSup/Desktop/스크린샷 2020-09-10 오전 9.33.53.png)

https://developers.naver.com/docs/common/openapiguide/apilist.md#%EB%A1%9C%EA%B7%B8%EC%9D%B8-%EB%B0%A9%EC%8B%9D-%EC%98%A4%ED%94%88-api

https://developers.kakao.com/docs/latest/ko/reference/rest-api-reference

그래서 Provider에 맞는 정보를 잘 매핑하면 된다. 토큰은 토큰대로 뭐 프로필 정보 조회는 조회대로 잘 매칭하면 되기때문에 필요에 따라서 넣으면 될것이다.



## 그럼 로그인 페이지로는 어떤 URI가 필요한가요?

우리가 구글 로그인 소셜 로그인 버튼을 만들때 A태그에 어떤 URI를 넣어서 작동해야하는가를 고민해보면 이것또한 스프링부트가 알아서 생성을 해준다.

```html
<a href="/oauth2/authorization/google">Google</a>
```

보면 `/oauth2/authoriztion/google`과 같은 모습을 만들었는데,  이런 것도 역시 일정 Form만 유지해주면 알아서 자기가 생성해준다

`OAuth2AuthorizationRequestRedirectFilter.DEFAULT_AUTHORIZATION_REQUEST_BASE_URI + "/{registrationId}"`

과 같이 RegistrationId만 존재하면 대부분 동일하게 만들어준다는 것을 알 수 있다. 



**즉, 이걸 바탕으로 소셜 로그인에 필요한 입구 URI  돌아올 수 있게하는 URI가 완성됨을 알 수 있다.  **

이런 방식으로 Spring Security에서는 Redirect URI와 소셜 로그인 URI를 만들어주는 것을 알 수 있었다. 차후에 다른 것에 대해서도 알아보는 기회를 가지겠다.

