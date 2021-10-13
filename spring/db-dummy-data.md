# DB Dummy data 자동 삽입하기

## Spring에서 DB를 자동으로 삽입하게 하려면...

![](../.gitbook/assets/image.png)

Spring의 In memory-DB인 H2를 활용해서 코딩을 하다보면,서버를 한번 끄고 키는 동안 그안의 모든 데이터가 날라가게된다.

Test 용도로는 훌륭하지만,끌때마다 아까 넣었던 데이터를 다시 삽입하고 넣고...확인하는 과정을 거치는것은 여간 귀찮은 것이 아니다.

Spring에서는 자동으로 SQL문을 넣어주는 기능이 세팅되어있다.

## SQL문의 자동 삽입

방식은 이런식으로 작동되어진다.내가 따로 설정을 하지 않아도,`resources `에서 `data.sql`,`schema.sql`는 자신이 사용하고 있는 DB에 맞춰서 서버가 켜짐에 따라서 자동으로 `data.sql`은 DML (Insert문) `schema.sql`은 DDL(Create)문을 자동으로 적용시켜준다.

![요론 느낌쓰.](<../.gitbook/assets/스크린샷 2021-04-30 오후 4.02.17.png>)

더 나아가서, Spring Data JPA는 DB의 변경에 따른 SQL문의 변경 상황도 고려하여 플렛폼에 따라서 SQL문 역시 작동시킬 수 있다.

`schema-${platform}.sql`, `data-${platform}.sql` 와 같은 형태로 DB플렛폼에 따라서 변경할 수 있으며, 그 설정은 application.properties에서 혹은 yaml파일에서 DB 플렛폼 명에 따라서 작동할 수 있다. -> `spring.datasource.platform` = `hsqldb`, `h2`, `oracle`, `mysql`, `postgresql` 등등...

참고로 Schema의 경우 Hibernate에서 설정을 따로 하지 않으면 자동으로 DDL이 삽입되게 되있어서, 만약 자신만의 스키마를 따로 잡아줘야하는 경우에는 반드시 옵션을 설정해줘야한다.

`spring.jpa.generate-ddl` 나 `spring.jpa.hibernate.ddl-auto` 를 설정을 바꿔줘야지 동시에 삽입이 되지 않아, 오류를 발생할 수 있는 구석을 더 좁힐 수 있다.

{% hint style="success" %}
좀 더 구체적인 DB 삽입과 관련된 옵션들이 있는데, `spring.datasource.continue-on-error` 는 데이터가 이상하게 넣어져서 오류가 발생하더라도 그냥 진행 시키게 하는 방법이 있다.

`spring.datasource.sql-script-encoding=UTF-8` `data.sql`을 활용하면서 인코딩 옵션을 따로 줄 수 있는데 검색을 많이해본 결과, 한글 옵션을 재대로 주지 않는다면, 깨져서 들어가서 인코딩 옵션을 줘서 안깨지게 넣는 방법도 있었다.
{% endhint %}

더 나아가서, Spring Data JPA는 DB의 변경에 따른 SQL문의 변경 상황도 고려하여 플렛폼에 따라서 SQL문 역시 작동시킬 수 있다.

`schema-${platform}.sql`, `data-${platform}.sql` 와 같은 형태로 DB플렛폼에 따라서 변경할 수 있으며, 그 설정은 application.properties에서 혹은 yaml파일에서 DB 플렛폼 명에 따라서 작동할 수 있다. -> `spring.datasource.platform` = `hsqldb`, `h2`, `oracle`, `mysql`, `postgresql` 등등...

참고로 Schema의 경우 Hibernate에서 설정을 따로 하지 않으면 자동으로 DDL이 삽입되게 되있어서, 만약 자신만의 스키마를 따로 잡아줘야하는 경우에는 반드시 옵션을 설정해줘야한다.

`spring.jpa.generate-ddl` 나 `spring.jpa.hibernate.ddl-auto` 를 설정을 바꿔줘야지 동시에 삽입이 되지 않아, 오류를 발생할 수 있는 구석을 더 좁힐 수 있다.

힌트: 좀 더 구체적인 DB 삽입과 관련된 옵션들이 있는데, `spring.datasource.continue-on-error` 는 데이터가 이상하게 넣어져서 오류가 발생하더라도 그냥 진행 시키게 하는 방법이 있고,

`spring.datasource.sql-script-encoding=UTF-8` `data.sql`을 활용하면서 인코딩 옵션을 따로 줄 수 있는데 검색을 많이해본 결과, 한글 옵션을 재대로 주지 않는다면, 깨져서 들어가서 인코딩 옵션을 줘서 안깨지게 넣는 방법도 있었다.

## TestCode에서 Data.sql의 삽입

저것만으로도 큰 수확이었으나, TDD를 위해서 JPA TestCode를 작성중이었는데 오류가 지속적으로 발생했다.

@DataJpaTest 어노테이션을 삽입하고 실행했을 경우 자꾸 이런 오류가 발생했다.

```java
Caused by: org.h2.jdbc.JdbcSQLSyntaxErrorException: Syntax error in SQL statement  ENGINE=[*]INNODB"; expected "identifier"; SQL statement:
```

왜 이런 오류가 발생했느냐 확인해보니까

Application.yaml 의 dialect의 조건을 mysql engine을 innodb로 줬기때문에 자꾸 DDL문이 ENGINE=INNODB 과 같은 오류를 만들어 냈다.

```yaml
spring:
    properties:
      hibernate:
        dialect: org.hibernate.dialect.MySQL5InnoDBDialect
        storage_engine: innodb # <-따지자면 이친구가 문제였다.
  datasource:
    hikari:
      jdbc-url: jdbc:h2:mem://localhost/~/testdb;MODE=MYSQL
    continue-on-error: true
```

datasource는 분명히 재대로 MODE=MYSQL인데, 도대체 왜 안되는가 했더니만, 공식문서와 어노테이션을 타고 타고 가보면 다음처럼 되어있다.

```java
@AutoConfigureTestDatabase //이 어노테이션의 함수인
// ㄴ 밑의
EmbeddedDatabaseConnection connection() default EmbeddedDatabaseConnection.NONE; // 이 임베디드Connection의 조건중에 H2옵션을 확인해보면
//ㄴ 안의
H2(EmbeddedDatabaseType.H2, DatabaseDriver.H2.getDriverClassName(),
      "jdbc:h2:mem:%s;DB_CLOSE_DELAY=-1;DB_CLOSE_ON_EXIT=FALSE", (url) -> url.contains(":h2:mem")),
```

H2 옵션이 자동설정될 것이다. (뭐 내가 설정을 따로하지않으면, 자동으로 AutoConfiguration에 의해서 h2로 잡힐 것이다. ) 근데 잘 JDBC 조건에는 MYSQL조건이 없으므로 dialect가 MySQL5InnoDBDialect 이라면 당연히 스키마 적용시 오류가 나고 DML 부분에서도 오류가 발생한 것이다.

@DataJPATest를 하면 자동으로 `@AutoConfigureTestDatabase` 가 적용이 되니까 우리는 무슨 수를 써도 H2인 경우에는 H2의 옵션 변경이 안된다는 뜻이기도 한데...

물론 Spring개발자들이 그렇게 빡빡하게 굴리는 없지않은가? 내가 테스트용 DB를 H2로 고정할건 아닐텐데, 다른 DB를 사용하는 경우에 대한 대첵도 역시 마련되어있다.

공식문서: [https://docs.spring.io/spring-boot/docs/current/reference/html/howto.html#howto-database-initialization](https://docs.spring.io/spring-boot/docs/current/reference/html/howto.html#howto-database-initialization)

### TestDB를 실제 DB로 설정하는 방법

`@AutoConfigureTestDatabase` 를 잘 둘러본다면, 다음과 같은 교체가능한 Enum이 존재한다.

```java
Replace replace() default Replace.ANY;
enum Replace {
    ANY,
    AUTO_CONFIGURED,
    NONE
  }
```

기본적으로 ANY가 선택되있고, ANY는 NONE, AUTO_CONFIGURED 중 하나 선택하는것인데, 거의 대부분 AUTO로 인메모리를 선택할것이기때문에 우리는 반드시 NONE을 선택해야지만, url 옵션들이 사라진채로 우리가 집어넣는 DB옵션 값을 받아서 DB Testing을 진행시켜준다.

이러한 DB옵션들은 당연히 Application.yaml이나 properties에서 DB관련 정보들을 넣을 수 있다.

`@ActiveProfiles("....")` 을 따로 지정해놓지 않으면, 기본으로 설정되어있는 application.yaml으로 붙어서 처리되기 때문에 공용으로 사용되어지는 옵션으로 처리가 되어서 나 같은 경우는 차라리 Test용 application.yaml을 따로 만들고 따로 profile을 지정해서 `@ActiveProfiles("....")`을 통해서 활성화시키는 방향으로 바꾸었다.

공식문서: [https://docs.spring.io/spring-boot/docs/current/reference/html/spring-boot-features.html#boot-features-testing-spring-boot-applications-testing-autoconfigured-jpa-test](https://docs.spring.io/spring-boot/docs/current/reference/html/spring-boot-features.html#boot-features-testing-spring-boot-applications-testing-autoconfigured-jpa-test)

{% hint style="info" %}
@DataJpaTest는 @Transactional 어노테이션이 들어있어서 테스트 기능중에도 테스트가 끝나면 자동 RollBack되게 되있다.아래처
{% endhint %}

```bash
2021-04-30 16:57:37.854  INFO 85053 --- [    Test worker] o.s.t.c.transaction.TransactionContext   : Rolled back transaction for test:
```

{% hint style="info" %}
spring.jpa.show-sql 역시 true처리되어있어서 따로 설정해주지 않아도 자동으로 확인가능하다.
{% endhint %}

### 마무리

TestCode를 작성하려다보니 확실히 얻는 점이 많아졌다. 내가 알지 못했던 것들에 대해서 재대로 알고 넘어갈 수 있어서 좋았던것 같아 정리도 좀 빡세게 진행해봤다.

### 출처

블로그: [https://kangwoojin.github.io/programing/auto-configure-test-database/](https://kangwoojin.github.io/programing/auto-configure-test-database/)

[https://pravusid.kr/java/2018/10/10/spring-database-initialization.html](https://pravusid.kr/java/2018/10/10/spring-database-initialization.html)

[https://github.com/HomoEfficio/dev-tips/blob/master/Spring%20%40DataJpaTest%20%EC%82%AC%EC%9A%A9%20Tips.md](https://github.com/HomoEfficio/dev-tips/blob/master/Spring%20%40DataJpaTest%20%EC%82%AC%EC%9A%A9%20Tips.md)
