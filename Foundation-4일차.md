# 테스트

시작은 일단 spring-boot-starter-test를 추가하는 것 부터

- test 스콥으로 추가

**@SpringBootTest**

- @RunWith(SpringRunner.class)랑 같이 써야 함.

- 빈 설정 파일은 설정을 안해주나? 알아서 찾습니다. (@SpringBootApplication)

```java
@RunWith(SpringRunner.class)
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.MOCK)
@AutoConfigureMockMvc
public class SampleControllerTest {
}  
```

- webEnvironment  
  `@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.MOCK)`

  - **MOCK**: mock servlet environment. 내장 톰캣 구동 안 함. 서블릿과 유사한 목업을 띄웁니다.  

    ```java
    
    import org.junit.Test;
    import org.junit.runner.RunWith;
    import org.springframework.beans.factory.annotation.Autowired;
    import org.springframework.boot.test.autoconfigure.web.servlet.AutoConfigureMockMvc;
    import org.springframework.boot.test.context.SpringBootTest;
    import org.springframework.test.context.junit4.SpringRunner;
    import org.springframework.test.web.servlet.MockMvc;
    
    import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.get;
    import static org.springframework.test.web.servlet.result.MockMvcResultHandlers.print;
    import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.content;
    import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;
    
    @RunWith(SpringRunner.class)
    @SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.MOCK)
    @AutoConfigureMockMvc
    public class SampleControllerTest {
    
        @Autowired
        MockMvc mockMvc;
    
        @Test
        public void hello() throws Exception {
            mockMvc.perform(get("/hello"))
                    .andExpect(status().isOk())
                    .andExpect(content().string("hello Len"))
                    .andDo(print());
        }
    }
    
    
    ```

  - RANDON_PORT, DEFINED_PORT: 내장 톰캣 사용 함.   

    ```java
    @RunWith(SpringRunner.class)
    @SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
    @AutoConfigureMockMvc
    public class SampleControllerTest {
    
        @Autowired
        TestRestTemplate testRestTemplate;
    
        @Test
        public void hello() throws Exception {
            String result = testRestTemplate.getForObject("/hello", String.class);
            assertThat(result).isEqualTo("helloLen");
        }
    }
    
    ```

  - NONE: 서블릿 환경 제공 안 함.

  - webFlux로 테스트하기.

  ```java
  @RunWith(SpringRunner.class)
  @SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
  @AutoConfigureMockMvc
  public class SampleControllerTest {
  
      //어싱크노로스하게 동작함.
      @Autowired
      WebTestClient webTestClient;
  
      //만약 서비스가 아니라 컨트롤러까지만 가고싶다면?
      @MockBean
      SampleSerive sampleSerive;
  
      @Test
      public void hello() throws Exception {
  
          when(sampleSerive.getName()).thenReturn("Go-Go");
  
          webTestClient.get().uri("/hello").exchange()
                  .expectStatus().isOk()
                  .expectBody(String.class).isEqualTo("helloGo-GO");
      }
  }
  ```

  

@MockBean슬라이스 테스트

- 레이어 별로 잘라서 테스트하고 싶을 때  

  ```java
  //만약 서비스가 아니라 컨트롤러까지만 가고싶다면?
  @RunWith(SpringRunner.class)
  @SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
  @AutoConfigureMockMvc
  public class SampleControllerTest {
    
  		@Rule
      public OutputCapture outputCapture = new OutputCapture();
  
      @Autowired
      TestRestTemplate testRestTemplate;
  
      @MockBean
      SampleSerive sampleSerive;
  
      @Test
      public void hello() throws Exception {
  
          when(sampleSerive.getName()).thenReturn("Go-Go");
  
          String result = testRestTemplate.getForObject("/hello", String.class);
          assertThat(result).isEqualTo("helloGo-Go");
  
      }
  }
  ```

- @JsonTest  
  ![image-20190801164525940](http://ww1.sinaimg.cn/large/006tNc79gy1g5k83qsv6qj30pt0mm41x.jpg)

- @WebMvcTest  
  : Web과 관련된 컴포넌트만 주입받을 수 있을 것이다.  
  `bean` 하나만 테스트할 수 있당.

- @WebFluxTest

- @DataJpaTest

- ...

---

# 테스트 유틸

- **OutputCapture**  
  :Log를 비롯해 모든 것을 캡쳐한다.
- TestPropertyValues
- TestRestTemplate
- ConfigFileApplicationContextInitializer



---

# Spring-Boot-Devtools

https://docs.spring.io/spring-boot/docs/current/reference/html/using-boot-devtools.html

- 캐시 설정을 개발 환경에 맞게 변경.  
  ![image-20190801165939673](/Users/lenkim/Library/Application Support/typora-user-images/image-20190801165939673.png)
- 클래스패스에 있는 파일이 변경 될 때마다 자동으로 재시작.
  - 직접 껐다 켜는거 (cold starts)보다 빠른다. 왜?  
    `class Loader 를 두 개 쓴다.`
  - 릴로딩 보다는 느리다. (JRebel 같은건 아님)
  - 리스타트 하고 싶지 않은 리소스는? spring.devtools.restart.exclude
  - 리스타트 기능 끄려면? spring.devtools.restart.enabled = false
- 라이브 릴로드? 리스타트 했을 때 브라우저 자동 리프레시 하는 기능
  - 브라우저 플러그인 설치해야 함.
  - 라이브 릴로드 서버 끄려면? spring.devtools.liveload.enabled = false
- 글로벌 설정
  - ~/.spring-boot-devtools.properties  // 1순위 
- 리모트 애플리케이션

---

# 스프링 웹 MVC

- 스프링 웹 MVC
  - https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/web.html#spring-web
- 스프링 부트 MVC
  - 자동 설정으로 제공하는 여러 기본 기능 (앞으로 살펴볼 예정)
- 스프링 MVC 확장
  - @Configuration + WebMvcConfigurer
- 스프링 MVC 재정의
  - @Configuration + @EnableWebMvc