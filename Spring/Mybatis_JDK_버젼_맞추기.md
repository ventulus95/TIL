# Mybatis JDK 버젼 맞추기

Spring 프로젝트를 받고 난뒤에, 서버를 켰는데 오류가 발생했다.

`클래스 [org.mybatis.spring.SqlSessionFactoryBean]을(를) 로드할 수 없습니다`

왜 이런 오류가 발생했는지 재대로 확인 해보면 대략 이런 구문을 발견할 수 있을 것이다.

`Unsupported major.minor version 52.0` 을 확인할 수 있다. 

문제는 요건 JDK 1.8 버젼부터 적용이 가능하다는 점이다. 

이번 프로젝트 특성상 JDK 를 1.7버젼으로 강제해야하는 상황인지라, 실제로 mybatis를 돌리려면 다른 방안이 필요했다.



### 해결 방법

mybatis 깃허브에는 다음과 같은 issue가 등록 되어있다. 

https://github.com/mybatis/mybatis-3/issues/1207

링크의 내용은 다음과 같다.  **The MyBatis 3.5.0 will support JDK 8+** 

그러니까 3.5.0 버젼부터는 JDK 8버젼이상부터 지원한다는 뜻이다. 결국에 이 문제를 해결하기 위해서는 3.5.0 이하 버젼을 이용해서 오류를 해결하는 방식을 취해야한다.



```xml
<!-- MyBatis -->
		<dependency>
		    <groupId>org.mybatis</groupId>
		    <artifactId>mybatis</artifactId>
		    <version>3.4.6</version>
		</dependency>
		
		<dependency>
		    <groupId>org.mybatis</groupId>
		    <artifactId>mybatis-spring</artifactId>
		    <version>1.3.3</version>
		</dependency>
```



참고로 추가적으로 mybatis-spring을 통해서 의존성을 받아와야하는 경우에도 버젼을 낮춰서 실행해야한다. 

mybatis-spring도 높은 버젼에서는  JDK 1.8버젼이상에서만 작동되는 것 같다.



다음과 같이 의존성을 변경하거나 라이브러리를 버젼을 낮춰서 설치하면 오류없이 작동된다!