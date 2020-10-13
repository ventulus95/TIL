# Spring security authorizationGrantType cannot be null 해결 방법



## Spring boot에서 Oauth 설정 값을 처리할때

스프링 부트에서 application.properties나 yaml를 통해서 쉽게 Oauth에 관련된 여러가지 설정들을 적용할 수 있다. 물론 버젼별로 Oauth를 적용하는 방식이 좀 다르긴 하다.

```yaml
security:
    oauth2:
      client:
        registration:
          microsoft:
            client-id: -
            client-secret: -
            redirect-uri-template: '{baseUrl}/login/oauth2/code/{registrationId}'

```

Spring Boot 1.5 버젼에서는 이런식이다.  

하지만, 2.0 버젼으로 넘어오면서 여러가지 포멧이 많이 줄었는데

```yaml
 security:
    oauth2:
      client:
        registration:
          google:
            client-id: -
            client-secret: -
            scope: email, profile
          kakao:
            client-id: d-
            client-secret: -
            redirectUri: '{baseUrl}/login/oauth2/code/{registrationId}'
            authorization-grant-type: authorization_code
            scope: profile,account_email
            client-name: kakao
            clientAuthenticationMethod: post
        provider:
          kakao:
            authorization-uri: https://kauth.kakao.com/oauth/authorize
            token-uri: https://kauth.kakao.com/oauth/token
            user-info-uri: https://kapi.kakao.com/v2/user/me
            user-name-attribute: id
```

다음과 같이 버젼이 변화함에 따라서, 설정에서도 약간의 변경 사항이 적용되어있는데, 소셜로그인을 전달해주는 redirectUri를 지정하는 주소 포멧을 만들때 그 설정 key값이 변경되었다.

1.5버젼에서는 `redirect-uri-template`이  `redirectUri` 로 다음과 같이 변화하였다.

## 근데 변화한 방식을 그대로 적용해도 되는줄 알았으나...

사실 이게 큰 문제를 발생시키는 것 같다고 생각은 안했지만, 실제로 2.2버젼에서 이렇게 `redirect-uri-template`로 사용하게 되면 다음과 같은 에러가 발생한다 

```java
org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'clientRegistrationRepository' 
defined in class path resource [org/springframework/boot/autoconfigure/security/oauth2/client/OAuth2ClientRegistrationRepositoryConfiguration.class]: 
Bean instantiation via factory method failed; nested exception is org.springframework.beans.BeanInstantiationException: 
Failed to instantiate [org.springframework.security.oauth2.client.registration.InMemoryClientRegistrationRepository]: 
Factory method 'clientRegistrationRepository' threw exception; 
nested exception is java.lang.IllegalArgumentException: authorizationGrantType cannot be null
```

보아하니 이게 버젼업에 의해서 생긴 오류였던것을 확인할 수 있었다. 

즉, Spring boot 2.2 버젼내에서는 `redirect-uri-template`를 사용할 수 없다.  `redirectUri`로 바꿔서 사용하자. 아마 2.X 버젼이라면 될 것으로 추측된다.

Oauth Redirect uri 설정시 꼭 주의하자!

https://stackoverflow.com/questions/49315552/authorizationgranttype-cannot-be-null-in-spring-security-5-oauth-client-and-spri





