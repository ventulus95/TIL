# Spring boot & Redis를 통한 Cache 서버 구축하기

## 백엔드에서 캐시서버란...

백엔드 서버에서 캐시 서버를 둔다는 것은 가용성적인 측면에서 접근할 수 있다. 쉽게이야기하면, DB서버에서 쿼리를 한번이라도 더 쿼리를 덜 보낼수 있게된다. 

DB에 쿼리를 줄이는 방법을 통해서 실제 서버의 가용성을 높일수 있다. 

물론 어떤 데이터를 캐시로 둘것인가를 가정하는 것은 고민해봄직한 측면이지만, 개념적인 측면보다는 이번에는 캐시서버를 어떤식으로 구성할까에 더 집중하도록 하자.

## SpringBoot 레디스 서버 구축 참고 링크

https://deveric.tistory.com/98 : 프로젝트

https://yonguri.tistory.com/82 : Cache 어노테이션과 관련된 참고할만한 링크

다음과 같은 링크를 바탕으로 약간 변형해서 구현했다.  여기서 나는 H2 인메모리 DB를 바탕으로 실제 DB까지 구현해서 실제로 캐시가 있는 경우 쿼리를 날리는지 안날리는지 까지 확인 해보고 싶었다.

## Redis 준비

레디스 준비는 간단하게 Docker를 통해서 구성해봤다. 나는 kemetics를 통해서 쉽게 다운 받았다. 다음과 같이 설치하면 된다. 

![img](https://blog.kakaocdn.net/dn/bhTybP/btqReQ0tBrK/EOQy0DK6kF5g4zhVt2kGwk/img.png)

## Spring boot 준비

스프링 부트는 다음과 같은 의존성을 포함해야한다. 나는 Gradle로 진행하였다.

```gradle
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-data-redis'
    implementation 'org.springframework.boot:spring-boot-starter-web'
    implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
    compileOnly 'org.projectlombok:lombok'
    runtimeOnly 'com.h2database:h2'
    developmentOnly 'org.springframework.boot:spring-boot-devtools'
    annotationProcessor 'org.projectlombok:lombok'
    testImplementation 'org.springframework.boot:spring-boot-starter-test'
}
```

스프링부트는 대략적으로 다음처럼 구성된다.

- WEB
- JPA - H2 (H2 DB를 사용하기위한 JPA )
- Spring data-redis

1. Spring boot에 간단한 게시판 예제를 만들자. 

   Spring boot 게시판 예제는 밑에 깃허브 링크를 남기겠다. 

   https://github.com/ventulus95/SpringBoot-redis-Cache

2. TemplateRedisApplication.java에  @EnableCaching 추가

   ```java
   package com.example.templateredis;
   
   import org.springframework.boot.SpringApplication;
   import org.springframework.boot.autoconfigure.SpringBootApplication;
   import org.springframework.cache.annotation.EnableCaching;
   
   @EnableCaching
   @SpringBootApplication
   public class TemplateRedisApplication {
   
       public static void main(String[] args) {
           SpringApplication.run(TemplateRedisApplication.class, args);
       }
   
   }
   ```

3. Redis 설정에 관련된 여러가지 설정을 세팅을 해야한다. 

   ```java
   package com.example.templateredis;
   
   import com.fasterxml.jackson.databind.ObjectMapper;
   import org.springframework.beans.factory.annotation.Autowired;
   import org.springframework.beans.factory.annotation.Value;
   import org.springframework.context.annotation.Bean;
   import org.springframework.context.annotation.Configuration;
   import org.springframework.data.redis.connection.RedisConnectionFactory;
   import org.springframework.data.redis.connection.RedisStandaloneConfiguration;
   import org.springframework.data.redis.connection.lettuce.LettuceConnectionFactory;
   import org.springframework.data.redis.core.RedisTemplate;
   import org.springframework.data.redis.serializer.GenericJackson2JsonRedisSerializer;
   import org.springframework.data.redis.serializer.StringRedisSerializer;
   
   @Configuration
   public class RedisConfig {
   
       @Value("${spring.redis.port}")
       public int port;
   
       @Value("${spring.redis.host}")
       public String host;
   
       @Autowired
       public ObjectMapper objectMapper;
   
       @Bean //요친구가 실제로 Template 역할 Key Serializer, Value Serializer를 통해서 실제 데이터를 변환하는 과정이 필요하다. 
       public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory redisConnectionFactory){
           RedisTemplate<String, Object> redisTemplate = new RedisTemplate<>();
           redisTemplate.setKeySerializer(new StringRedisSerializer());
           redisTemplate.setValueSerializer(new GenericJackson2JsonRedisSerializer()); //이친구 덕에 객체가 json형태로 변환되는 것으로 생각된다. 
           redisTemplate.setConnectionFactory(redisConnectionFactory);
           return redisTemplate;
       } 
   
       @Bean //이친구는 Connect와 관련된 객체이다. 보면 HostName Port등을 지정하고 Lettuce까지 지정.
       public RedisConnectionFactory redisConnectionFactory(){
           RedisStandaloneConfiguration redisStandaloneConfiguration = new RedisStandaloneConfiguration();
           redisStandaloneConfiguration.setHostName(host);
           redisStandaloneConfiguration.setPort(port);
           LettuceConnectionFactory connectionFactory = new LettuceConnectionFactory(redisStandaloneConfiguration);
           return connectionFactory;
       }
   
   
   }
   ```

4. 다음은 캐시와 관련된 CacheManager에 대해서 설정을 해줘야한다.

   ```java
   package com.example.templateredis;
   
   import com.fasterxml.jackson.databind.ObjectMapper;
   import org.springframework.beans.factory.annotation.Autowired;
   import org.springframework.cache.CacheManager;
   import org.springframework.context.annotation.Bean;
   import org.springframework.context.annotation.Configuration;
   import org.springframework.data.redis.cache.RedisCacheConfiguration;
   import org.springframework.data.redis.cache.RedisCacheManager;
   import org.springframework.data.redis.connection.RedisConnectionFactory;
   import org.springframework.data.redis.serializer.GenericJackson2JsonRedisSerializer;
   import org.springframework.data.redis.serializer.RedisSerializationContext;
   import org.springframework.data.redis.serializer.StringRedisSerializer;
   
   import java.time.Duration;
   
   @Configuration
   public class CacheConfig {
   
       @Autowired
       ObjectMapper objectMapper;
   
       @Autowired //아까 등록해뒀던 캐쉬팩토리 
       RedisConnectionFactory connectionFactory;
   
       @Bean
       public CacheManager cacheManager(){
           RedisCacheConfiguration redisConfiguration = RedisCacheConfiguration.defaultCacheConfig()
                   .serializeKeysWith(RedisSerializationContext.SerializationPair.fromSerializer(new StringRedisSerializer()))
                   .serializeValuesWith(RedisSerializationContext.SerializationPair.fromSerializer(new GenericJackson2JsonRedisSerializer())) 
                   .entryTtl(Duration.ofSeconds(30)); //TTL 적용도 가능하다. 
   
           RedisCacheManager redisCacheManager = RedisCacheManager.RedisCacheManagerBuilder.fromConnectionFactory(connectionFactory) //Connect 적용하고
                   .cacheDefaults(redisConfiguration).build();  //캐쉬설정과 관련된 것을 여기에 적용. 
           return redisCacheManager;
       }
   }
   ```

4, 다음은 캐시 적용에 대해서 설명

```java
//Service 파일의 한 메소드. 여기서 직접 캐싱하도록 지정한다. 
@Cacheable(key = "#id", value="findOne") //cacheable을 붙혀주면 캐쉬가 없으면 넣고 있으면 조회하는 방식으로 작동한다. 
public Board findOne(Long id){
    return boardRepository.findById(id).get();
}
```

이게 캐쉬로 작동하는지 확인해보도록 하겠다. 

알아보는 방식은 Redis안에 같은 파일이 있는지?, Cache가 있으면 SQL 작동을 하지 않을꺼니까 show-sql을 통해서 쿼리문이 작동여부를 직접 눈으로 확인하는 방식으로 했다.

일단 데이터를 하나 넣고, 1번 아이디를 가지는 게시물이 있는지 확인하자.

![img](https://blog.kakaocdn.net/dn/c8CiTq/btqQ8N4tbgw/oedoX6BlUsTcUJeB6LdA81/img.png)

찾으면 다음처럼 나오게 된다.  처음에는 SQL을 통해서 SELECT 되게 된다. 처음 검색하는 경우는 쿼리문을 통해서 검색이 되지만...

![img](https://blog.kakaocdn.net/dn/ct1TI7/btqRje65RkR/NKayav5ky3sDUQWJPZhEG1/img.png)

두번째부터는 쿼리가 실행되지 않았다. 물론 30초 동안만이다. 일단,  redis에도 키가 있는걸 확인해보자

![img](https://blog.kakaocdn.net/dn/cu6cPi/btqRcHvY2KB/j5kpsb8wNVp2zyzlMtgt9K/img.png)

사진찍다가 TTL 시간때문에 삭제되긴했지만... 다시 확인을 하면

![img](https://blog.kakaocdn.net/dn/cI0Nap/btqRifrCxzb/Alx6t4iVL4zeqRsIZL9Ax0/img.png)

이렇게 들어가져있는지를 확인할 수 있다. 영어같은 경우 문제 없이 들어가지만, 한글은 깨져서 들어간다.그리고 아까 Cache Manager에서 TTL 적용이 됬는지 확인하려먼 다음과 같이 확인한다. 

> TTL이란? 
> 캐쉬에서 언제 데이터가 사라질지를 지정하는 시간. Redis에선 시간을 초로 지정할 수 있게 된다. 

Redis-CLI에서 TTL [key값]을 입력해서 확인하자. 

![img](https://blog.kakaocdn.net/dn/cI0Nap/btqRifrCxzb/Alx6t4iVL4zeqRsIZL9Ax0/img.png)

그러면 다음과 같이 내가 지정해둔 30초에 맞춰서 시간이 흘러간다. 30초가 지나면, 알아서 Redis에서 사라지게된다. 



## 트러블 슈팅

1. kitematic을 통해 Redis를 설치하면 port 명이 안맞는 오류가 있다.

   ![img](https://blog.kakaocdn.net/dn/wrxEd/btqRgjgY0lB/EjDtbW40kzYjZoWoBc8530/img.png)

내부망은 6379를 사용하지만, 외부 즉 현재 내 컴퓨터랑 연동하기위해서는 저기 적혀있는 55001로 연동을 해야한다. 물론 뭐 이걸 6379로 바꾸고 할 수 도 있음. 아마 Docker를 통해서 실행할 경우는 상관없는 문제이다.

2. TTL이 재대로 적용안되는 이슈

   application.yml나 properties에 

   `spring.cache.redis.time-to-live=` 을 통해서 TTL 적용하면 TTL이 적용이 재대로 안되는 이슈가 있음. 왜그런지는 모르겠는데 TTL이 시간이 만료되도 안사라지는 -1 상태 (키는 남아있는데 TTL은 끝난 상태)로 유지됨. 

   CacheManager를 통해서 TTL을 직접 설정하는게 확실하다. 정확하게 왜 Properties나 yaml파일을 통해서 설정하는 것이 안되는건  왜그런지는 모르겠음..ㅋㅋ



이런식으로 구성하게되면, Spring Boot를 이용하면서 Cache서버를 구성하게 된다.

