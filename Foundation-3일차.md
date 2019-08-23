# 배울 내용

**스프링 부트 핵심 기능**

- SpringApplication
- 외부 설정
- 프로파일
- 로깅
- 테스트
- Spring-Dev-Tools



각종 기술 연동

- 스프링 웹 MVC
- 스프링 데이터
- 스프링 시큐리티
- REST API 클라이언트
- 다루지 않는 내용들



---

# SpringApplication #1

https://docs.spring.io/spring-boot/docs/current/reference/html/boot-features-spring-application.html#boot-features-spring-application



- 기본 로그 레벨 INFO
  - `-Debug` 
    ![image-20190731143628923](http://ww2.sinaimg.cn/large/006tNc79gy1g5iyqso486j30yi0pdaf6.jpg)
  - 뒤에 로깅 수업때 자세히 살펴볼 예정 `
- FailureAnalyzer  
  : 어떤 어플리케이션 Error를 이쁘게 보기 위해서 존재한다.
- 배너  
  : 어플리케이션을 실행할 때 뜨는 것  
  ![image-20190731143736885](http://ww3.sinaimg.cn/large/006tNc79gy1g5iyrxa8huj30bk04wt8p.jpg)
  - banner.txt | gif | jpg | png
  - classpath 또는 spring.banner.location
  - ${spring-boot.version} 등의 변수를 사용할 수 있음.
  - Banner 클래스 구현하고 SpringApplication.setBanner()로 설정 가능.
  - 배너 끄는 방법  
    `app.setBannerMode(Banner.Mode.OFF);`
  - ![image-20190731143857455](http://ww2.sinaimg.cn/large/006tNc79gy1g5iytbn8c2j30px0oggqw.jpg)
- SpringApplicationBuilder로 빌더 패턴 사용 가능  
  ![image-20190731144834577](http://ww4.sinaimg.cn/large/006tNc79gy1g5iz3c18zuj30pp0cgq58.jpg)

---

# SpringApplication #2

https://docs.spring.io/spring-boot/docs/current/reference/html/boot-features-spring-application.html#boot-features-application-events-and-listeners

- ApplicationEvent 등록

  - ApplicationContext를 만들기 전에 사용하는 리스너는 @Bean으로 등록할 수 없다.  

    ```java
    public class SimpleListener implements ApplicationListener<ApplicationStartingEvent> {
        @Override
        public void onApplicationEvent(ApplicationStartingEvent event) {
            System.out.println("A");
        }
    }
    
    ---
      
    @Components  
    public class SimpleListener implements ApplicationListener<ApplicationStartedEvent> {
        @Override
        public void onApplicationEvent(ApplicationStartingEvent event) {
            System.out.println("A");
        }
    }
    
    
    ```

    - `SpringApplication.addListners(new SimpleListener); `

- WebApplicationType 설정  

  - `SpringApplication.setWebApplicationType(WebApplicationType.REACTIVE);`
  - 

- 애플리케이션 Arguments 사용하기

  - `-D` 는 VM options, `--foo` 는 프로그램 Arguments
  - ApplicationArguments를 빈으로 등록해 주니까 가져다 쓰면 됨.
  - `JVM options` 은 아예 받지 못함.

- 애플리케이션 실행한 뒤 뭔가 실행하고 싶을 때

  - `ApplicationRunner` (추천) 또는 `CommandLineRunner`   
    왜 `ApplicationRunner`가 더 좋은가?  `ApplicationArgument` 으로 받기 때문에 더 좋다.
  - 순서 지정 가능 `@Order`   
    숫자 낮은 것이 우선순위가 높은 것

---

# 외부 설정 #1

https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#boot-features-external-config 

## 사용할 수 있는 외부 설정

- properties
- YAML
- 환경 변수
- 커맨드 라인 아규먼트



**프로퍼티 우선 순위**

1. 유저 홈/테스트 디렉토리에 있는 spring-boot-dev-tools.properties  
  - ![image-20190731154053529](http://ww1.sinaimg.cn/large/006tNc79gy1g5j0lrmfk6j309a0h2tc6.jpg)
2. 테스트에 있는 @TestPropertySource  

   - ![image-20190731153328864](http://ww4.sinaimg.cn/large/006tNc79gy1g5j0e2ppm3j30hs0ar78s.jpg)
- ![image-20190731153653495](http://ww1.sinaimg.cn/large/006tNc79gy1g5j0hlx5mtj30qh03nq6i.jpg)
   - test - resources 파일 생성 
3. @SpringBootTest 애노테이션의 properties 애트리뷰트  

   - `@Value`  
   - ![image-20190731153051386](http://ww2.sinaimg.cn/large/006tNc79gy1g5j0bbums0j30gf0bajwa.jpg)
4. 커맨드 라인 아규먼트  
   ![image-20190731152407566](http://ww2.sinaimg.cn/large/006tNc79gy1g5j04bv9owj30b10cpmzq.jpg)
5. SPRING_APPLICATION_JSON (환경 변수 또는 시스템 프로티) 에 들어있는 프로퍼티
6. ServletConfig 파라미터
7. ServletContext 파라미터
8. java:comp/env JNDI 애트리뷰트
9. System.getProperties() 자바 시스템 프로퍼티
10. OS 환경 변수
11. RandomValuePropertySource
12. JAR 밖에 있는 특정 프로파일용 application properties
13. JAR 안에 있는 특정 프로파일용 application properties
14. JAR 밖에 있는 application properties
15. JAR 안에 있는 application properties
16. @PropertySource
17. 기본 프로퍼티 (SpringApplication.setDefaultProperties)  
   - 아무 것도 사용하지 않았을 때.



- Module에 - Test Resources 설정

- application.properties 를 Test 패키지의 application.properties 로 넣기.

![image-20190731152743172](http://ww4.sinaimg.cn/large/006tNc79gy1g5j082477cj30h208vwhq.jpg)



**application.properties 우선 순위 (높은게 낮은걸 덮어 씁니다.)**

1. file:./config/
2. file:./
3. classpath:/config/
4. classpath:/
  

**랜덤값 설정하기**

- ${random.*}
  

**플레이스 홀더**

- name = keesun
- fullName = ${name} baik

---

# 외부설정 #2

## 타입-세이프 프로퍼티 

### @ConfigurationProperties

- 여러 프로퍼티를 묶어서 읽어올 수 있음
- 빈으로 등록해서 다른 빈에 주입할 수 있음 - `@Autowire`
  - @EnableConfigurationProperties 를 Application.class 에 작성.
  - **@Component**
  - @Bean
  - ![image-20190731155523236](http://ww1.sinaimg.cn/large/006tNc79gy1g5j10u87udj3063028t94.jpg)
  - `random.int(0, 100) 안에 공백이 있으면 안됨!`
- Propertis안에 적는 내용을 융통성 있게 바인딩
  - context-path (케밥)
  - context_path (언드스코어)
  - contextPath (캐멀)
  - CONTEXTPATH

- 프로퍼티 타입 컨버전
  - 기본적인 Properties 안의 내용이 타입에 따라 변경됨.
  - **@DurationUnit** - 시간 정보를 받고 싶을 때 활용한다.
- 프로퍼티 값 검증
  - @Validated
  - JSR-303 (@NotNull, ...)
- 메타 정보 생성  
  - @NotNull
  - @NotEmpty
  - ...
- @Value@
  - SpEL 을 사용할 수 있지만.
  - 그러나, 위에 있는 기능들은 전부 사용 못합니다.

---

### Profile

@Profile 애노테이션은 어디에?

- @Configuration
- @Component

어떤 프로파일을 활성화 할 것인가?

- spring.profiles.active=prod
- 등등 커멘트라인으로 어떤 Profile을 활성화시킬지 정할 수 있음.

어떤 프로파일을 추가할 것인가?

- spring.profiles.include
- 약간 계층 구조로-

프로파일용 프로퍼티

- application-{profile}.properties

여기 부분은 연습이 필요하다고 판단된다. 환경 변수등의 설정을 할 때 용이하다고 판단됨.

---

# 로깅 1부 - 스프링 부트 기본 로거 설정

로깅 퍼사드 VS 로거

- **Commons Logging**, SLF4j  => 로깅 퍼사드  
  : 실제 로깅을 하는게 아니라 추상화 해놓은 것이다.
- JUL, Log4J2, **Logback**

스프링 5에 로거 관련 변경 사항

- https://docs.spring.io/spring/docs/5.0.0.RC3/spring-framework-reference/overview.html#overview-logging
- **Spring-JCL**
  - Commons Logging -> SLF4j or Log4j2
  - pom.xml에 exclusion 안해도 됨.

![image-20190731164805772](http://ww1.sinaimg.cn/large/006tNc79gy1g5j2jprye5j30kf094qai.jpg)



스프링 부트 로깅

- **기본 포맷**
- --debug (일부 핵심 라이브러리만 디버깅 모드로)
- --trace (전부 다 디버깅 모드로)
- **컬러 출력**: spring.output.ansi.enabled
- **파일 출력**: logging.file 또는 logging.path
- **로그 레벨 조정**: logging.level.패지키 = 로그 레벨



***특정 포맷으로 로그를 설정해야되는 경우 어떻게 해야 될까?***



https://docs.spring.io/spring-boot/docs/current/reference/html/howto-logging.html 

커스텀 로그 설정 파일 사용하기

- Logback: logback-spring.xml
- Log4J2: log4j2-spring.xml
- JUL (비추): logging.properties
- Logback extension
  - 프로파일 \<springProfile name="프로파일">
  - Environment 프로퍼티 \<springProperty>
- 로거를 Log4j2로 변경하기
  - [https://docs.spring.io/spring-boot/docs/current/reference/html/howto-logging.html#howto-configure-log4j-for-logging](https://docs.spring.io/spring-boot/docs/current/reference/html/howto-logging.html#howto-configure-log4j-for-logging)
  - ![image-20190731170244864](http://ww4.sinaimg.cn/large/006tNc79gy1g5j2yxqut9j30kg0htk1d.jpg)



![image-20190731170128356](http://ww4.sinaimg.cn/large/006tNc79gy1g5j2xlxlg2j30pz09n0uk.jpg)