웹서비스를 이용하다보면, 다음과 같은 메일을 받아봤을것이다. 이런 것을 한번 구현해보고 싶어서 만들어보았다.



![img](https://blog.kakaocdn.net/dn/cLUkcA/btqTKLCmdQE/m0iQM1edug7IEKOUaWEKY0/img.png)![img](https://blog.kakaocdn.net/dn/pPZPq/btqTKMVz4Go/G9SvFevxeciJh5zkGclNYk/img.png)

###  

### 회원가입시 이메일에 대한 검증이 필요합니다.

Spring boot를 통해서 개발을 할때, user 등록시 유저이름을 email로 하거나, 혹은 id로 하는 경우가 있는데. 물론 이메일 인증의 경우 이메일의 양식만 맞으면 대부분 회원가입이 될 가능성이 큽니다. (그래서 FE 검증, BE 검증이 필요한 것이겠지만...)



> 예를 들면 [test@test.com](mailto:test@test.com) 이라던지.. [Admin@admin.com](mailto:Admin@admin.com) 처럼 없는 이메일을 써서 회원가입을 처리할 수 있다



그러면 이런 있을수 없는 이메일을 확인하는 방법은 이메일 인증체계를 만드는 것이다. 즉, 백엔드에서 이메일과 관련된 서비스를 제공할 수 있어야 한다는 것이다.

어떤식으로 구성해야할까? 일단 이메일 인증전에, 기본적으로 이메일을 보낼 수 있게끔 만들어야한다.

나는 View만들기가 귀찮아서 API를 통해서 전송할 수 있는 구조로 만들어 보았다.



## 필요한 의존성

```
implementation 'org.springframework.boot:spring-boot-starter-mail'
implementation 'org.springframework.boot:spring-boot-starter-web'
compileOnly 'org.projectlombok:lombok'
developmentOnly 'org.springframework.boot:spring-boot-devtools'
```

`org.springframework.boot:spring-boot-starter-mail`가 이메일과 관련된 프로세스를 담당한다.



나는 이메일 서비스는 Gmail을 이용할 것이므로 간단한 설정을 해줘야한다. https://myaccount.google.com/lesssecureapps에서 **"보안 수준이 낮은 앱 허용: 사용"**으로 변경해줘야한다.



그리고 이런 이메일을 어떤걸 사용해서 이메일을 보낼 정보가 필요하다. 이러한 기본 정보를 필요로 하는데, 그런 정보는 Application.yml 혹은 properties를 통해서 추가해줘야한다.

```
spring:
  mail:
    host: smtp.gmail.com
    port: 587
    username: 구글 아이디
    password: 구글 비번
    properties:
      mail:
        smtp:
          auth: true
          starttls:
            enable: true
```

물론, 꼭 gmail만 고집할 필요는 없다. smtp 서버는 여러가지이므로 그에 맞춰서 진행하면 된다.

나는 다음과 같은 글들을 참고 하였다.

https://victorydntmd.tistory.com/342

[[SpringBoot\] 이메일 전송 ( JavaMailSender, MimeMessageHelper )이번 글에서는 MailSender 인터페이스를 상속받은 JavaMailSender를 사용하여 이메일 전송 시스템을 구현해보도록 하겠습니다. 전체 코드는 깃헙을 참고하시길 바랍니다. 개발환경 IntelliJ 2019.02 Java 11victorydntmd.tistory.com](https://victorydntmd.tistory.com/342)

##  단순한 텍스트만 포함된 이메일을 보내기

Service단

```
package com.ventulus95.springsecurityemail;

import lombok.AllArgsConstructor;
import org.springframework.mail.SimpleMailMessage;
import org.springframework.mail.javamail.JavaMailSender;
import org.springframework.stereotype.Service;

import java.util.Random;

@Service
@AllArgsConstructor
public class MailService {

    private JavaMailSender javaMailSender;
    private static final String FROM_ADDRESS = "이메일에 보낼 주소";

    public void mailSend(MailDto mailDto){
        SimpleMailMessage  message = new SimpleMailMessage();
        message.setTo(mailDto.getAddress());
        message.setSubject(mailDto.getTitle());
        message.setText(mailDto.getMessage());
        javaMailSender.send(message);
    }
}
```

DTO는 맘대로 구성해도 된다. 차후 적용할 방식때문에 MultipartFile file을 적용했다.

```
package com.ventulus95.springsecurityemail;

import lombok.Getter;
import lombok.NoArgsConstructor;
import lombok.Setter;
import org.springframework.web.multipart.MultipartFile;

@Getter
@Setter
@NoArgsConstructor
public class MailDto {
    private String address;
    private String title;
    private String message;
    private MultipartFile file;
}
```

컨트롤러단은 api통신으로 처리할 수 있게 아예 만들어보았다.

```
package com.ventulus95.springsecurityemail;

import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.ResponseBody;

import java.util.Map;

@Controller
@RequiredArgsConstructor
public class MailController {
    private final MailService mailService;

    @PostMapping("/mail")
    public void execMail(MailDto mailDto) {
        mailService.mailSend(mailDto);
    }

}
```

이렇게 구성하면 다음과 같은 방식으로 보낼수 있게된다.

![img](https://blog.kakaocdn.net/dn/cHmbXy/btqTLxDWSmA/bF2uGsawyx56MeOMT2b8ik/img.png)

나의 경우 Postman을 사용해서, body의 form-data를 통해서 파일을 보냈지만, Controller를 다음과 같이 변경하면 이런식도 가능하다.

```
    @PostMapping("/mail")
    public void execMail(@RequestBody MailDto mailDto) {
        mailService.mailSend(mailDto);
    }
```

![img](https://blog.kakaocdn.net/dn/bIsfaH/btqTNAtBM7r/tEWJ86sgJdiBx5h3BPYu40/img.png)



이렇게 POST를 보내면 다음과 같이 메일을 받아볼수 있다.

![img](https://blog.kakaocdn.net/dn/bvbpZ6/btqTJmQDVNk/MIC5VMkF72STPyzyHprG9k/img.png)

##  첨부파일 포함한 이메일

사실 그냥 이메일만 오는 경우는 거의 드물다 어느정도의 HTML의 포멧을 맞춘 이메일이 오면 더 단정한 이메일 받아 볼수 있다. 그러면 어떤식으로 이메일을 구성해야하는지를 확인해보겠다.

첨부파일을 포함한 경우 다음과 같이 변동이 생긴다.

1. service단의 SimpleMailMessage를 사용할 수 없다. -> 다른 MailHandler를 이용해서 처리한다.
2. raw방식으로는 파일 전송이 안된다. -> POST시 body는 form-data타입으로 보내야한다.

Service는 다음과 같이 변경해야한다.

```
public void mailSend(MailDto mailDto){
    try {
        MailHandler mailHandler = new MailHandler(javaMailSender);
        mailHandler.setTo(mailDto.getAddress());
        mailHandler.setSubject("인증메일입니다.");
        String htmlContent = "<p>" + mailDto.getMessage() +"<p> <img src='cid:sample-img'>";
        mailHandler.setText(htmlContent, true);
        mailHandler.setAttach(mailDto.getFile().getOriginalFilename(), mailDto.getFile());
        mailHandler.setInline("sample-img", mailDto.getFile());
        mailHandler.send();
    }
    catch (Exception e){
        e.printStackTrace();
    }
}
```

mailHandler.java를 생성해야한다

```
package com.ventulus95.springsecurityemail;

import org.springframework.core.io.ByteArrayResource;
import org.springframework.core.io.InputStreamResource;
import org.springframework.mail.javamail.JavaMailSender;
import org.springframework.mail.javamail.MimeMessageHelper;
import org.springframework.web.multipart.MultipartFile;

import javax.activation.DataHandler;
import javax.activation.FileDataSource;
import javax.mail.MessagingException;
import javax.mail.internet.MimeBodyPart;
import javax.mail.internet.MimeMessage;
import java.io.ByteArrayInputStream;
import java.io.IOException;
import java.io.InputStream;

public class MailHandler {
    private JavaMailSender sender;
    private MimeMessage message;
    private MimeMessageHelper msgHelper;

    public MailHandler(JavaMailSender sender) throws MessagingException {
        this.sender = sender;
        message = sender.createMimeMessage();
        msgHelper = new MimeMessageHelper(message, true, "UTF-8");
    }

    public void setFrom(String fromAddress) throws MessagingException {
        msgHelper.setFrom(fromAddress);
    }

    public void setTo(String email) throws MessagingException {
        msgHelper.setTo(email);
    }

    public void setSubject(String subject) throws MessagingException {
        msgHelper.setSubject(subject);
    }

    public void setText(String text, boolean useHtml) throws MessagingException {
        msgHelper.setText(text, useHtml);
    }

    public void setAttach(String displayFileName, MultipartFile file) throws MessagingException {
        msgHelper.addAttachment(displayFileName, file); 
    }

    public void setInline(String contentId, MultipartFile file) throws MessagingException, IOException {
        msgHelper.addInline(contentId, new ByteArrayResource(file.getBytes()), "image/jpeg");
    }

    public void send() {
        try {
            sender.send(message);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

이코드 자체는 사실 [victolee](https://victorydntmd.tistory.com/)님의 코드와 대부분 같은데, 내가 차별점을 조금 준 부분은 setAttach와 setInline이다. victolee님의 경우 서버 내부의 있는 정적파일을 바로 전송하는 경우지만, 나는 client에서 보낸 파일을 바로 전송한다.



setInline은 파일을 본문에 추가하는 경우 필요한 것이다. setAttach는 이메일에 첨부파일을 얹어서 처리하는 경우다.

그래서 service단의 `mailHandler.setAttach(mailDto.getFile().getOriginalFilename(), mailDto.getFile());` 을사용하는 경우 이 파일의 파일명과 함께 전송되는데, 만약 paramater의 첫번째 이름을 바꾸면 파일명자체도 바뀐다. 그래서 확장자명까지 적어줘야한다.



이런상태에서 `POST /mail` 을 전송하면, 다음과 같이 이메일이 온다.

![img](https://blog.kakaocdn.net/dn/rcR6j/btqTJTOdMQV/F9OOAE4bNukfuJB87KYzo1/img.png)

HTML을 사용해서 더 명확하게 보여주고 싶다면, `mailHandler.setText(htmlContent, true);` 전달할 내용을 html으로 작성하면 알아서 잘 작성되어진다.



## 끝으로

이 상태에서 Spring Security와 함께, 회원가입인증 절차도 만들 수도 있다만. 당장은 하지않을 것 같다. 왜냐면 너무 귀찮아서... 현재 내 깃허브에는 엄청 대충 만든 이메일 인증 절차가 있으므로, 그것에 대해서 언급하지는 않겠다. 링크를 통해서 보는 정도만 할것이다.



https://github.com/ventulus95/spring-security-email



Spring Security를 통해서 구현한 방식도 아니고 단순히 인증번호를 받고-> 인증번호를 인증하는 식으로 구현해놨고 Static을 써놨기때문에 **정말정말 위험한 코드**이다. 그 점을 알고 개발하면 좋을 것