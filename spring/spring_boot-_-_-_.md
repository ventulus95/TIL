# Spring Boot Security로 카카오 소셜 로그인 만들기

## 참고 할점....

이번 포스팅은 스프링 부트와 AWS로 혼자 구현하는 웹 서비스의 소셜로그인 파트를 참조하여 만들었습니다. 즉, 이 책을 기반으로 코드를 추가한 것이기 때문에 자세한 코드는 책을 통해서 확인 해보시는 것을 추천드립니다.

![&#xC2A4;&#xD504;&#xB9C1; &#xBD80;&#xD2B8;&#xC640; AWS&#xB85C; &#xD63C;&#xC790; &#xAD6C;&#xD604;&#xD558;&#xB294; &#xC6F9; &#xC11C;&#xBE44;&#xC2A4;](http://image.yes24.com/goods/83849117/800x0)

이동욱님의 강좌인 스프링부트를 활용한 소셜로그인 파트를 확인했을때는 구글과 네이버의 소셜 로그인이 예제로 나와있다. 하지만, 카카오는 없어서 한번 동일한 구성으로 카카오를 만들 수 있는지 궁금해서 만들어보았다.

## 카카오 소셜로그인 추가하기

기존 구성과 카카오는 조금의 다른 구성을 가지고 있다. 일단 카카오 소셜로그인을 위해서 카카오 디벨로퍼로 가야한다.

1. **카카오톡 로그인 후 내 애플리케이션을 만든다.**

   ![&#x1109;&#x1173;&#x110F;&#x1173;&#x1105;&#x1175;&#x11AB;&#x1109;&#x1163;&#x11BA; 2020-08-20 &#x110B;&#x1169;&#x1112;&#x116E; 11.32.41](https://github.com/ventulus95/TIL/tree/76f05b8af8c692b1182e679132dbfffb4736169a/Users/LeeChnagSup/Desktop/스크린샷%202020-08-20%20오후%2011.32.41.png)

2. **들어가면 앱키로 각자의 id값이 존재한다. 현재 Spring Security를 통해서 개발할 예정이기때문에, REST API키를 id로 알고 있으면 된다.**

   ![&#x1109;&#x1173;&#x110F;&#x1173;&#x1105;&#x1175;&#x11AB;&#x1109;&#x1163;&#x11BA; 2020-08-20 &#x110B;&#x1169;&#x1112;&#x116E; 11.32.55](https://github.com/ventulus95/TIL/tree/76f05b8af8c692b1182e679132dbfffb4736169a/Users/LeeChnagSup/Desktop/스크린샷%202020-08-20%20오후%2011.32.55.png)

> 좀 해맨부분이 있는데 실제로 네이버는 Secret이 따로 존재하는데 카카오는 처음 설정하면 Secret이 따로 없었다. 근데 이게 설정하는 곳이 따로 존재했다?!

1. **아래 보안으로 내려가면 Client Secret을 발급받을 수 있다. 이 값을 secret으로 지정하면된다.**

![&#x1109;&#x1173;&#x110F;&#x1173;&#x1105;&#x1175;&#x11AB;&#x1109;&#x1163;&#x11BA; 2020-08-20 &#x110B;&#x1169;&#x1112;&#x116E; 11.33.11](https://github.com/ventulus95/TIL/tree/76f05b8af8c692b1182e679132dbfffb4736169a/Users/LeeChnagSup/Desktop/스크린샷%202020-08-20%20오후%2011.33.11.png)

1. **카카오는 Spring Security Oauth에 url가 바로 제공되지 않으므로, 스스로 입력을 해줘야한다. 이때는 application-oauth.properties에 추가를 해줘야한다. `Provider`와 `Registration`을 스스로 등록해야한다.  `Registration`는 내가 이 인증하기 위한 방식들 예를들어 Key값 secret값과 같은 정보들과 무슨 정보를 받을건지를 지정하는 쪽이다. `Provider`는 내가 이 oauth인증을 받기위한 주소들을 기입하는 장소이다. 토큰발급과 유저정보를 받아오는 URI에 대한 정보가 적혀있어야한다.**

   ```text
   ## KAKAO Login
   spring.security.oauth2.client.registration.kakao.client-id=REST API 키
   spring.security.oauth2.client.registration.kakao.client-secret=아까만든 secret키 
   spring.security.oauth2.client.registration.kakao.redirect-uri={baseUrl}/{action}/oauth2/code/{registrationId}
   spring.security.oauth2.client.registration.kakao.authorization-grant-type=authorization_code
   spring.security.oauth2.client.registration.kakao.scope=profile
   spring.security.oauth2.client.registration.kakao.client-name=kakao
   spring.security.oauth2.client.registration.kakao.client-authentication-method=POST

   ## kAKAO Provider
   spring.security.oauth2.client.provider.kakao.authorization-uri=    https://kauth.kakao.com/oauth/authorize
   spring.security.oauth2.client.provider.kakao.token-uri=https://kauth.kakao.com/oauth/token
   spring.security.oauth2.client.provider.kakao.user-info-uri=https://kapi.kakao.com/v2/user/me
   spring.security.oauth2.client.provider.kakao.user-name-attribute=id
   ```

   카카오에서는 Provider의 대한 정보를 대략적으로...[https://developers.kakao.com/docs/latest/ko/reference/rest-api-reference](https://developers.kakao.com/docs/latest/ko/reference/rest-api-reference) 에서 제공을하기때문에 참고해서 더 많은 정보들을 가져오거나 뺄수 있을 것이다.

   실제로 위의 properties의 값들을 그대로 이용해도 적어도 지금 당장\(20.08.21\)까지는 별 문제가 없다.

   여기서 좀 오류가 많았는데, 실제로 token을 받아오는걸 실패하거나, 재대로 값이 매핑이 안되는 오류가 많았는데 일단 이 scope를 profile로 해야하고 또한 user-name-attribute를 id로 지정해야한다.

   실제로 유저 정보를 가져오는 json값은 대략 이렇다.

   ```javascript
   {
     "id": ----,
     "connected_at": "2020-08-15T16:08:16Z",
     "properties": {
       "nickname": "----",
       "profile_image": ""----",
       "thumbnail_image": "----"
     },
     "kakao_account": {
       "profile_needs_agreement": false,
       "profile": {
         "nickname": "---",
         "thumbnail_image_url": "----",
         "profile_image_url": "----"
       },
       "has_email": true,
       "email_needs_agreement": false,
       "is_email_valid": true,
       "is_email_verified": true,
       "email": "ventulus95@gmail.com",
       "has_age_range": false,
       "age_range_needs_agreement": false,
       "has_birthday": false,
       "birthday_needs_agreement": false,
       "has_gender": true,
       "gender_needs_agreement": false,
       "gender": "male"
     }
   }
   ```

   여기를 대충 보면 kakao\_account에 꽤 많은 정보가 저장되어있어서 이걸 끌고오려고 굳이 scope에 kakao\_account를 넣었다가 오히려 오류가 나서 왜그런지는 모르겠네요.. 오히려 profile로 하면 문제가 안생김. 차후에 더 알아보고 포스팅할게요.

2. 그리고 이미 예전에 만들어뒀던 코드인 OAuthAttributes에 추가를 합니다. kakao인 경우 ofKakao를 통해서 kakao의 유저 정보 response를 받아 올수 있게 한다. 참고로 response는 kakao\_account를 통해서 받아오는 값이지만, email의 경우 kakao account 내부에서 바로 뽑아서 사용가능하지만, nickname이나 img를 가져오기위해서는 profile를 긁어와야한다.

   그래서 response에서 profile을 get으로 받아와야지 그 값이 들어간다. \(Json Object식으로 되어있어서 그렇게 받아와야함.\)

```java
    public static OAuthAttributes of(String registrationId, String userNameAttributeName, Map<String, Object> attributes){
           if("naver".equals(registrationId)){
               return ofNaver("id", attributes);
           }
           if("kakao".equals(registrationId)){
               return ofKakao("id", attributes);
           }
           return ofGoogle(userNameAttributeName, attributes);
       }

       private static OAuthAttributes ofKakao(String userNameAttributeName, Map<String, Object> attributes) {
           Map<String,Object> response = (Map<String, Object>)attributes.get("kakao_account");
           Map<String, Object> profile = (Map<String, Object>) response.get("profile");
           return OAuthAttributes.builder()
                   .name((String)profile.get("nickname"))
                   .email((String)response.get("email"))
                   .Picture((String)profile.get("profile_image_url"))
                   .attributes(attributes)
                   .nameAttributeKey(userNameAttributeName)
                   .build();

       }
```

이렇게만해도 kakao로그인이 작동하게된다.

1. 그러면 프론트에서는 버튼을 만들어주기만하면...

   ![&#x1109;&#x1173;&#x110F;&#x1173;&#x1105;&#x1175;&#x11AB;&#x1109;&#x1163;&#x11BA; 2020-08-21 &#x110B;&#x1169;&#x110C;&#x1165;&#x11AB; 10.11.53](https://github.com/ventulus95/TIL/tree/76f05b8af8c692b1182e679132dbfffb4736169a/Users/LeeChnagSup/Desktop/스크린샷%202020-08-21%20오전%2010.11.53.png)

   ```markup
    <a href="/oauth2/authorization/kakao" class="btn btn-warning active" role="button">카카오 로그인</a>
   ```

   이렇게 인증 url을 만들어주고 a태그만 걸어주게 된다면, 스프링 시큐리티로 카카오로그인이 정상적으로 작동하게된다.

   첫 인증의 경우, 무슨 개인정보를 가져가는지 알려주는 창이 뜨는데 실제로 나 같은 경우는 인증을 하도 많이 받아서 바로 접속된다.

   ![&#x1109;&#x1173;&#x110F;&#x1173;&#x1105;&#x1175;&#x11AB;&#x1109;&#x1163;&#x11BA; 2020-08-21 &#x110B;&#x1169;&#x110C;&#x1165;&#x11AB; 10.18.21](https://github.com/ventulus95/TIL/tree/76f05b8af8c692b1182e679132dbfffb4736169a/Users/LeeChnagSup/Desktop/스크린샷%202020-08-21%20오전%2010.18.21.png)

   ![&#x1109;&#x1173;&#x110F;&#x1173;&#x1105;&#x1175;&#x11AB;&#x1109;&#x1163;&#x11BA; 2020-08-21 &#x110B;&#x1169;&#x110C;&#x1165;&#x11AB; 10.18.21](https://github.com/ventulus95/TIL/tree/76f05b8af8c692b1182e679132dbfffb4736169a/Users/LeeChnagSup/Desktop/스크린샷%202020-08-21%20오전%2010.18.36.png)

여기까지 카카오 소셜로그인에 대한 방법을 알아보았다. 실제로 이동욱님의 코드를 카카오 소셜로그인을 위해서 조금의 코드 수정만 있으면 여러가지 로그인 수단을 계속 덧붙일수 있을 것 같았다.

이상으로 Spring Boot Security로 카카오 소셜 로그인 만들기를 마치도록 하겠다.

