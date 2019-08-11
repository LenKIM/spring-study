# RestTemplate과 WebClient

#### RestTemplate

- Blocking I/O 기반의 Synchronous API
- RestTemplateAutoConfiguration
- 프로젝트에 spring-web 모듈이 있다면 RestTemplate**Builder**를 빈으로 등록해 줍니다.
- https://docs.spring.io/spring/docs/current/spring-framework-reference/integration.html#rest-client-access



```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.ApplicationArguments;
import org.springframework.boot.ApplicationRunner;
import org.springframework.boot.web.client.RestTemplateBuilder;
import org.springframework.stereotype.Component;
import org.springframework.util.StopWatch;
import org.springframework.web.client.RestTemplate;

@Component
public class RestRunner implements ApplicationRunner {

    @Autowired
    RestTemplateBuilder restTemplateBuilder;

    @Override
    public void run(ApplicationArguments args) throws Exception {
        RestTemplate restTemplate = restTemplateBuilder
                .build();

        StopWatch stopWatch = new StopWatch();
        stopWatch.start();

        String forObject = restTemplate.getForObject("http://localhost:8080/hello", String.class);
        System.out.println(forObject);

        String forObject2 = restTemplate.getForObject("http://localhost:8080/world", String.class);
        System.out.println(forObject2);

        stopWatch.stop();
        System.out.println(stopWatch.prettyPrint());

    }
}
```

*싱크로나이즈한 코드.*



```tex
hello
world
StopWatch '': running time (millis) = 8262
-----------------------------------------
ms     %     Task name
-----------------------------------------
08262  100%  
```

8262



#### WebClient

- Non-Blocking I/O 기반의 Asynchronous API
- WebClientAutoConfiguration
- 프로젝트에 spring-webflux 모듈이 있다면 WebClient.**Builder**를 빈으로 등록해 줍니다.
- https://docs.spring.io/spring/docs/current/spring-framework-reference/web-reactive.html#webflux-client



```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.ApplicationArguments;
import org.springframework.boot.ApplicationRunner;
import org.springframework.stereotype.Component;
import org.springframework.util.StopWatch;
import org.springframework.web.reactive.function.client.WebClient;
import reactor.core.publisher.Mono;

@Component
public class RestRunner implements ApplicationRunner {

    @Autowired
    WebClient.Builder builder;

    @Override
    public void run(ApplicationArguments args) {
        WebClient webClient = builder.build();

        StopWatch stopWatch = new StopWatch();
        stopWatch.start();

        // Stream API
        Mono<String> helloMono = webClient.get().uri("http://localhost:8080/hello")
                .retrieve()
                .bodyToMono(String.class);
        helloMono.subscribe( s -> {
            System.out.println(s);

            if(stopWatch.isRunning()){
                stopWatch.stop();
            }
            System.out.println(stopWatch.prettyPrint());
            stopWatch.start();
        });
        Mono<String> worldMono = webClient.get().uri("http://localhost:8080/world")
                .retrieve()
                .bodyToMono(String.class);

        worldMono.subscribe( s -> {
            System.out.println(s);

            if(stopWatch.isRunning()){
                stopWatch.stop();
            }
            System.out.println(stopWatch.prettyPrint());
            stopWatch.start();
        });
    }
}
```

```
world
StopWatch '': running time (millis) = 3566
-----------------------------------------
ms     %     Task name
-----------------------------------------
03566  100%  

hello
StopWatch '': running time (millis) = 5512
-----------------------------------------
ms     %     Task name
-----------------------------------------
03566  065%  
01946  035%  
```

---

# RestTemplate과 WebClient 커스터마이징

#### RestTemplate

- 기본으로 java.net.HttpURLConnection 사용.
- 커스터마이징
  - 로컬 커스터마이징
  - 글로벌 커스터마이징
    - RestTemplateCustomizer
    - 빈 재정의



#### WebClient

- 기본으로 Reactor Netty의 HTTP 클라이언트 사용.
- 커스터마이징
  - 로컬 커스터마이징
  - 글로벌 커스터마이징
    - WebClientCustomizer
    - 빈 재정의



```java

@SpringBootApplication
public class SpringrestwebclientApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringrestwebclientApplication.class, args);
    }

    @Bean
    public WebClientCustomizer webClientCustomizer(){
        return webClientBuilder -> webClientBuilder.baseUrl("http://localhost:8080");
    }
    
    @Bean
    public RestTemplateCustomizer restTemplateCustomizer(){
        return new RestTemplateCustomizer() {
            @Override
            public void customize(RestTemplate restTemplate) {
                restTemplate.setRequestFactory(new HttpComponentsClientHttpRequestFactory());
            }
        };
    }
}
```

---

# 스프링 부트 운영 :  Actuator

https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#production-ready-endpoints   
**의존성 추가**

위 Docs에서 여러 가지 프로퍼티를 설정할 수 있다.

- spring-boot-starter-actuator

**애플리케이션의 각종 정보를 확인할 수 있는 Endpoints**

- 다양한 Endpoints 제공.
- JMX 또는 HTTP를 통해 접근 가능 함.
- shutdown을 제외한 모든 Endpoint는 기본적으로 **활성화** 상태.
- 활성화 옵션 조정
  - management.endpoints.enabled-by-default=false
  - management.endpoint.info.enabled=true



![image-20190807193848749](http://ww2.sinaimg.cn/large/006tNc79gy1g5rath9yn3j31a20diae0.jpg)



**JConsole 사용하기**

- https://docs.oracle.com/javase/tutorial/jmx/mbeans/
- https://docs.oracle.com/javase/7/docs/technotes/guides/management/jconsole.html

**VisualVM 사용하기**

- https://visualvm.github.io/download.html



위 2개로 여러가지 상태 모니터링을 가능하게 해준다.



**HTTP 사용하기**

- /actuator
- health와 info를 제외한 대부분의 Endpoint가 기본적으로 **비공개** 상태
- 공개 옵션 조정
  - management.endpoints.web.exposure.include=*
  - management.endpoints.web.exposure.exclude=env,beans



---

# 스프링 부트 어드민



https://github.com/codecentric/spring-boot-admin

스프링 부트 Actuator UI 제공 어드민 서버 설정

```java
<dependency>
  <groupId>de.codecentric</groupId>
  <artifactId>spring-boot-admin-starter-server</artifactId>
  <version>2.0.1</version>
</dependency>

@EnableAdminServer
```

클라이언트 설정

```java
<dependency>
  <groupId>de.codecentric</groupId>
  <artifactId>spring-boot-admin-starter-client</artifactId>
  <version>2.0.1</version>
</dependency>

spring.boot.admin.client.url=http://localhost:8080
management.endpoints.web.exposure.include=*
```

