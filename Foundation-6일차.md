

# HtmlUnit

HTML 템플릿 뷰 테스트를 보다 전문적으로 하자.

- http://htmlunit.sourceforge.net/
- http://htmlunit.sourceforge.net/gettingStarted.html
- 의존성 추가

```java
<dependency>
   <groupId>org.seleniumhq.selenium</groupId>
   <artifactId>htmlunit-driver</artifactId>
   <scope>test</scope>
</dependency>
<dependency>
   <groupId>net.sourceforge.htmlunit</groupId>
   <artifactId>htmlunit</artifactId>
   <scope>test</scope>
</dependency>
```

- @Autowire WebClient



---

**스프링 @MVC 예외 처리 방법**

- @ControllerAdvice
- @ExceptionHandler

**스프링 부트가 제공하는 기본 예외 처리기**

- BasicErrorController
  - HTML과 JSON 응답 지원
- 커스터마이징 방법
  - ErrorController 구현

**커스텀 에러 페이지**

- 상태 코드 값에 따라 에러 페이지 보여주기
- src/main/resources/static|template/error/  
  error 디렉토리 만들고 그 안에 404, 5xx html 만들어주면 자동으로 인식해서 사용됨.
- 404.html
- 5xx.html
- ErrorViewResolver 구현

```java
@Controller
public class SampleController {

    @GetMapping("/hello")
    public String hello(){
        throw new SampleException();
    }

    @ExceptionHandler(SampleException.class) //Sample 익셉션 발생하면 이걸 발발
    public @ResponseBody AppError sampleError(SampleException s){
        AppError appError = new AppError();
        appError.setMsg("error.app.key");
        appError.setMsg("IDID");
        return appError;
    }
}
```

https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-handlermapping-interceptor

---

# Spring HATEOAS

**H**ypermedia **A**s **T**he **E**ngine **O**f **A**pplication **S**tate

- 서버: 현재 리소스와 **연관된 링크 정보**를 클라이언트에게 제공한다.
- 클라이언트: **연관된 링크 정보**를 바탕으로 리소스에 접근한다.

- 연관된 링크 정보
  - **Rel**ation
  - **H**ypertext **Ref**erence)
- spring-boot-starter-hateoas 의존성 추가
- https://spring.io/understanding/HATEOAS
- https://spring.io/guides/gs/rest-hateoas/
- https://docs.spring.io/spring-hateoas/docs/current/reference/html/

ObjectMapper 제공

- spring.jackson.*
- Jackson2ObjectMapperBuilder

LinkDiscovers 제공

- 클라이언트 쪽에서 링크 정보를 Rel 이름으로 찾을때 사용할 수 있는 XPath 확장 클래스



```java
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.get;
import static org.springframework.test.web.servlet.result.MockMvcResultHandlers.print;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.jsonPath;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;

@RunWith(SpringRunner.class)
@WebMvcTest(SampleController.class)
public class SampleControllerTest {

    @Autowired
    MockMvc mo;

    @Autowired
    ObjectMapper mapper;

    @Test
    public void hello() throws Exception {
        mo.perform(get("/hello"))
                .andDo(print())
                .andExpect(status().isOk())
                .andExpect(jsonPath("$._links.self").exists());

    }
}

---
  
import org.springframework.hateoas.Resource;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

import static org.springframework.hateoas.mvc.ControllerLinkBuilder.linkTo;
import static org.springframework.hateoas.mvc.ControllerLinkBuilder.methodOn;

@RestController
public class SampleController {
    @GetMapping("/hello")
    public Resource<Hello> Hello(){
        Hello hello = new Hello();
        hello.setName("Hey, ");
        hello.setPrefix("LEN");

        Resource<Hello> helloResource = new Resource<>(hello);
        helloResource.add(linkTo(methodOn(SampleController.class).Hello()).withSelfRel());
        return helloResource;
    }
}
```

---

# CORS

## SOP과 CORS

- Single-Origin Policy
- Cross-Origin Resource Sharing
- Origin이란?
  - URI 스키마 (http, https)
  - hostname (whiteship.me, localhost)
  - 포트 (8080, 18080)

## 스프링 MVC @CrossOrigin

- https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/web.html#mvc-cors
- @Controller나 @RequestMapping에 추가하거나
- WebMvcConfigurer 사용해서 글로벌 설정



8080에 `Hello` GET매서드 만들고, 다른 어플에서 ajax 통신을 시도하니,

![image-20190805132513031](http://ww2.sinaimg.cn/large/006tNc79gy1g5oos4xi1cj30m601kwer.jpg)

와 같이 나왔다.

즉, CrossOrigin을 추가해야 한다.



```java
@SpringBootApplication
@RestController
public class Application {

    @CrossOrigin(origins = "http://localhost:18080")
    @GetMapping("/hello")
    public String hello(){
        return "Hello";
    }

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```



그 외 `@Controller` 에 넣거나, `WebConfig`에 추가할 수 있다.

```java
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.CorsRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;
@Configuration
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/**")
                .allowedOrigins("http://localhost:18080")
    }
}
```

---

# SpringBoot 데이터

## 소개

![image-20190805133159928](http://ww2.sinaimg.cn/large/006tNc79gy1g5ooz6ve1lj30l80a5jsg.jpg)



지원하는 인-메모리 데이터베이스  

- H2 (추천, 콘솔 때문에...)
- HSQL
- Derby

Spring-JDBC가 클래스패스에 있으면 자동 설정이 필요한 빈을 설정 해줍니다.

- DataSource
- JdbcTemplate

인-메모리 데이터베이스 기본 연결 정보 확인하는 방법

- URL: “testdb”
  ![image-20190805142723192](http://ww3.sinaimg.cn/large/006tNc79gy1g5oqktlpfpj305l03fgll.jpg)
- username: “sa”
- password: “”

```java

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.ApplicationArguments;
import org.springframework.boot.ApplicationRunner;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.stereotype.Component;

import javax.sql.DataSource;
import java.sql.Connection;
import java.sql.Statement;

@Component
public class H2Runner implements ApplicationRunner {

    @Autowired
    DataSource dataSource;

    @Autowired
    JdbcTemplate jdbcTemplate;

    @Override
    public void run(ApplicationArguments args) throws Exception {
        try(Connection connection = dataSource.getConnection()) {
            connection.getMetaData().getURL();
            connection.getMetaData().getUserName();

            Statement statement = connection.createStatement();
            String sql = "CREATE TABLE USER(ID INTEGER NOT NULL, name VARCHAR(255), PRIMARY KEY (ID))";
            statement.executeUpdate(sql);
        }
        jdbcTemplate.execute("INSERT INTO USER VALUES (1, 'Len')");
    }
}
```



---

# MySQL

DBCP - 데이터 컨넥션 푸우~~

## 지원하는 DBCP

1. [HikariCP](https://github.com/brettwooldridge/HikariCP) (기본)
   1. https://github.com/brettwooldridge/HikariCP#frequently-used
2. [Tomcat CP](https://tomcat.apache.org/tomcat-7.0-doc/jdbc-pool.html)
3. [Commons DBCP2](https://commons.apache.org/proper/commons-dbcp/)



기본적으로 HikariCP를 채택함.

조금더 자세한 사항은 위 깃헙 링크에서 확인할 수 있다.



#### DBCP 설정

- **spring.datasource.hikari.\***
- spring.datasource.tomcat.*
- spring.datasource.dbcp2.*

#### MySQL 커넥터 의존성 추가

```java
<dependency>
   <groupId>mysql</groupId>
   <artifactId>mysql-connector-java</artifactId>
</dependency>
```

#### MySQL 추가 (도커 사용)

- docker run -p 3306:3306 --name **mysql_boot** -e MYSQL_ROOT_PASSWORD=**1** -e MYSQL_DATABASE=**springboot** -e MYSQL_USER=**keesun** -e MYSQL_PASSWORD=**pass** -d mysql
- docker exec -i -t mysql_boot bash
- mysql -u root -p

#### MySQL용 Datasource 설정

- spring.datasource.url=jdbc:mysql://localhost:3306/springboot?useSSL=false
- spring.datasource.username=keesun
- spring.datasource.password=pass

### MySQL 접속시 에러

MySQL 5.* 최신 버전 사용할 때

***Problem***

Sat Jul 21 11:17:59 PDT 2018 WARN: Establishing SSL connection without server's identity verification is not recommended. **According to MySQL 5.5.45+, 5.6.26+ and 5.7.6+ requirements SSL connection must be established by default if explicit option isn't set.** For compliance with existing applications not using SSL the **verifyServerCertificate property is set to 'false'**. You need either to explicitly disable SSL by setting **useSSL=false**, or set **useSSL=true and provide truststore** for server certificate verification.



=> jdbc:mysql:/localhost:3306/springboot?**useSSL=false**



***Problem***

com.mysql.jdbc.exceptions.jdbc4.MySQLNonTransientConnectionException: Public Key Retrieval is not allowed



=> jdbc:mysql:/localhost:3306/springboot?useSSL=false&**allowPublicKeyRetrieval=true**



MySQL 라이센스 (GPL) 주의

- MySQL 대신 MariaDB 사용 검토
- 소스 코드 공개 의무 여부 확인



---

# postgresql

```
<dependency>
   <groupId>org.postgresql</groupId>
   <artifactId>postgresql</artifactId>
</dependency>

```

## PostgreSQL 설치 및 서버 실행 (docker)

```shell
docker run -p 5432:5432 -e POSTGRES_PASSWORD=pass -e POSTGRES_USER=keesun -e POSTGRES_DB=springboot --name postgres_boot -d postgres

docker exec -i -t postgres_boot bash

su - postgres

psql springboot

데이터베이스 조회
\list

테이블 조회
\dt

쿼리
SELECT * FROM account;
```

## PostgreSQL 경고 메시지

경고 :  `org.postgresql.jdbc.PgConnection.createClob() is not yet implemented`   

해결 :  `spring.jpa.properties.hibernate.jdbc.lob.non_contextual_creation=true`

---

# JPA

```
<dependency>
   <groupId>org.springframework.boot</groupId>
   <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
```

스프링 데이터 JPA 사용하기

- @Entity 클래스 만들기
- Repository 만들기

스프링 데이터 리파지토리 테스트 만들기

- H2 DB를 테스트 의존성에 추가하기
- @DataJpaTest (슬라이스 테스트) 작성

---

# JPA 연동

```java
@RunWith(SpringRunner.class)
@DataJpaTest //슬라이싱 테스트!!!!
public class RepoTest {

    @Autowired
    DataSource dataSource;

    @Autowired
    JdbcTemplate jdbcTemplate;

    @Autowired
    AccountRepository accountRepository;

    @Test
    public void di() throws SQLException {
        try(Connection connection = dataSource.getConnection()){
            DatabaseMetaData metaData = connection.getMetaData();
        }
    }
}
```

이렇게 하면, 임베디드 DB 말고, 연결된 DB 활용할 수 있다.

---

# 데이터베이스 초기화

JPA를 사용한 데이터베이스 초기화

- spring.jpa.hibernate.ddl-auto
- spring.jpa.generate-dll=true로 설정 해줘야 동작함.

SQL 스크립트를 사용한 데이터베이스 초기화

- schema.sql 또는 schema-${platform}.sql
- data.sql 또는 data-${platform}.sql
- ${platform} 값은 spring.datasource.platform 으로 설정 가능.



> spring.jpa.hibernate.ddl-auto 선택 사항은

`create` `update` `validate`

schema.sql 실행되고 난 뒤에 data.sql 이 실행된다.

 

- 스프링 부트에서는 클래스패스 경로에(src/main/resources등) schema.sql, data.sql, schema-${platform}.sql, data-${platform}.sql 파일등이 존재한다면 자동으로 실행해서 스키마 구조와 데이터를 초기화 시켜주는데. ${platform} 값은,만약 application.properties 파일에서 spring.datasource.platform=mysql 이라고 했다면 schema-${platform}.sql 파일의 이름은 schema-mysql.sql이 될 것이다.



 

- JPA는 시작시점에 DDL을 자동 생성 및 실행해 주는 기능이 있는데 두개의 외부 속성으로 정의한다. spring.jpa.generate-ddl (boolean) on, off 값을 가지고 DB벤더에 종속적이지 않으며 spring.jpa.hibernate.ddl-auto (enum)는 하이버네이트 특성으로 하이버네이트 Sessionfactory가 시작할 때 JPA의 엔티티 매핑, 연관관계 설정을 기본으로 테이블과 같은 스키마 생성 스크립트를 만들고 실행하여 데이터베이스 초기화를 지원하는데 열거형 값인 none, validate, update, create, create-drop 값을 가진다.



 

*hsqldb, h2 and derby 데이터베이스는 create-drop이 기본이며 그외 DB는 none이 기본값이다.* 

**none :** 자동 DDL 생성 안함.

**create :** 하이버네이트 Sessionfactory가 시작될 때 항상 다시 생성, 이미 있다면 지우고 생성.

**create-drop :** Sessionfactory가 시작될 때 생성 후 종료할 때 삭제한다.

**update :** Sessionfactory가 시작될 때 엔티티 클래스(도메인 클래스)와 DB에 생성된 스키마 구조를 비교해서 DB쪽에 생성이 안된 테이블 또는 칼럼이 있다면 DB 스키마를 변경해서 생성시키지만 기 생성된 스키마 구조를 삭제하지는 않는다.

**validate :** Sessionfactory가 시작될 때 엔티티 클래스(도메인 클래스)와 DB에 생성된 스키마 구조를 비교해 같은지 확인만 할 뿐 DB 스키마 구조는 변경하지 않고 만약 다르다면 예외를 발생시킨다.



---

# 데이터베이스 마이그레이션

Flyway와 Liquibase가 대표적인데, 지금은 Flyway를 사용하겠습니다. 

 https://docs.spring.io/spring-boot/docs/2.0.3.RELEASE/reference/htmlsingle/#howto-execute-flyway-database-migrations-on-startup

- org.flywaydb:flyway-core

### 마이그레이션 디렉토리

- db/migration 또는 db/migration/{vendor}
- spring.flyway.locations로 변경 가능

### 마이그레이션 파일 이름

- V숫자__이름.sql
- V는 꼭 대문자로.
- 숫자는 순차적으로 (타임스탬프 권장)
- 숫자와 이름 사이에 언더바 **두 개**.
- 이름은 가능한 서술적으로.

 

---

# Redis 활용하기.

캐시, 메시지 브로커, 키/밸류 스토어 등으로 사용 가능.

- **의존성 추가**
  - spring-boot-starter-data-redis
- **Redis 설치 및 실행 (도커)**
  - docker run -p 6379:6379 --name redis_boot -d redis
  - docker exec -i -t redis_boot redis-cli
- **스프링 데이터 Redis**
  - https://projects.spring.io/spring-data-redis/
  - StringRedisTemplate 또는 RedisTemplate
  - extends CrudRepository
- **Redis 주요 커맨드**
  - https://redis.io/commands
  - keys *
  - get {key}
  - hgetall {key}
  - hget {key} {column}
- **커스터마이징**
  - spring.redis.*



![image-20190807152828103](http://ww3.sinaimg.cn/large/006tNc79gy1g5r3l2r525j306u04t3zc.jpg)



---

# Neto4j

이 친구의 친구의 정보를 가져온다거나-

정보의 정보를 가져온다거나 등의 행위를 할 때 좋다.

그러나 하위 버전과의 호환이 좋지 못하다.



[Neo4j](https://neo4j.com/)는 노드간의 연관 관계를 영속화하는데 유리한 그래프 데이터베이스 입니다.

- 의존성 추가
  - spring-boot-starter-data-neo4j
- **Neo4j 설치 및 실행 (도커)**docker run -p 7474:7474 -p 7687:7687 -d --name noe4j_boot neo4j
- http://localhost:7474/browser
- 스프링 데이터 Neo4J
  - Neo4jTemplate (Deprecated)
  - **SessionFactory**
  - Neo4jRepository



---

https://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/#boot-features-sql

