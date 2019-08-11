# AutoConfiguration

- @EnableAutoConfiguration (@SpringBootApplication 안에 숨어 있음)
- 빈은 사실 두 단계로 나눠서 읽힘
  - 1단계: @ComponentScan
  - 2단계: @EnableAutoConfiguration
- @ComponentScan
  - @Component
  - @Configuration @Repository @Service @Controller @RestController
- @EnableAutoConfiguration
  - spring.factories
    - org.springframework.boot.autoconfigure.EnableAutoConfiguration
      여기서 각 설정파일을 찾아서 설정해준다. 
  - @Configuration
  - @ConditionalOnXxxYyyZzz



만약 `@EnableAutoConfiguration` 쓰지 않겠다해도 쓸 수 있음.

기본적으로 웹 어플리션으로 만들려고 한다면?

```java 
@Configuration
@ComponentScan
public class SpringGettingStartApplication {

    public static void main(String[] args) {
        SpringApplication application = new SpringApplication(Application.class);
        application.setWebApplicationType(WebApplicationType.NONE);
        application.run(args);
    }
}
```

*질문*

`TypeExcludeFilter` 의 역할은 무엇 일까?

---

https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#boot-features-developing-auto-configuration

#### 구현 방법

1. 의존성 추가
2. @Configuration 파일 작성 
3. spring.factories 안에 자동 설정 파일 추가

```
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
FQCN,\
FQCN
```

4. mvn install



그럼 이제, 다른 프로젝트에서 설정한 @Configuration 패키지를 가져오는 작업을 시작해보자.

`likelen-springboot-starter` 라는 프로젝트를 생성한다.



![image-20190730114116379](http://ww2.sinaimg.cn/large/006tNc79gy1g5ho26i27pj308o06974f.jpg)

pox.xml은

```java
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.example</groupId>
    <artifactId>likelenspringbootstarter</artifactId>
    <version>0.0.1-SNAPSHOT</version>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-autoconfigure</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-autoconfigure-processor</artifactId>
            <optional>true</optional>
        </dependency>

    </dependencies>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-dependencies</artifactId>
                <version>2.0.3.RELEASE</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
</project>
```

`HolomanConfiguration.java`

```java
package com.example.likelenspringbootstarter;

import org.springframework.context.annotation.Bean;

public class HolomanConfiguration {

    @Bean
    public Holoman holoman(){
        Holoman holoman = new Holoman();
        holoman.setHowLong(5);
        holoman.setName("likeLen");
        return holoman;
    }
}
```

`Holoman.java`

```java
package com.example.likelenspringbootstarter;

public class Holoman {

    String name;
    int howLong;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getHowLong() {
        return howLong;
    }

    public void setHowLong(int howLong) {
        this.howLong = howLong;
    }

    @Override
    public String toString() {
        return "Holoman{" +
                "name='" + name + '\'' +
                ", howLong=" + howLong +
                '}';
    }
}
```



그리고 마지막으로 `spring.factores` 에다가

```java
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
com.example.likelenspringbootstarter.HolomanConfiguration
```

와 같이 값을 준다.



다시 본 프로젝트로 돌아와서 `pom.xml`에서 

```java
<dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>com.example</groupId>
            <artifactId>likelenspringbootstarter</artifactId>
            <version>0.0.1-SNAPSHOT</version>
        </dependency>
    </dependencies>
```

와 같이 설정하고 import를 확인하면

![image-20190730114437282](http://ww3.sinaimg.cn/large/006tNc79gy1g5ho5mmadaj30bu08r3z0.jpg)





다음과 같이 외부라이브러리가 들어온 것을 알 수 있다.



여기서 한가지 알아야 할점!!

```java
package com.example.springgettingstart;

import com.example.likelenspringbootstarter.Holoman;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.WebApplicationType;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.annotation.Bean;

@SpringBootApplication
public class SpringGettingStartApplication {

    public static void main(String[] args) {
        SpringApplication application = new SpringApplication(SpringGettingStartApplication.class);
        application.setWebApplicationType(WebApplicationType.NONE);
        application.run(args);
    }
// ApplicationRunner에서 외부라이브러리가 잘 동작함을 확인했다.  
// 그러나 아래 빈은 무시가 된다. 왜? 빈은 2가지의 face를 걸치게 되는데.
// 1. ComponentScan 으로 Holeman을 스캔.
// 2. AutoConfiguration으로 위에 덮어 쓰기. 그러므로 아래 빈은 무시.
    @Bean
    public Holoman holoman(){
        Holoman holoman = new Holoman();
        holoman.setName("_________Like___LEN_______");
        holoman.setHowLong(50);
        return holoman;
    }
}
```

---

# 자동 설정 만들기 2부: @ConfigurationProperties

- 덮어쓰기 방지하기
  - @ConditionalOnMissingBean

- 빈 재정의 수고 덜기
  - @ConfigurationProperties(“holoman”)
  - @EnableConfigurationProperties(HolomanProperties)
  - 프로퍼티 키값 자동 완성



```java
		@Bean
    @ConditionalOnMissingBean
    public Holoman holoman(){
        Holoman holoman = new Holoman();
        holoman.setName("_________Like___LEN_______");
        holoman.setHowLong(50);
        return holoman;
    }
```

요렇게 되면 값이 오버라이딩 기능이 된다.

---

# 내장 웹 서버 이해

- 스프링 부트는 서버가 아니다.
  - 톰캣 객체 생성
  - 포트 설정
  - 톰캣에 컨텍스트 추가
  - 서블릿 만들기
  - 톰캣에 서블릿 추가
  - 컨텍스트에 서블릿 맵핑
  - 톰캣 실행 및 대기

```java
package com.example.springgettingstart;

import org.apache.catalina.Context;
import org.apache.catalina.LifecycleException;
import org.apache.catalina.startup.Tomcat;

import javax.servlet.ServletException;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.io.PrintWriter;

public class Application {

    public static void main(String[] args) throws LifecycleException {
        Tomcat tomcat = new Tomcat();
        tomcat.setPort(8080);

        Context context = tomcat.addContext("/", "/");
        HttpServlet servlet = new HttpServlet() {
            @Override
            protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
                PrintWriter writer = resp.getWriter();
                writer.println("<html><head><title>");
                writer.println("Hey, Tomcat");
                writer.println("</title></head>");
                writer.println("<BODY><h1>Hello Tomcat</h1>");
                writer.println("</html>");
            }
        };

        String servletName = "helloServelt";
        tomcat.addServlet("/", servletName, servlet);
        context.addServletMappingDecoded("/hello", servletName);

        tomcat.start();
        tomcat.getServer().await();
    }
}

```

이 모든 과정을 보다 상세히 또 유연하고 설정하고 실행해주는게 바로 스프링 부트의 자동 설정



- `ServletWebServerFactoryAutoConfiguration` (서블릿 웹 서버 생성)

  ```java
  
  /**
   * {@link EnableAutoConfiguration Auto-configuration} for servlet web servers.
   *
   * @author Phillip Webb
   * @author Dave Syer
   * @author Ivan Sopov
   * @author Brian Clozel
   * @author Stephane Nicoll
   */
  @Configuration
  @AutoConfigureOrder(Ordered.HIGHEST_PRECEDENCE)
  @ConditionalOnClass(ServletRequest.class)
  @ConditionalOnWebApplication(type = Type.SERVLET)
  @EnableConfigurationProperties(ServerProperties.class)
  @Import({ ServletWebServerFactoryAutoConfiguration.BeanPostProcessorsRegistrar.class,
  		ServletWebServerFactoryConfiguration.EmbeddedTomcat.class,
  		ServletWebServerFactoryConfiguration.EmbeddedJetty.class,
  		ServletWebServerFactoryConfiguration.EmbeddedUndertow.class })
  public class ServletWebServerFactoryAutoConfiguration {
  ```

  - `TomcatServletWebServerFactoryCustomizer` (서버 커스터마이징)

- `DispatcherServletAutoConfiguration`

  ```java
  
  /**
   * {@link EnableAutoConfiguration Auto-configuration} for the Spring
   * {@link DispatcherServlet}. Should work for a standalone application where an embedded
   * web server is already present and also for a deployable application using
   * {@link SpringBootServletInitializer}.
   *
   * @author Phillip Webb
   * @author Dave Syer
   * @author Stephane Nicoll
   * @author Brian Clozel
   */
  @AutoConfigureOrder(Ordered.HIGHEST_PRECEDENCE)
  @Configuration
  @ConditionalOnWebApplication(type = Type.SERVLET)
  @ConditionalOnClass(DispatcherServlet.class)
  @AutoConfigureAfter(ServletWebServerFactoryAutoConfiguration.class)
  public class DispatcherServletAutoConfiguration {
  ```

  - 서블릿 만들고 등록

  ---

  # 내장 웹 서버 응용 1부 : 컨테이너와 포트

  https://docs.spring.io/spring-boot/docs/current/reference/html/howto-embedded-web-servers.html

  - 다른 서블릿 컨테이너로 변경
    - undertow / jetty

  메이븐에서 아래와 같이 설정해야 한다.

  - Tomcat을 예외시킨다.
  - undertow 또는 jetty를 다른 서블릿 컨테이너로 변경시킨다.

  ```xml
  <dependencies>
          <dependency>
              <groupId>org.springframework.boot</groupId>
              <artifactId>spring-boot-starter-test</artifactId>
              <scope>test</scope>
          </dependency>
          <dependency>
              <groupId>org.springframework.boot</groupId>
              <artifactId>spring-boot-configuration-processor</artifactId>
              <optional>true</optional>
          </dependency>
          <dependency>
              <groupId>org.springframework.boot</groupId>
              <artifactId>spring-boot-starter-web</artifactId>
              <exclusions>
                  <exclusion>
                      <groupId>org.springframework.boot</groupId>
                      <artifactId>spring-boot-starter-tomcat</artifactId>
                  </exclusion>
              </exclusions>
          </dependency>
  
          <dependency>
              <groupId>org.springframework.boot</groupId>
              <artifactId>spring-boot-starter-undertow</artifactId>
          </dependency>
      </dependencies>
  ```

  - 웹 서버 사용 하지 않기  
    

  - 포트를 설정하거나 알 수 있는 방법은?  
    그외 `application.properties` 입력

    - server.port  
      `server.port=0`

    - 랜덤 포트  
      `server.port=0`

    - ApplicationListner\<ServletWebServerInitializedEvent>  

      ```java
      
      import org.springframework.boot.web.servlet.context.ServletWebServerApplicationContext;
      import org.springframework.boot.web.servlet.context.ServletWebServerInitializedEvent;
      import org.springframework.context.ApplicationListener;
      import org.springframework.stereotype.Component;
      
      @Component
      public class PortListener implements ApplicationListener<ServletWebServerInitializedEvent> {
      
          @Override
          public void onApplicationEvent(ServletWebServerInitializedEvent event) {
              ServletWebServerApplicationContext applicationContext = event.getApplicationContext();
              System.out.println(applicationContext.getWebServer().getPort());
          }
      }
      
      // 밑에 로그에서 어디 포트인지 알 수 있다.
      ```

      

---

# 내장 웹 서버 응용 2부 : HTTPS와 HTTP2

https://opentutorials.org/course/228/4894

https://gist.github.com/keesun/f93f0b83d7232137283450e08a53c4fd

- HTTPS 설정하기
  - 키스토어 만들기
  - HTTP는 못쓰네?

```properties
generate-keystore.sh

keytool -genkey 
  -alias tomcat 
  -storetype PKCS12 
  -keyalg RSA 
  -keysize 2048 
  -keystore keystore.p12 
  -validity 4000
  
server.ssl.key-store=keystore.p12
server.ssl.key-store-type= PKCS12
server.ssl.key-store-password=1qaz2wsx#
server.ssl.key-alias=spring
```



- HTTP 커넥터는 코딩으로 설정하기

  - https://github.com/spring-projects/spring-boot/tree/v2.0.3.RELEASE/spring-boot-samples/spring-boot-sample-tomcat-multi-connectors

- HTTP2 설정
  `curl -I -K --http2 https://localhost:8080/hello`  

  하면 200인데, HTTP/1.1 로 받아진다.

  ```java
  @Bean
  public ServletWebServerFactory serverFactory(){
    TomcatServletWebServerFactory tomcat = new TomcatServletWebServerFactory();
    tomcat.addAdditionalTomcatConnectors(createStandardConnect());
    return tomcat;
  }
  
  private Connector createStandardConnect() {
    Connector connector = new Connector(
      "org.apache.coyote.http11.Http11NioProtocal"
    );
    connector.setPort(8080);
    return connector;
  }
  ```

  - server.http2.enable  
    http2를 사용하기 위해서는 SSL이 먼저 적용되어야 한다.  

    https://docs.spring.io/spring-boot/docs/current/reference/html/howto-embedded-web-servers.html#howto-configure-http2-netty

  - **사용하는 서블릿 컨테이너 마다 다름.**

    Tomcat 8.5에서는 http, https 를 같이 사용하기 위한 설정이 번거롭기 때문에 권장하지 않음.

---

- JDK9와 Tomcat 9+ 추천
- 그 이하는 아래 링크 참고 https://docs.spring.io/spring-boot/docs/current/reference/html/howto-embedded-web-servers.html#howto-configure-http2-tomcat



---

# 독립적으로 실행 가능한 JAR

: maven plugin 관련내용이다.

https://docs.spring.io/spring-boot/docs/current/reference/html/executable-jar.html “그러고 보니 JAR 파일 하나로 실행할 수 있네?”

- mvn package를 하면 실행 가능한 **JAR 파일 “하나가"** 생성 됨.
- spring-maven-plugin이 해주는 일 (패키징)
- 과거 “uber” jar 를 사용
  - 모든 클래스 (의존성 및 애플리케이션)를 하나로 압축하는 방법
  - 뭐가 어디에서 온건지 알 수가 없음
    - 무슨 라이브러리를 쓰는건지..
  - 내용은 다르지만 이름이 같은 파일은 또 어떻게?
- 스프링 부트의 전략
  - 내장 JAR : 기본적으로 자바에는 내장 JAR를 로딩하는 **표준적인 방법이 없음**.
  - 애플리케이션 클래스와 라이브러리 위치 구분
  - org.springframework.boot.loader.jar.JarFile을 사용해서 내장 JAR를 읽는다.
  - org.springframework.boot.loader.Launcher를 사용해서 실행한다.

---

# 스프링 부트 원리 정리

- **의존성 관리**
  - 이것만 넣어도 이만큼이나 다 알아서 가져오네?
    
- **자동 설정**
  - @EnableAutoConfiguration이 뭘 해주는지 알겠어.
    
- **내장 웹 서버**
  - 아 스프링 부트가 서버가 아니라 내장 서버를 실행하는 거군.
    
- **독립적으로 실행 가능한 JAR**
  - spring-boot-maven 플러그인이 이런걸 해주는구나..
    

https://sjh836.tistory.com/131