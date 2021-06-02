# SpringBoot와 MongoDB를 활용한 간단한 프로젝트 만들어보기

## MongoDB를 활용해보고 싶다...

Spring boot는 MongoDB를 JPA를 활용할 수 있는데, MongoDB의 가장 큰 장점이라고 알려져 있는 Geolocation한 데이터 타입을 활용하는 예제를 만들려면, 실제로 MongoDB를 활용하는 것이 좋아보인다. 

[[잡담/관심 가는 기술들\] - if-kakao MongoDB 세션 정리](https://sundries-in-myidea.tistory.com/106) 

실제로 카카오모빌리티에서는 MongoDB를 활용한 사례가 있으므로, 위치 정보를 바탕으로 하는 프로젝트를 만들기에는 MongoDB만한 것이 없을 것이다!

## SpringBoot와 MongoDB를 연동해보자

MongoDB를 활용하는 방법은 그렇게 어렵지 않다. 일단 기본적으로 MongoDB부터 실행시켜보자. Local에 설치할 수도 있겠지만 Docker를 활용한 방법을 사용했었다.

Docker의 설치가 됬다는 전제조건하에 실행해보자

`docker run -p 27017:27017 --name mongo_springboot -d mongo` 로 다운로드하고

`docker exec -i -t mongo_springboot bash` 로 배쉬셀 실행후, `mongo`로 몽고DB를 실행시켜준다. 

그리고 새로 SpringBoot 프로젝트를 생성해줍니다. 저는 Gradle로 프로젝트를 실행했다.

다음과 같은 의존성을 새팅해두었다.

```
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-data-mongodb'
    implementation 'org.springframework.boot:spring-boot-starter-web'
    compileOnly 'org.projectlombok:lombok'
    developmentOnly 'org.springframework.boot:spring-boot-devtools'
    implementation 'org.springframework.boot:spring-boot-starter-thymeleaf'
    annotationProcessor 'org.projectlombok:lombok'
    testImplementation group: 'de.flapdoodle.embed', name: 'de.flapdoodle.embed.mongo', version: '2.2.0'
    testImplementation 'org.springframework.boot:spring-boot-starter-test'
}
```

`testImplementation group: 'de.flapdoodle.embed', name: 'de.flapdoodle.embed.mongo', version: '2.2.0'` 는 mongoDB의 테스트코드를 작성해보고 싶어서 만든 의존성임으로, 꼭 추가하지 않아도 된다



## MongoDB와 JPA를 활용해서 삽입하는 예제를 만들어보자

JPA를 사용하기위해 DB 스키마를 만들어야 한다. 물론, MongoDB에는 스키마가 따로 없으므로 비슷한 역할을 하는 Document를 활용할 수 있다!

```java
package com.ventulus95.mongodb_springboot.account;

import lombok.Data;
import org.springframework.data.annotation.Id;
import org.springframework.data.mongodb.core.geo.GeoJsonPoint;
import org.springframework.data.mongodb.core.mapping.Document;

@Document(collection = "account")
@Data
public class Account {

    @Id
    private String id;

    private String username;

    private String email;

    private GeoJsonPoint location;

}
```

원래 다른 예제와 비슷한 모습으로 구성하고, 특이하게 한개의 특이한 지표가있는데, GeoJsonPoint 이것이 바로 현재 자신의 위경도를 저장할 수 있는 데이터 타입이다

JPA를 활용할 수 있게 Repository 인터페이스를 만들고 

```java
public interface AccountRepository extends MongoRepository<Account, String> {
```

Controller에 다음과 같은 방식으로 만들었다.

```java
package com.ventulus95.mongodb_springboot.account;

import lombok.AllArgsConstructor;
import org.springframework.data.mongodb.core.geo.GeoJsonPoint;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.*;

@Controller
@AllArgsConstructor
public class AccountController {

    private final AccountRepository accountRepository;

    @PostMapping("/account")
    @ResponseBody
    public String makeAccount(@RequestBody GeoDto geoDto){
        Account temp = new Account();
        temp.setUsername("새로운 글!");
        temp.setLocation(new GeoJsonPoint(Double.parseDouble(geoDto.getLatitude()), Double.parseDouble(geoDto.getLongtitude())));
        return accountRepository.save(temp).getId();
    }

    @GetMapping("/account1")
    public String page(){
        return "account";
    }
}


//GeoDto는 이렇게 구성된다.

import lombok.Data;

@Data
public class GeoDto {
    private String latitude;
    private String longtitude;

}

```

Controller의 경우 `GET` 메소드로는 현재 위치를 받아올수 있는 버튼이 있는 페이지로 이동시키고 `POST`로 이 현재 위치를 전달하여 저장하는 방식으로 구성해보았다.

버튼만 누르면, userName은 새로운 글이라는 정보+ GeoJsonPoint로  현재위치 데이터가 들어갈 수 있게 만들었다. 내 현재 위치를 저장할 수 화면을 구성해보자.



## HTML 5 Geolocation을 활용하여 현재위치를 받아오자

JavaScript에서 navigator.geolocation.**getCurrentPosition()** 를 활용하면, 현재위치를 받아올 수 있다.  

```javascript
let k 
navigator.geolocation.getCurrentPosition(function(pos) {
    var latitude = pos.coords.latitude;
    var longitude = pos.coords.longitude;
    alert("현재 위치는 : " + latitude + ", "+ longitude);
    k = {latitude:latitude+"", longtitude:longitude+""};
});
```

일단, 가장 단순하게 script를 이런식으로 짜보았다. 처음에 /account1을 통해서 페이지로 이동하면 브라우저의 위치정보 인식을 통해서 현재 위치를 받아와서 위 경도를 가저온다. 그리고 alert로 화면에 띄워준다. 

그리고 이 데이터를 k에 JSON 형태로 구성한다. `latitude+""`처럼 구성되어 있으나 꼭 String타입으로 만들어주지 않아도 가능하다. 그리고 이 데이터 K를 Post로 전송할 것이다. 

나는 POST 전송을 쉽게 어떻게 하지 고민하다가, 제일 처음 떠오른것은 AJAX라 AJAX를 사용했는데 이 방식이외에도 여러가지 방식이 있다고 생각하여 3가지정도 구성했다.  이글을 읽는 분들에게는  async를 활용한, await fetch 방식으로 전송하는 것을 보여줄 것이다. 

나머지 두개의 예제는 접은글에 써보겠다. 나의 Javascript에 대한 지식을 좀 정리겸 써놓은 글이니 신경은 안써도 될것 같다.

input button을 하나 만들고 거기에 onclick메소드로 test라는 함수를 만들것이다.

```javascript
async function test(){
    // async await fetch로도 구성해보기.
    const response = await fetch("/account", {
        method: "POST",
        headers: {
            "Content-Type" : "application/json",
        },
        body: JSON.stringify(k)
    })

    console.log(await response.text())
}
```

fetch는 Promise 형으로 구성되어서 .then을 쓰고 싶지는 않아서 await를 활용한 예제다. 

POST형으로 전달하고, JSON.stringify()로 아까 받아왔던 데이터를 넣어서 body형으로 보낸다.  Controller에서 보았듯, @ResponseBody를 통해서 id값을 받아올 예정이다. 그리고 그 값은 text로 전달될 예정이라 .text()로 변환하여 console.log()로 노출 시킬 것이다.

[접은글]

AJAX의 경우 사용법이 바로 떠올랐다. 대신  Jquery에 종속되는 경향이 있다.  레거시로 사용하는 경우가 많기때문에 예전 인턴하던 기억이 나서 ajax를 통해 데이터를 전달하는 예제를 만들어 보았다.

```javascript
$.ajax({
    url: "/account",
    data: JSON.stringify(k),
    type : 'POST',
  	dataType: 'json',
		contentType : "application/json",
    success: function(e) {
        alert("성공~~~")
    },
    fail: function(e){
        alert("실패했습니다 오류코드는:"+e);
    }
});
```

근데 생각해보니까 JS도 버전업에 따라서 분명히 API를 전송하거나 전송받는 함수가 있겠지 생각했다. (난 지금 당장 노드로 개발중인데, 이런게 없을리 없잖아..) 그래서 찾아본 방식은 fetch이었는데, 나름 사용 편의성이 있었다.

```javascript
//fetch로도 구성해보기.
fetch("/account", {
    method: "POST",
    headers: {
        "Content-Type" : "application/json",
    },
    body: JSON.stringify(k)
}).then((response) => console.log(response))
```

대신 Promise 객체로 전달되기때문에, .then과 같은 방식으로 처리를 한번 해줘야한다. 문제는 이러한 .then이 콜백처럼 점점 많아지는 것을 보기 싫을 것 같아, `await`를 활용해보았다. 

[접은글]

## 구현된 프로젝트를 확인해보기

