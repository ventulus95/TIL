# Spring Data JPA에서 Query를 사용하는 방식.

## 쿼리를 자동 생성해준다고?

![쩌..쩐다](https://i.pinimg.com/564x/87/f9/8b/87f98babe8052baab77792f22f9a75ae.jpg)

Spring boot를 통해서 개발을 하게된다면, DB에 데이터를 삽입,읽기등 여러가지 작동을 하기위해서는 방식이 필요하다. 쿼리를 작동시키는 방식에는 여러가지 방식이 존재한다.

실제로 현재 Spring Data JPA를 사용하면 꽤 편리한점이 있기 때문에,  그 점을 잘 이용하기위해서는 결국에는 Query를 **"자동 생성할 수 있는 포인트를 아는 것이 좋다."** 

## 하지만 하나 짚고 넘어가야 할 점

김영한님의 JPA강좌에서는 다음과 같이 주의점을 준다. 

> 스프링 데이터 JPA는 JPA를 편리하게 사용하도록 도와주는 기술입니다. 따라서 JPA를 먼저 학습한 후에 스프링 데이터 JPA를 학습해야 합니다.

JPA를 좀 더 구체적으로 배운뒤에 Spring Data JPA를 사용하는 걸 추천해봅니다.

## Spring JPA Data Doc에서는?

https://docs.spring.io/spring-data/jpa/docs/2.3.3.RELEASE/reference/html/#jpa.repositories

다음과 같은 방식으로 키워드만 있으면, JPA를 통해서 SQL이 자동 생성됩니다. 키워드는 다음과 같다.

| Keyword                | Sample                                                       | JPQL snippet                                                 |
| :--------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| `And`                  | `findByLastnameAndFirstname`                                 | `… where x.lastname = ?1 and x.firstname = ?2`               |
| `Or`                   | `findByLastnameOrFirstname`                                  | `… where x.lastname = ?1 or x.firstname = ?2`                |
| `Is`, `Equals`         | `findByFirstname`,`findByFirstnameIs`,`findByFirstnameEquals` | `… where x.firstname = ?1`                                   |
| `Between`              | `findByStartDateBetween`                                     | `… where x.startDate between ?1 and ?2`                      |
| `LessThan`             | `findByAgeLessThan`                                          | `… where x.age < ?1`                                         |
| `LessThanEqual`        | `findByAgeLessThanEqual`                                     | `… where x.age <= ?1`                                        |
| `GreaterThan`          | `findByAgeGreaterThan`                                       | `… where x.age > ?1`                                         |
| `GreaterThanEqual`     | `findByAgeGreaterThanEqual`                                  | `… where x.age >= ?1`                                        |
| `After`                | `findByStartDateAfter`                                       | `… where x.startDate > ?1`                                   |
| `Before`               | `findByStartDateBefore`                                      | `… where x.startDate < ?1`                                   |
| `IsNull`, `Null`       | `findByAge(Is)Null`                                          | `… where x.age is null`                                      |
| `IsNotNull`, `NotNull` | `findByAge(Is)NotNull`                                       | `… where x.age not null`                                     |
| `Like`                 | `findByFirstnameLike`                                        | `… where x.firstname like ?1`                                |
| `NotLike`              | `findByFirstnameNotLike`                                     | `… where x.firstname not like ?1`                            |
| `StartingWith`         | `findByFirstnameStartingWith`                                | `… where x.firstname like ?1` (parameter bound with appended `%`) |
| `EndingWith`           | `findByFirstnameEndingWith`                                  | `… where x.firstname like ?1` (parameter bound with prepended `%`) |
| `Containing`           | `findByFirstnameContaining`                                  | `… where x.firstname like ?1` (parameter bound wrapped in `%`) |
| `OrderBy`              | `findByAgeOrderByLastnameDesc`                               | `… where x.age = ?1 order by x.lastname desc`                |
| `Not`                  | `findByLastnameNot`                                          | `… where x.lastname <> ?1`                                   |
| `In`                   | `findByAgeIn(Collection<Age> ages)`                          | `… where x.age in ?1`                                        |
| `NotIn`                | `findByAgeNotIn(Collection<Age> ages)`                       | `… where x.age not in ?1`                                    |
| `True`                 | `findByActiveTrue()`                                         | `… where x.active = true`                                    |
| `False`                | `findByActiveFalse()`                                        | `… where x.active = false`                                   |
| `IgnoreCase`           | `findByFirstnameIgnoreCase`                                  | `… where UPPER(x.firstame) = UPPER(?1)`                      |

이런 예시를 통해서 아래쪽과 같이 메소드를 만들어쓰는 방식을 통해서 서비스단에서 그 메소드를 이용하면, 스스로 쿼리를 생성하여  작동하게 된다.

![KakaoTalk_Photo_2020-09-06-10-19-41](https://user-images.githubusercontent.com/17822723/92316189-91e22b80-f02b-11ea-9c9a-73e94f4c17c6.png)

@Repository에서 메소드를 만들고 나면, 

![KakaoTalk_Photo_2020-09-06-10-19-51](https://user-images.githubusercontent.com/17822723/92316193-94448580-f02b-11ea-8948-56485e6fe8ec.png)

@Service에서 다음과 같이 사용할수 있다.



## Query를 직접 써서 직접 메소드를 만드는 방법

물론 Spring Data JPA를 메소드 키워드를 통해서, 쿼리를 직접 만드는 방법도 있지만, 구체적인 쿼리를 작성해야하는 경우도 있다. 사실 저런 키워드를 가지고 충분히 만드는 것도 가능하지만, 직접 쿼리를 사용하는 것도 직관적인 경우도 있다.

1. 쿼리를 직접 쓰는 경우는 다음과 같은 조건이 있어야한다.

```java
@Query("SELECT u FROM User u WHERE u.status = 1") 
Collection<User> findAllActiveUsers();

```

From table 별칭을 Select문안에 넣어야한다. 안그러면 오류를 만들어낸다. 하지만 이런 오류를 찝어내면서 해결하는 것이 은근 귀찮다.

2. DB에서 쓰는 것처럼 직접 쿼리를 작성하는 방식

```java
@Query(
  value = "SELECT * FROM USERS u WHERE u.status = 1", 
  nativeQuery = true)
Collection<User> findAllActiveUsersNative();
```

@Query 옵션에서 nativeQuery = true를 줘야지만... DB에서 쿼리문을 작성하는 방식으로 작성할 수도 있다.



## JPA에서 메소드 Parameter를 통해서 @Query에 그 parameter가 넣어지는 방식

쿼리를 작성시 파라미터를 통한 구체적 조건을 줘야하는 경우가 종종 있다.  그러면 어떤식으로 처리해야하는가에 대한 문제가 발생할때 다음과 같은 방법을 사용하자.

1. ?를 통한 경우 -> parameter의 위치에 따른 숫자를 넣어주면 된다.

   ?1 -> parameter 첫번째 자리에 있는걸 넣겠다는 뜻.

```java
public interface UserRepository extends JpaRepository<User, Long> {

  @Query("select u from User u where u.emailAddress = ?1")
  User findByEmailAddress(String emailAddress);
}
```

2. :name -> 파라미터의 이름으로 검색하는 경우

   ```java
   public interface UserRepository extends JpaRepository<User, Long> {
   
     @Query("select u from User u where u.firstname = :firstname or u.lastname = :lastname")
     User findByLastnameOrFirstname(@Param("lastname") String lastname,
                                    @Param("firstname") String firstname);
     
     @Query("select u from User u where u.firstname = :firstname or u.lastname = :lastname")
     User findByLastnameOrFirstname(String lastname, String firstname); //이렇게도 사용 가능하다.
   }
   ```

   **물론 @Param으로 지정해줘도 되지만, Spring 4버젼,  java8 기준으로 @Param주석 없이도 파라미터의 이름 기준으로 알아서 잘 찾아준다.**  

물론 이경우 말고도 여러가지 케이스가 존재한다. SPEL을 이용할 수도 있기도하고 Sorting하는 방식이라던가 진짜 여러가지 방식으로 활용할 수 있다. 자세한건 

https://docs.spring.io/spring-data/jpa/docs/2.3.3.RELEASE/reference/html/#jpa.repositories

를 참고해서 넘어가는 것이 좋을 듯합니다.