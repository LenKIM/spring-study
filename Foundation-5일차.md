# HttpMessageConverters

HTTP 요청 본문을 객체로 변경하거나, 객체를 HTTP 응답 본문으로 변경할 때 사용

 *{“username”:”keesun”, “password”:”123”} <-> User*

- *@ReuqestBody*
- *@ResponseBody*



 *JSON 맵핑을 할 때는 Getter/Setter 가 존재해야지만 맵핑이 된다.*

```java
package com.example.demospringmvc;

import com.example.demospringmvc.user.UserController;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.WebMvcTest;
import org.springframework.http.MediaType;
import org.springframework.test.context.junit4.SpringRunner;
import org.springframework.test.web.servlet.MockMvc;

import static org.hamcrest.Matchers.equalTo;
import static org.hamcrest.Matchers.is;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.get;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.post;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.*;

@RunWith(SpringRunner.class)
@WebMvcTest(UserController.class)
public class UserControllerTest {

    @Autowired
    MockMvc mockMvc;

    @Test
    public void hello() throws Exception {
        mockMvc.perform(get("/hello"))
                .andExpect(status().isOk())
                .andExpect(content().string("hello"));
    }

    @Test
    public void createUser_JSON() throws Exception {
        String userJson = "{\"username\":\"len\", \"password\":\"1234\"}";
        mockMvc.perform(post("/users/create")
                .contentType(MediaType.APPLICATION_JSON_UTF8)
                .accept(MediaType.APPLICATION_JSON_UTF8)
                .content(userJson))
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.username",
                        is(equalTo("len"))))
                .andExpect(jsonPath("$.password",
                        is(equalTo("1234"))));
    }
}
```

---

# 웹 MVC - ViewResolve

- 뷰 리졸버 설정 제공
- HttpMessageConvertersAutoConfiguration

**XML 메시지 컨버터 추가하기**



![image-20190802164245891](http://ww1.sinaimg.cn/large/006tNc79gy1g5ldms6aotj30pr0gpdiw.jpg)



# 정적 리소스 지원

**정적 리소스 맵핑 “ /**”**

- 기본 리소스 위치
  - classpath:/static
  - classpath:/public
  - classpath:/resources/
  - classpath:/META-INF/resources
  - 예) “/hello.html” => /static/hello.html
  - spring.mvc.static-path-pattern: 맵핑 설정 변경 가능
  - spring.mvc.static-locations: 리소스 찾을 위치 변경 가능
- Last-Modified 헤더를 보고 304 응답을 보냄.
- ResourceHttpRequestHandler가 처리함.
  - WebMvcConfigurer의 addRersourceHandlers로 커스터마이징 할 수 있음

```java
@Override
public void addResourceHandlers(ResourceHandlerRegistry registry) {
  registry.addResourceHandler("/m/**")
    .addResourceLocations("classpath:/m/")
    .setCachePeriod(20);
```

꼭 끝나는 부분은 `/` 로 끝나야 한다! 실수 ㄴㄴ

---

# 웹 JAR

웹JAR 맵핑 “ /webjars/**”

- 버전 생략하고 사용하려면
  - webjars-locator-core 의존성 추가

```java
<script src="/webjars/jquery/dist/jquery.min.js"></script>
<script>
   $(function() {
       console.log("ready!");
   });
</script>
```



---

# index 페이지와 파비콘

- index.html 찾아 보고 있으면 제공.
- index.템플릿 찾아 보고 있으면 제공.
- 둘 다 없으면 에러 페이지.



## 파비콘

- favicon.ico
- 파이콘 만들기 https://favicon.io/
- 파비콘이 안 바뀔 때?
  - https://stackoverflow.com/questions/2208933/how-do-i-force-a-favicon-refresh

---



# 스프링 시큐리티

- 웹 시큐리티
- 메소드 시큐리티
- 다양한 인증 방법 지원
  - LDAP, 폼 인증, Basic 인증, OAuth, ...

#### 스프링 부트 시큐리티 자동 설정

- SecurityAutoConfiguration
- UserDetailsServiceAutoConfiguration
- spring-boot-starter-security
  - 스프링 시큐리티 5.* 의존성 추가
- 모든 요청에 인증이 필요함.
- 기본 사용자 생성
  - Username: user
  - Password: 애플리케이션을 실행할 때 마다 랜덤 값 생성 (콘솔에 출력 됨.)
  - spring.security.user.name
  - spring.security.user.password
- 인증 관련 각종 이벤트 발생
  - DefaultAuthenticationEventPublisher 빈 등록
  - 다양한 인증 에러 핸들러 등록 가능

#### 스프링 부트 시큐리티 테스트

- https://docs.spring.io/spring-security/site/docs/current/reference/html/test-method.html



스프링부트 시큐리티 - Basic Authorized 인증을 필요하게 된다.

Baisc Authenticate.

![image-20190802115130373](http://ww3.sinaimg.cn/large/006tNc79gy1g5l57r9d5zj30n30iptca.jpg)



![image-20190802115305652](http://ww3.sinaimg.cn/large/006tNc79gy1g5l59d7h3xj30me0ftgnp.jpg)



기본

![image-20190802115407520](http://ww3.sinaimg.cn/large/006tNc79gy1g5l5afblwbj30l3096my9.jpg)

---

# 웹 시큐리티 설정

```java
@Configuration
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {
   @Override
   protected void configure(HttpSecurity http) throws Exception {
       http.authorizeRequests()
               .antMatchers("/", "/hello").permitAll()
               .anyRequest().authenticated()
               .and()
           .formLogin() //accepts 해야되 근데 Login하면 
               .and()
           .httpBasic(); //html이 없는 경우 httpbasic에서 걸린다.
   }
}
```

