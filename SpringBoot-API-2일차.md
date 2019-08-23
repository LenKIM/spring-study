# 01. Account 도메인 추가

OAuth2로 인증을 하려면 일단 Account 부터

- id

- email
- password
- roels



AccountRoles

- ADMIN, USER



JPA 맵핑

- @Table(“Users”)
  

JPA enumeration collection mapping

```java
@ElementCollection(fetch = FetchType.EAGER)
@Enumerated(EnumType.STRING) 
private Set<AccountRole> roles;

```

Event에 owner 추가

```java
@ManyToOne
Account manager;
```



# 02. 스프링 시큐리티

## 스프링 시큐리티

- 웹 시큐리티 (Filter 기반 시큐리티)
- 메소드 시큐리티

![image-20190822204948492](http://ww1.sinaimg.cn/large/006y8mN6gy1g68p640prcj30vy0igdlz.jpg)

- 이 둘 다 Security Interceptor를 사용합니다.

  - 리소스에 접근을 허용할 것이냐 말것이냐를 결정하는 로직이 들어있음.

- 의존성 추가  

  ```java
  <dependency>
  <groupId>org.springframework.security.oauth.boot</groupId> <artifactId>spring-security-oauth2-autoconfigure</artifactId> <version>2.1.0.RELEASE</version> </dependency>
  ```

- 테스트 다 깨짐 (401 Unauthorized)

  - 깨지는 이유는 스프링 부트가 제공하는 스프링 시큐리티 기본 설정 때문.

UserDetailsService 구현

- 예외 테스트하기
  - expected
  - @Rule ExpectedException
  - try-catch
- assertThat(collection).extracting(GrantedAuthroity::get)



# 02. 예외 테스트

1. `@Test(expected)`  
예외 타입만 확인 가능
2. `try-catch`  
  예외 타입과 메시지 확인 가능.하지만 코드가 다소 복잡.
3. `@Rule ExpectedException`  
  코드는 간결하면서 예외 타입과 메시지 모두 확인 가능



# 03. 스프링 시큐리티 기본 설정

시큐리티 필터를 적용하기 않음...

- /docs/index.html

로그인 없이 접근 가능

- GET /api/eventsGET /api/events/{id}

로그인 해야 접근 가능나머지 다...

- POST /api/events
- PUT /api/events/{id}
- ...

