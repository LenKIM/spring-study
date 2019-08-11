# 2부: 스프링 데이터 JPA 활용

## 15. 스프링 데이터 JPA 활용 파트 소개



![img](http://ww4.sinaimg.cn/large/006tNc79gy1g59r4sc53nj30kd0c6wfv.jpg)


| 스프링 데이터         | SQL & NoSQL 저장소 지원 프로젝트의 묶음.                     |
| --------------------- | ------------------------------------------------------------ |
| 스프링 데이터 Common  | 여러 저장소 지원 프로젝트의 공통 기능 제공.                  |
| 스프링 데이터 REST    | 저장소의 데이터를 하이퍼미디어 기반 HTTP 리소스로(REST API로) 제공하는 프로젝트. |
| 스프링 데이터 **JPA** | 스프링 데이터 Common이 제공하는 기능에 **JPA** 관련 기능 추가. |



## 16.스프링 데이터 Common: Repository

![image-20190723154424863](http://ww3.sinaimg.cn/large/006tNc79gy1g59rqz60pzj30gt0aotad.jpg)



```java
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.orm.jpa.DataJpaTest;
import org.springframework.data.domain.Page;
import org.springframework.data.domain.PageRequest;
import org.springframework.test.annotation.Rollback;
import org.springframework.test.context.junit4.SpringRunner;

import java.util.List;

import static org.assertj.core.api.Assertions.assertThat;

@RunWith(SpringRunner.class)
@DataJpaTest
public class PostRepository2Test {
//레포지토리만 설정할 수 있음. h2database를 설정하면, 메모리디비를 쓸수 있음 더빠름.
    @Autowired
    PostRepository2 postRepository2;

    @Test
    @Rollback(false)
    public void crudRepository(){

        //Given
        Post post = new Post();
        post.setTitle("Hello World");
        assertThat(post.getId()).isNull();

//        When
        Post newPost = postRepository2.save(post);
//        Then
        assertThat(newPost.getId()).isNotNull();

//        when
        List<Post> posts = postRepository2.findAll();
//        Then
        assertThat(posts.size()).isEqualTo(1);
        assertThat(posts).contains(newPost);

//        When
        Page<Post> page = postRepository2.findAll(PageRequest.of(0,10));
        assertThat(page.getTotalElements()).isEqualTo(1);
        assertThat(page.getNumber()).isEqualTo(0);
        assertThat(page.getSize()).isEqualTo(10);
        assertThat(page.getNumberOfElements()).isEqualTo(1);
        page = postRepository2.findByTitleContains("Hello", PageRequest.of(0, 10));

        assertThat(page.getTotalElements()).isEqualTo(1);
        assertThat(page.getNumber()).isEqualTo(0);
        assertThat(page.getSize()).isEqualTo(10);
        assertThat(page.getNumberOfElements()).isEqualTo(1);
    }
}
```



JPA를 테스트 할 때는 `@DataJpaTest` 를 넣고, 

```java
 <dependency>
            <groupId>com.h2database</groupId>
            <artifactId>h2</artifactId>
            <scope>test</scope>
        </dependency>
```

포함하여, 메모리 DB를 활용하게 끔 설정해, Postgres를 사용하지 않고 테스트 할 수 있도록 한다.

` @Rollback(false)`  을 위 코드에 만약 넣지 않는다면, `JPA`는  데이터를 SAVE하고 쓰지 않았기 때문에 롤백하게 된다.  `@DataJpaTest ` 안에 `@Transactional` 가 명시되어 있다.

또한, Pageable 에 대해서 학습이 진행되었다.

```java
public interface PostRepository2 extends JpaRepository<Post, Long> {

    Page<Post> findByTitleContains(String title, Pageable pageable);

}
```

뒤에 `pageable` 을 넣어 페이징네이션 기능도 동작하게 만들 수도 있다.

> page = postRepository2.findByTitleContains("Hello", PageRequest.of(0, 10));

`PageRequest.of` 이런 식으로 팩터리 패턴으로 PageRequest를 만들 수 있다.

---

# 17. 스프링 데이터 Common: Repository 인터페이스 정의하기 

Repository 인터페이스로 공개할 메소드를 직접 일일히 정의하고 싶다면 특정 리포지토리 당 

- @RepositoryDefinition 

```java
@RepositoryDefinition(domainClass = Comment.class, idClass = Long.class)
public interface CommentRepository {
    Comment save(Comment comment);
    List<Comment> findAll();
}
```

공통 인터페이스 정의하고 싶다면 아래와 같이 작성한다.

-  @NoRepositoryBean 

```java
@NoRepositoryBean
public interface MyRepository<T, ID extends Serializable> extends Repository<T, ID> {
    <E extends T> E save(E entity);
    List<T> findAll();
  	long count();
}

public interface CommentRepository extends MyRepository<Comment, Long>{

}
```

그러고 난 후 테스트는 다음과 같이 할 수 있다.

```java
package com.example.springboot.jpa.springbootjpa;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.orm.jpa.DataJpaTest;
import org.springframework.test.context.junit4.SpringRunner;

import java.util.List;

import static org.assertj.core.api.Assertions.assertThat;


@RunWith(SpringRunner.class)
@DataJpaTest
public class CommentRepositoryTest {

    @Autowired
    CommentRepository commentRepository;

    @Test
    public void crud(){
        Comment comment = new Comment();
        comment.setComment("Helo comment");
        commentRepository.save(comment);

        List<Comment> all = commentRepository.findAll();
        assertThat(all.size()).isEqualTo(1);

        long l = commentRepository.count();
        assertThat(l).isEqualTo(1);
    }

}
```

# 18. 스프링 데이터 Common: Null 처리하기

**스프링 데이터 2.0 부터 자바 8의 Optional 지원.** 

- Optional<Post> findById(Long id); 

콜렉션은 Null을 리턴하지 않고, 비어있는 콜렉션을 리턴합니다. 

스프링 프레임워크 5.0부터 지원하는 Null 애노테이션 지원. 

- @NonNullApi, @NonNull, @Nullable. 
- 런타임체크지원함. 
- JSR 305 애노테이션을 메타 애노테이션으로 가지고 있음. (IDE 및 빌드 툴 지원) 

인텔리J 설정 - 요건 옵션, `NotNull`, `NonNull` 이 인텔리J에 들어가 있지 않기 때문에 추가 시켜줌.

- Build, Execution, Deployment 
  - Compile
    - Add runtime assertion for notnull-annotated methods and parameters 

![image-20190723164742303](http://ww2.sinaimg.cn/large/006tNc79gy1g59tkvrralj30he0cf42d.jpg)

---

`<E extends T>Optional<E> findById(Id id);` 리턴 되는 값이 Null이여도 빈 컬렉션이 리턴된다.

또한 `if(check(null)) ` 보다는 Optional을 활용해서 `Comment comment = byId.orElseThrow(IllegalArgumentException::new);` 해주는 게 더 깔끔하다고 판단된다.



그리고 `@NonNullApi`,` @NonNull`, `@Nullable` 가 사용될 수 있는데, `NonNullApi ` 요건 Package-info.java에서 나머지는 런타임 과정에서 

```java
<E extends T> E save(@NonNull E entity);

    List<T> findAll();

    long count();

    @Nullable
    <E extends T> Optional<E> findById(Id id);
```

와 같이 해서 Warnning을 띄어 찾아 줄 수 있다.



# 스프링 데이터 Common: 쿼리 만들기 개요 

**스프링 데이터 저장소의 메소드 이름으로 쿼리 만드는 방법** 

- 메소드 이름을 분석해서 쿼리 만들기 (CREATE)   
  `List<Comment> findByTitleContains(String keyword);`

- 미리 정의해 둔 쿼리 찾아 사용하기 (USE_DECLARED_QUERY)   

  ```java
  @Query(value = "SELECT c FROM Commnet AS c", nativeQuery = true)
  List<Comment> findByTitleContains(String keyword);
  ```

- 미리 정의한 쿼리 찾아보고 없으면 만들기 (CREATE_IF_NOT_FOUND)   

  ![image-20190723172907385](http://ww1.sinaimg.cn/large/006tNc79gy1g59urwrz4sj30h503jjs2.jpg)
  
  

**쿼리 만드는 방법** 

- 리턴타입  {접두어}{도입부}By{프로퍼티 표현식}(조건식)[(And|Or){프로퍼티 표현식}(조건식)]{정렬 조건} (매개변수)  
  `List<Comment> findByTitleContains(String keyword);`



| 접두어          | Find, Get, Query, Count, ...                                 |
| --------------- | ------------------------------------------------------------ |
| 도입부          | Distinct, First(N), Top(N)                                   |
| 프로퍼티 표현식 | Person.Address.ZipCode => find(Person)ByAddress_ZipCode(...) |
| 조건식          | IgnoreCase, Between, LessThan, GreaterThan, Like, Contains, ... |
| 정렬 조건       | OrderBy{프로퍼티}Asc\|Desc                                   |
| 리턴 타입       | E, Optional<E>, List<E>, Page<E>, Slice<E>, Stream<E>        |
| 매개변수        | Pageable, Sort                                               |

**쿼리 찾는 방법** 

- 메소드 이름으로 쿼리를 표현하기 힘든 경우에 사용. 
- 저장소 기술에 따라 다름. 
- JPA: 1순위 @Query 2순위 @NamedQuery 

```java
@Override
protected RepositoryQuery resolveQuery(JpaQueryMethod method, EntityManager em, NamedQueries namedQueries) {

			RepositoryQuery query = JpaQueryFactory.INSTANCE.fromQueryAnnotation(method, em, evaluationContextProvider);

			if (null != query) {
				return query;
			}

			query = JpaQueryFactory.INSTANCE.fromProcedureAnnotation(method, em);

			if (null != query) {
				return query;
			}

			String name = method.getNamedQueryName();
			if (namedQueries.hasQuery(name)) {
				return JpaQueryFactory.INSTANCE.fromMethodWithQueryString(method, em, namedQueries.getQuery(name),
						evaluationContextProvider);
			}

			query = NamedQuery.lookupFrom(method, em);

			if (null != query) {
				return query;
			}

			throw new IllegalStateException(
					String.format("Did neither find a NamedQuery nor an annotated query for method %s!", method));
		}
	}
```

# 20. 스프링 데이터 Common: 쿼리 만들기 실습 



**기본** **예제**



List<Person> findByEmailAddress**And**Lastname(EmailAddress emailAddress, String lastname);

// distinct

List<Person> find**Distinct**PeopleByLastname**Or**Firstname(String lastname, String firstname);

List<Person> findPeople**Distinct**ByLastname**Or**Firstname(String lastname, String firstname);

// ignoring case

List<Person> findByLastname**IgnoreCase**(String lastname);

// ignoring case

List<Person> findByLastnameAndFirstname**AllIgnoreCase**(String lastname, String firstname);



**정렬**



List<Person> findByLastname**OrderBy**Firstname**Asc**(String lastname);

List<Person> findByLastname**OrderBy**Firstname**Desc**(String lastname);



**페이징**



**Page**<User> findByLastname(String lastname, **Pageable** pageable);

**Slice**<User> findByLastname(String lastname, **Pageable** pageable);

**List**<User> findByLastname(String lastname, **Sort** sort);

**List**<User> findByLastname(String lastname, **Pageable** pageable);



**스트리밍**



**Stream**<User> readAllByFirstnameNotNull();

- try-with-resource 사용할 것. (Stream을 다 쓴다음에 close() 해야 함)



![image-20190723180829544](http://ww1.sinaimg.cn/large/006tNc79gy1g59vwvlkntj30o406nq3p.jpg)



# 21. 스프링 데이터 Common: 비동기 쿼리 

비동기 쿼리 
<User> findByFirstname(String firstname);
<User> findOneByFirstname(String firstname);
<User> findOneByLastname(String lastname);

- 해당 메소드를 스프링 TaskExecutor에 전달해서 별도의 쓰레드에서 실행함. 

- Reactive랑은 다른 것임 

  권장하지 않는 이유 

- 테스트 코드 작성이 어려움. 

- 코드 복잡도 증가. 

- 성능상 이득이 없음. 

  ○ DB 부하는 결국 같고.
  ○ 메인 쓰레드 대신 백드라운드 쓰레드가 일하는 정도의 차이.
  ○ 단, 백그라운드로 실행하고 결과를 받을 필요가 없는 작업이라면 @Async를 

  사용해서 응답 속도를 향상 시킬 수는 있다.
  



# 22. 스프링 데이터 Common: 커스텀 리포지토리 

쿼리 메소드(쿼리 생성과 쿼리 찾아쓰기)로 해결이 되지 않는 경우 직접 코딩으로 구현 가능. 

1. - 스프링 데이터 리포지토리 인터페이스에 기능 추가. 
   - 스프링 데이터 리포지토리 기본 기능 덮어쓰기 가능. 
   - 구현방법 
     1. 커스텀 리포지토리 인터페이스 정의 
     2. 인터페이스 구현 클래스 만들기 (기본 접미어는 Impl) 
     3. 엔티티 리포지토리에 커스텀 리포지토리 인터페이스 추가 



```java
//Entity

@Entity
public class Post {

    @Id @GeneratedValue
    private Long id;

    private String title;

    @Lob //255자 넘을 경우 사용함.
    private String contents;

    @Temporal(TemporalType.TIMESTAMP)
    private Date created;

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getTitle() {
        return title;
    }

    public void setTitle(String title) {
        this.title = title;
    }

    public String getContents() {
        return contents;
    }

    public void setContents(String contents) {
        this.contents = contents;
    }

    public Date getCreated() {
        return created;
    }

    public void setCreated(Date created) {
        this.created = created;
    }
}

---
  
import org.springframework.data.jpa.repository.JpaRepository;

public interface PostRepository extends JpaRepository<Post, Long>, PostCustomRepository<Post>{

}

import java.util.List;

public interface PostCustomRepository<T> {

    List<Post> findMyPost();

    void delete(T entity);

}

---
  
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Repository;
import org.springframework.transaction.annotation.Transactional;

import javax.persistence.EntityManager;
import java.util.List;

@Repository
@Transactional
public class PostCustomRepositoryImpl implements PostCustomRepository<Post> {

    @Autowired
    EntityManager entityManager;

    @Override
    public List<Post> findMyPost() {
        return entityManager.createQuery("SELECT p FROM Post AS p", Post.class).getResultList();
    }

    @Override
    public void delete(Post entity) {
        System.out.println("custom delete");
        entityManager.remove(entity);
    }
}
```


**기능 추가하기**   

**기본 기능 덮어쓰기**  

**접미어 설정하기**  

```java
@SpringBootApplication
@EnableJpaRepositories(repositoryImplementationPostfix = "Impl")
public class SpringbootJpaApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringbootJpaApplication.class, args);
    }
}
```

접미어를 보고 Repository를 커스텀 할 수 있다.



# 23. 스프링 데이터 Common: 기본 리포지토리 커스터마이징

모든 리포지토리에 공통적으로 추가하고 싶은 기능이 있거나 덮어쓰고 싶은 기본 기능이 있다면 

1. JpaRepository를 상속 받는 인터페이스 정의  
   - @NoRepositoryBean 
2. 기본 구현체를 상속 받는 커스텀 구현체 만들기  
3. @EnableJpaRepositories에 설정  
   - repositoryBaseClass 

```java
@NoRepositoryBean
public interface MyRepository<T, ID extends Serializable> extends JpaRepository<T, ID> {
    boolean contains(T entity);
}
```

```java
public class SimpleMyRepository<T, ID extends Serializable> extends
SimpleJpaRepository<T, ID> implements MyRepository<T, ID> {
    private EntityManager entityManager;
    public SimpleMyRepository(JpaEntityInformation<T, ?> entityInformation,
EntityManager entityManager) {
        super(entityInformation, entityManager);
        this.entityManager = entityManager;
    }
    @Override
    public boolean contains(T entity) {
        return entityManager.contains(entity);
    }
}
```

```java
@EnableJpaRepositories(repositoryBaseClass = SimpleMyRepository.class)
```

```java
public interface PostRepository extends MyRepository<Post, Long> {
}
```



여러 레포지토리에 공통적으로 추가하고 싶은 기능이 있다면, 또는 덮어쓰고 싶은 기본 기능이 있다면 활용할 수 있을 것 같다.



# 24. 스프링 데이터 Common: 도메인 이벤트

### **스프링 프레임워크의 이벤트 관련 기능**

- https://docs.spring.io/spring/docs/current/spring-framework-reference/core.html#cont ext-functionality-events
- ApplicationContext extends **​ApplicationEventPublisher**
- 이벤트: extends ApplicationEvent
- 리스너
  - implements ApplicationListener<E extends ApplicationEvent>  

    ```java
    import org.springframework.context.ApplicationListener;
    
    public class PostListener implements ApplicationListener<PostPublishedEvent> {
    //    받았을 때 해야할 잇
        @Override
        public void onApplicationEvent(PostPublishedEvent event) {
            System.out.println("---------------------------");
            System.out.println(event.getPost().getTitle() + " is published!!");
            System.out.println("---------------------------");
    ```

  -  @EventListener  

    ```java
    public class PostListener{
    //    받았을 때 해야할 잇
        @EventListener
        public void onApplicationEvent(PostPublishedEvent event) {
            System.out.println("---------------------------");
            System.out.println(event.getPost().getTitle() + " is published!!");
            System.out.println("---------------------------");
        }
    }
    ```

    그러나 활용하기 위해서는 Listener을 Event Bean으로 등록 해줘야 한다.  

    ```java
    @Configuration
    public class PostListenerTestConfig {
    
        @Bean
        public PostListener postListener(){
            return new PostListener();
        }
    }
    ```

    빈 설정을 따로 만든 이유는 `@DataJpaTest` 가 있기 때문입니다.  
    또는 

    ```java
    @Configuration
    public class PostListenerTestConfig {
    
        @Bean
        public ApplicationListener<PostPublishedEvnet> postListener(){
            return event -> {
                      System.out.println("---------------------------");
            System.out.println(event.getPost().getTitle() + " is published!!");
            System.out.println("---------------------------");
            };
        }
    }
    ```

    

### 스프링 데이터의 도메인 이벤트 Publisher

- @DomainEvents - 이벤트를 모아놓는 곳

- @AfterDomainEventPublication - 다 모아놓은 상태에서 컬렉션을 비워야할 때 쓰는

  

- extends AbstractAggregateRoot<E> - 이  클래스 안에 위 두개가 모두 정의되어 있다.
  : AbstractAggregateRoot안에 `DomainEvents`가 실행되고, `AfterDomainEventPublication` 가 실행된다.

- 현재는 save() 할 때만 발생 합니다.

도메인의 엔티티 변화를 Listen하는 이벤트로 발생시켜서 어떤 리스너의 코드를 실행시키는 것을 말합니다. 그건 메뉴얼로 할 수 있다.



# 25. 스프링 데이터 Common: QueryDSL

이유? 조건문을 표현하는 방법이 Typesafe하다. 또한 조건문을 조합할 수도 있고 따로 관리하거나 지정할 수 있다.



`findByFirstNameIngoreCaseAndLastNameStartsWithIgnoreCase(String firstName, String lastName)`

이걸 한번에 이해하기 어렵다. 그러므로 QueryDSL이 도움을 준다.



여러 쿼리 메소드는 대부분 두 가지 중 하나. 

- Optional<T> findOne(**Predicate**): 이런 저런 조건으로 무언가 하나를 찾는다. 

- List<T>|Page<T>|.. findAll(**Predicate**): 이런 저런 조건으로 무언가 여러개를 찾는다. (또는 Paging)

- QuerydslPredicateExecutor 인터페이스 

  QueryDSL 

  ![image-20190726113033606](http://ww1.sinaimg.cn/large/006tNc79gy1g5d1a0jns3j30sv0a5gqi.jpg)

- http://www.querydsl.com/ 
- 타입세이프한쿼리만들수있게도와주는라이브러리 
- JPA, SQL, MongoDB, JDO, Lucene, Collection 지원 
- QueryDSL JPA 연동 가이드 

스프링 데이터 JPA + QueryDSL 

- 인터페이스: QuerydslPredicateExecutor<T> 
- 구현체: QuerydslPredicateExecutor<T> 

연동 방법 

- 기본 리포지토리 커스터마이징 안 했을 때.
- **기본 리포지토리 커스타마이징 했을 때.  (느무느무 어렵다 얘는...)**

의존성 추가

```xml
<dependency>
    <groupId>com.querydsl</groupId>
    <artifactId>querydsl-apt</artifactId>
</dependency>
<dependency>
    <groupId>com.querydsl</groupId>
    <artifactId>querydsl-jpa</artifactId>
</dependency>
```

1. 의존성 추가 

 

```xml
<dependency>
    <groupId>com.querydsl</groupId>
    <artifactId>querydsl-apt</artifactId>
</dependency>
<dependency>

    <groupId>com.querydsl</groupId>
    <artifactId>querydsl-jpa</artifactId>
</dependency>
<plugin>
                <groupId>com.mysema.maven</groupId>
                <artifactId>apt-maven-plugin</artifactId>
                <version>1.1.3</version>
                <executions>
                    <execution>
                        <goals>
                            <goal>process</goal>
                        </goals>
                        <configuration>
<outputDirectory>target/generated-sources/java</outputDirectory>
<processor>com.querydsl.apt.jpa.JPAAnnotationProcessor</processor>
                        </configuration>
                    </execution>
                </executions>
  </plugin> 

----

public interface AccountRepository extends JpaRepository<Account, Long>, QuerydslPredicateExecutor<Account> {
 } 
```



# 26. 스프링 데이터 Common: Web 1부: 웹 지원
기능 소개



스프링 데이터 웹 지원 기능 설정 

- 스프링 부트를 사용하는 경우에.. 설정할 것이 없음. (자동 설정) 
- 스프링 부트 사용하지 않는 경우? 

```java
@Configuration 
@EnableWebMvc 
@EnableSpringDataWebSupport 
class WebConfiguration {}
```

제공하는 기능
- **도메인 클래스 컨버터**
- **@RequestHandler 메소드에서 Pageable과 Sort 매개변수 사용**
- **Page 관련 HATEOAS(헤이토스) 기능 제공**
	- PagedResourcesAssembler
	- PagedResoure

---

- **Payload 프로젝션**
	- 요청으로 들어오는 데이터 중 일부만 바인딩 받아오기
	- @ProjectedPayload, @XBRead, @JsonPath
- **요청 쿼리 매개변수를 QueryDSLdml Predicate로 받아오기**
	- ?firstname=Mr&lastname=White => Predicate



HATEOAS 이란?



# 27. 스프링 데이터 Common: Web 2부:
DomainClassConverter

### 스프링 Converter 

: Long에서 다른 엔티티로 변환시킬 수 있는 것을 Converter라고 한다.

- https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework 

  /core/convert/converter/Converter.html 


  ```java
  import org.springframework.beans.factory.annotation.Autowired;
  import org.springframework.web.bind.annotation.GetMapping;
  import org.springframework.web.bind.annotation.PathVariable;
  import org.springframework.web.bind.annotation.RestController;
  
  @RestController
  public class PostController {
  
      @Autowired
      PostRepository postRepository;
  
      //생성자가 하나인 경우에만 Repo를 주입해준다.
  //    public PostController(    PostRepository postRepository) {
  
  //    }
  
      @GetMapping("/posts/{id}")
      public String getPost(@PathVariable("id") Post post){
          return post.getTitle();
      }
  
      //    public Post getPost(@PathVariable Long id){
  //        Optional<Post> byId = postRepository.findById(id);
  //        Post post = byId.get();
  //        return post;
  }
  ---Test Code
  import org.junit.Test;
  import org.junit.runner.RunWith;
  import org.springframework.beans.factory.annotation.Autowired;
  import org.springframework.boot.test.autoconfigure.web.servlet.AutoConfigureMockMvc;
  import org.springframework.boot.test.context.SpringBootTest;
  import org.springframework.test.context.ActiveProfiles;
  import org.springframework.test.context.junit4.SpringRunner;
  import org.springframework.test.web.servlet.MockMvc;
  
  import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.get;
  import static org.springframework.test.web.servlet.result.MockMvcResultHandlers.print;
  import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;
  import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.content;
  
  @RunWith(SpringRunner.class)
  @SpringBootTest
  @AutoConfigureMockMvc
  @ActiveProfiles("test")
  public class PostControllerTest {
  
      @Autowired
      MockMvc mockMvc;
  
      @Autowired
      PostRepository postRepository;
  
      @Test
      public void getPost() throws Exception {
  
          Post post = new Post();
          post.setTitle("Jpa");
          postRepository.save(post);
  
          mockMvc.perform(get("/posts/1"))
                  .andDo(print())
                  .andExpect(status().isOk())
                  .andExpect(content().string("Jpa"));
      }
  }
  ```

- Formatter도 들어 본 것 같은데... 

  - Formatter는 문자열로만 한정되어 있다.

---

# 28. 스프링 데이터 Common: Web 3부: Pageable과 Sort 매개변수 

스프링 MVC HandlerMethodArgumentResolver 

- 스프링MVC핸들러메소드의 매개변수로 받을수 있는 객체를 확장하고 싶을 때 

  사용하는 인터페이스 

- https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/web/method/support/HandlerMethodArgumentResolver.html 

페이징과 정렬 관련 매개변수 

- page: 0부터 시작. 
- size: 기본값 20. 
- sort: property,property(,ASC|DESC) 
- 예) sort=created,desc&sort=title (asc가 기본값) 



```java
@GetMapping("/posts")
    public Page<Post> getPosts(Pageable pageable){
        return posts.findAll(pageable);
    }

mockMvc.perform(get("/posts")
                .param("page", "0")
                .param("size", "10")
                .param("sort", "created,desc")
                .param("sort", "title")
        )
                .andDo(print())
                .andExpect(status().isOk());
//                .andExpect(jsonPath("$.content[0].title", is("jpa" )));
```



# 29. 스프링 데이터 Common: Web 4부: HATEOAS 

Page를 PagedResource로 변환하기 

-   일단 HATEOAS 의존성 추가 (starter-hateoas) 
-   핸들러 매개변수로 PagedResourcesAssembler 매개변수로 줘서 쓸 수 있다.  

사용하기 위해서는 스프링 부트 의존성 먼저 추가해야된다.

```java
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-hateoas</artifactId>
        </dependency>

```

![image-20190726140733753](http://ww2.sinaimg.cn/large/006tNc79gy1g5d5t3xy10j30m70j3gn4.jpg)

리소스로 변환한 후

![image-20190726140744947](http://ww4.sinaimg.cn/large/006tNc79gy1g5d5tbdxfbj30mj0lvgnm.jpg)

---

# 30. 스프링 데이터 Common: 마무리 

지금까지 살펴본 내용 

- 스프링 데이터 Repository 
- 쿼리메소드 
  - 메소드 이름 보고 만들기 
  - 메소드이름보고찾기 
- Repository 정의하기 
  - 내가 쓰고 싶은 메소드만 골라서 만들기 
  - Null 처리 
- 쿼리 메소드 정의하는 방법 
- 리포지토리 커스터마이징 
  - 리포지토리 하나 커스터마이징 
  - 모든 리포지토리의 베이스 커스터마이징 
- 도메인 이벤트 Publish 

- 스프링 데이터 확장 기능
  - QueryDSL 연동 
  - 웹지원

---

# 31. 스프링 스프링 데이터 JPA: JPA Repository

**@EnableJpaRepositories**

- 스프링부트사용할때는사용하지않아도자동설정됨.
- 스프링 부트 사용하지 않을 때는 @Configuration과 같이 사용. 

**@Repository 애노테이션을 붙여야 하나 말아야 하나...** 

- 안붙여도 됩니다. 만약 `extend JpaRepository` Samplerepository
- 이미 붙어 있어요. 또 붙인다고 별일이 생기는건 아니지만 중복일 뿐입니다. 

**스프링 @Repository**

- SQLExcpetion 또는 JPA 관련 예외를 스프링의 DataAccessException으로 변환 

해준다.


만약  `@GeneratedValue(strategy = GenerationType.IDENTITY)` 가 없다면? 

다음과 같은 에러가 발생한다.

`Caused by: org.hibernate.id.IdentifierGenerationException: ids for this class must be manually assigned before calling save(): co… `  

`org.springframework.orm.jpa.JpaSystemException: ids for this...`

  

# 32. 스프링 데이터 JPA: 엔티티 저장하기

JpaRepository의 save()는 단순히 새 엔티티를 추가하는 메소드가 아닙니다. 

- Transient 상태의 객체라면 EntityManager.persist() 
- Detached 상태의 객체라면 EntityManager.merge()   


![image-20190726144152374](http://ww2.sinaimg.cn/large/006tNc79gy1g5d6su99hpj30fi0du75j.jpg)

Transient인지 Detached 인지 어떻게 판단 하는가? 

- 엔티티의 @Id 프로퍼티를 찾는다. 해당 프로퍼티가 null이면 Transient 상태로 판단하고 id가 null이 아니면 Detached 상태로 판단한다. 
- 엔티티가 Persistable 인터페이스를 구현하고 있다면 isNew() 메소드에 위임한다. 
- JpaRepositoryFactory를 상속받는 클래스를 만들고 getEntityInfomration()을 오버라이딩해서 자신이 원하는 판단 로직을 구현할 수도 있습니다. 



**EntityManager.persist()**  

- https://docs.oracle.com/javaee/6/api/javax/persistence/EntityManager.html#persist(java.lang.Object) 

- Persist() 메소드에 넘긴 그 엔티티 객체를 Persistent 상태로 변경합니다. 

  *Transient란? 새로 만든 것이여서 ID를 몰라 아무것도 모르는것.*

  *Persistent란? 데이터가 삽입된적 있는 객체*

  

![image-20190726144305582](http://ww4.sinaimg.cn/large/006tNc79gy1g5d6u5bzoyj308r05mjrv.jpg)

**EntityManager.merge()** 

- https://docs.oracle.com/javaee/6/api/javax/persistence/EntityManager.html#merge(java.lang.Object) 

- Merge() 메소드에 넘긴 그 엔티티의 복사본을 만들고, 그 복사본을 다시 Persistent 상태로 변경하고 그 복사본을 반환합니다. 

  *Detached란? Properties 에 객체가 있으면*



![image-20190726144526598](http://ww4.sinaimg.cn/large/006tNc79gy1g5d6wlv870j30a406z0tc.jpg)

***무조건 리턴받은 인스턴스를 사용하는 것을 지향하라!***  

![image-20190726145346348](http://ww3.sinaimg.cn/large/006tNc79gy1g5d757ekqxj30eq0g7acb.jpg)



# 33. 스프링 데이터 JPA: 쿼리 메소드

![image-20190726145814690](http://ww2.sinaimg.cn/large/006tNc79gy1g5d79v1nr0j30m20alab6.jpg)

쿼리 찾아쓰기 

- 엔티티에 정의한 쿼리 찾아 사용하기 JPA Named 쿼리 

  - @NamedQuery 
  - @NamedNativeQuery 

- 리포지토리 메소드에 정의한 쿼리 사용하기 
  - @Query
  - @Query(nativeQuery=true) 



# 34. 스프링 데이터 JPA: 쿼리 메소드 Sort

이전과 마찬가지로 Pageable이나 Sort를 매개변수로 사용할 수 있는데, @Query와 같이 사용할 때 제약 사항이 하나 있습니다.  

Order by 절에서 함수를 호출하는 경우에는 Sort를 사용하지 못합니다. 그 경우에는 JpaSort.unsafe()를 사용 해야 합니다.
- Sort는 그 안에서 사용한 **프로퍼티** 또는 **alias**가 엔티티에 없는 경우에는 예외가 발생합니다.
- JpaSort.unsafe()를 사용하면 함수 호출을 할 수 있습니다. ○ JpaSort.unsafe(“LENGTH(firstname)”);



```java
@Query("SELECT p FROM  Post AS p WHERE p.title = ?1")
    List<Post> findByTitle(String title, Sort sort);

---
@Test
public void findByTitle(){
        savePost();
        List<Post> all = postRepository.findByTitle("Spring", Sort.by("title")); //Propertiry, 아닌것.
        assertThat(all.size()).isEqualTo(1);
    }
```



# 35. 스프링 데이터 JPA: Named Parameter과 SpEL 

**Named Parameter** 

- @Query에서 참조하는 매개변수를 ?1, ?2 이렇게 채번으로 참조하는게 아니라 이름으로 :title 이렇게 참조하는 방법은 다음과 같습니다. 
```java
@Query("SELECT p FROM Post AS p WHERE p.title = :title") 
List<Post> findByTitle(@Param("title") String title, Sort sort);
```

**SpEL** 

- 스프링 표현 언어
- https://docs.spring.io/spring/docs/current/spring-framework-reference/core.html#expressions 
- @Query에서 엔티티 이름을 #{#entityName} 으로 표현할 수 있습니다. 

```java
@Query("SELECT p FROM ​#{#entityName}​ AS p WHERE p.title = :title") List<Post> findByTitle(@Param("title") String title, Sort sort);
```

# 36. 스프링 데이터 JPA: Update 쿼리 메소드 

쿼리 생성하기 

- Find...
- Count...
- Delete….
- 흠…update는 어떻게하지?

Update 또는 Delete 쿼리 직접 정의하기

- @Modifying @Query
- 추천 ㄴㄴ

```java
@Modifying(clearAutomatically = true, flushAutomatically = true)
@Query("UPDATE Post p SET p.title = ?2 WHERE p.id = ?1")
int updateTitle(Long id, String title);
---
@Test
public void updateTitle(){
        Post spring = savePost();
        int update = postRepository.updateTitle("Hibernate",spring.getId());
        assertThat(update).isEqualTo(1);

        Optional<Post> byId = postRepository.findById(spring.getId());
//        여기서 실패가 될 것이다. 왜냐하면, Update 이 후 셀렉을 안할 것이다.
//        왜냐하면 Persistent가 비워지지 않았기 때문에, 캐쉬된 정보를 가지고 있을 것이다. 그러므로 맨 위에 clearAutomatically 를 True로 해주면 캐쉬된 정보가 update 쿼리 이후에 제거 될 것이다. 
        assertThat(byId.get().getTitle()).isEqualTo("Hibernate");
  
  Post spring = savePost();
spring.setTitle("hibernate")
  
 postRepository.findAll();
// 요렇게 되면 안전할 것이다. 위에 처럼 update 쿼리를 날리는 행위는 위험하다.
    }
```

# 37. 스프링 데이터 JPA: EntityGraph

쿼리 메소드 마다 연관 관계의 Fetch 모드를 설정 할 수 있습니다. 

**@NamedEntityGraph** 

- @Entity에서 재사용할 여러 엔티티 그룹을 정의할 때 사용.

**@EntityGraph**

- @NamedEntityGraph에 정의되어 있는 엔티티 그룹을 사용 함.
- 그래프타입설정가능
  - (기본값) FETCH: 설정한 엔티티 애트리뷰트는 EAGER 패치 나머지는 LAZY
    패치.
  - LOAD: 설정한 엔티티 애트리뷰트는 EAGER 패치 나머지는 기본 패치 전략
    따름.

```java
@NamedEntityGraph(name = "Comment.post", attributeNodes = @NamedAttributeNode("post"))
@Entity
public class Commnet {

    @Id
    @GeneratedValue
    private Long id;

    private String comment;

    @ManyToOne(fetch = FetchType.LAZY)
    private Post post;

    public Long getId() {
        return id;
    }
```

```java
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.orm.jpa.DataJpaTest;
import org.springframework.test.context.junit4.SpringRunner;

@RunWith(SpringRunner.class)
@DataJpaTest
public class CommnetRepositoryTest {

    @Autowired
    CommnetRepository commnetRepository;

    @Test
    public void getComment(){
        commnetRepository.getById(1l); //엔티티 쓴거
        /**
         *    select
         *         commnet0_.id as id1_0_0_,
         *         post1_.id as id1_1_1_,
         *         commnet0_.comment as comment2_0_0_,
         *         commnet0_.post_id as post_id3_0_0_,
         *         post1_.created as created2_1_1_,
         *         post1_.title as title3_1_1_
         *     from
         *         commnet commnet0_
         *     left outer join
         *         post post1_
         *             on commnet0_.post_id=post1_.id
         *     where
         *         commnet0_.id=?
         */
        System.out.println("------------------");
        commnetRepository.findById(1l); //엔티티 안쓴거
        /**
         * Hibernate:
         *     select
         *         commnet0_.id as id1_0_0_,
         *         commnet0_.comment as comment2_0_0_,
         *         commnet0_.post_id as post_id3_0_0_
         *     from
         *         commnet commnet0_
         *     where
         *         commnet0_.id=?
         */
    }
}
```

```java
@NamedEntityGraph(name = "Comment.post", attributeNodes = @NamedAttributeNode("post"))
@Entity
public class Commnet {

    @Id
    @GeneratedValue
    private Long id;

    private String comment;

    @ManyToOne(fetch = FetchType.LAZY)
    private Post post;

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

```

```java
public interface CommnetRepository extends JpaRepository<Commnet, Long> {

    @EntityGraph(value = "Comment.post")
    Optional<Commnet> getById(Long id);
}
```



각각의 다른 Fetch전략을 정해서 가져올수 있다.



# 38. 스프링 데이터 JPA: Projection

엔티티의 일부 데이터만 가져오기.

**인터페이스 기반** 프로젝션

- Nested 프로젝션 가능.
- **Closed 프로젝션()**
	- 쿼리를 최적화 할 수 있다. 가져오려는 애트리뷰트가 뭔지 알고 있으니까.
	- Java8의디폴트메소드를사용해서연산을할수있다.
- Open 프로젝션 (다 가져와서 보고싶은 것만 보는 행위)
	- @Value(SpEL)을 사용해서 연산을 할 수 있다. 스프링 빈의 메소드도 호출 가능.
	- 쿼리 최적화를 할 수 없다. SpEL을 엔티티 대상으로 사용하기 때문에.

**클래스 기반 프로젝션** 

- DTO
- 롬복 @Value로 코드 줄일 수 있음

**다이나믹 프로젝션**

- 프로젝션 용 메소드 하나만 정의하고 실제 프로젝션 타입은 타입 인자로 전달하기.

`<T> List<T> findByPost_Id(Long id, Class<T> type);`

```java
package com.example.commonweb.post;

public interface CommentSummary {

    String getComment();
    int getUp();
    int getDown();
  
  // 이렇게 하는게 더 직관적으로 보임.
    default String getVotes(){
        return getUp() + " " + getDown();
    }

//    @Value("#{target.up + ' ' + target.down}")
//    String getVotes();
}
--------
import org.springframework.data.jpa.repository.EntityGraph;
import org.springframework.data.jpa.repository.JpaRepository;

import java.util.List;
import java.util.Optional;

public interface CommnetRepository extends JpaRepository<Commnet, Long> {

  List<CommentSummary> findAllByPost_id(Long id);
      /**
       *  select
       *         commnet0_.comment as col_0_0_,
       *         commnet0_.up as col_1_0_,
       *         commnet0_.down as col_2_0_ 
       */
```



# 39. 스프링 데이터 JPA: Specifications

에릭 에반스의 책 DDD에서 언급하는 Specification 개념을 차용 한 것으로 QueryDSL의 

Predicate와 비슷합니다. 

**설정 하는 방법**

- https://docs.jboss.org/hibernate/stable/jpamodelgen/reference/en-US/html_single/ 
- 의존성 설정
- 플러그인 설정
- IDE에 에노테이션 처리기 설정
- 코딩 시작

```xml
        <dependency>
            <groupId>org.hibernate</groupId>
            <artifactId>hibernate-jpamodelgen</artifactId>
        </dependency>
```

```xml
            <plugin>
                <groupId>org.bsc.maven</groupId>
                <artifactId>maven-processor-plugin</artifactId>
                <version>2.0.5</version>
                <executions>
                    <execution>
                        <id>process</id>
                        <goals>
                            <goal>process</goal>
                        </goals>
                        <phase>generate-sources</phase>
                        <configuration>
                            <processors>
                                <processor>org.hibernate.jpamodelgen.JPAMetaModelEntityProcessor</processor>
                            </processors>
                        </configuration>
                    </execution>
                </executions>
                <dependencies>
                    <dependency>
                        <groupId>org.hibernate</groupId>
                        <artifactId>hibernate-jpamodelgen</artifactId>
                        <version>${hibernate.version}</version>
                    </dependency>
                </dependencies>
            </plugin>
```

그리고 인텔리제이에서 `org.hibernate.jpamodelgen.JPAMetaModelEntityProcessor` 넣기.

![image-20190727020001695](http://ww1.sinaimg.cn/large/006tNc79gy1g5dqej3pyfj31450u0qe4.jpg)





위에서 설정하라는 것을 다한 뒤에 

![image-20190727022203088](http://ww3.sinaimg.cn/large/006tNc79gy1g5dr1g0r3wj30jk04wtdb.jpg)



여기서 빌드를 하고 나면, 

![image-20190727022233404](http://ww2.sinaimg.cn/large/006tNc79gy1g5dr1yf2m8j30pk0e6gml.jpg)



이렇게  `Comment_` `Post_` 가 생겨난 것을 알 수 있다. 그러면 이렇게 생긴거를 바탕으로 Spec을 만들어 주면 된다.

```java
import org.springframework.data.jpa.domain.Specification;

import javax.persistence.criteria.CriteriaBuilder;
import javax.persistence.criteria.CriteriaQuery;
import javax.persistence.criteria.Predicate;
import javax.persistence.criteria.Root;

public class CommentSpecs {
    /**
     *    select
     *         commnet0_.id as id1_0_,
     *         commnet0_.best as best2_0_,
     *         commnet0_.comment as comment3_0_,
     *         commnet0_.down as down4_0_,
     *         commnet0_.post_id as post_id6_0_,
     *         commnet0_.up as up5_0_
     *     from
     *         commnet commnet0_
     *     where
     *         commnet0_.best=1
     */
    public static Specification<Commnet> isBest(){
        return new Specification<Commnet>() {
            @Override
            public Predicate toPredicate(Root<Commnet> root,
                                         CriteriaQuery<?> query,
                                         CriteriaBuilder criteriaBuilder) {
                return criteriaBuilder.isTrue(root.get(Commnet_.best));
            }
        };
    }

    public static Specification<Commnet> isGood(){
        return new Specification<Commnet>() {
            @Override
            public Predicate toPredicate(Root<Commnet> root,
                                         CriteriaQuery<?> query,
                                         CriteriaBuilder criteriaBuilder) {
                return criteriaBuilder.greaterThan(root.get(Commnet_.up), 10);
            }
        };
    }
}

```

위와 같이 설정 한뒤에 Test에 가서 이렇게 하면 된다.

![image-20190727022455255](http://ww1.sinaimg.cn/large/006tNc79gy1g5dr4cvghqj31cc0gadhz.jpg)



# 40. 스프링 데이터 JPA: Query by Example

**QBE는 필드 이름을 작성할 필요 없이*(*뻥*)*** 단순한 인터페이스를 통해 동적으로 쿼리를 만드는
기능을 제공하는 사용자 친화적인 쿼리 기술입니다. (감이 1도 안잡히는거 이해합니다.. 코드를 봐야
이해하실꺼에요.)



Example = Probe + ExampleMatcher

- Probe는 필드에 어떤 값들을 가지고 있는 도메인 객체. 
- ExampleMatcher는 Prove에 들어있는 그 필드의 값들을 어떻게 쿼리할 데이터와 비교할지 정의한 것. 
- Example은 그 둘을 하나로 합친 것. 이걸로 쿼리를 함. 



장점

- 별다른 코드 생성기나 애노테이션 처리기 필요 없음
- 도메인 객체 리팩토링 해도 기존 쿼리가 깨질 걱정하지 않아도 됨. (뻥)

- 데이터 기술에 독립적인 API

단점

- **nested 또는 프로퍼티 그룹 제약 조건을 못 만든다.**
- **조건이 제한적이다.** 문자열은 starts/contains/ends/regex가 가능하고 그 밖에 propery는 값이 정확히 일치해야 한다.

QueryByExampleExecutor

https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#query-by-example



*예제 객체로 Example을 만드는 것.*



**그래서 판단하기로 유연하고 테스트하기 좋은건 QueryDSL, Specifications 가 좋았다.**



# 41. 스프링 데이터 JPA: 트랜잭션

스프링 데이터 JPA가 제공하는 Repository의 모든 메소드에는 기본적으로 @Transaction이 

적용되어 있습니다. 



**스프링 @Transactional**

- 클래스, 인터페이스, 메소드에 사용할 수 있으며, 메소드에 가장 가까운 애노테이션이 

  우선 순위가 높다. 

- https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/transaction/annotation/Transactional.html (반드시 읽어볼 것, 그래야 뭘 설정해서 쓸 수 있는지 알죠..) 



**JPA 구현체로 Hibernate를 사용할 때 트랜잭션을 readOnly를 사용하면 좋은 점** 

- Flush 모드를 NEVER로 설정하여, Dirty checking을 하지 않도록 한다.

*Flush란? 데이터베이스를 싱크하는 것.*(ex, Commit, read하기 전에)- 데이터 변경이 일어나지 않을거야!  
*Dirty checking?*

데이터의 변경이 없다면 ReadOnly를 True로!



**충분한 학습이 필요함!**



# 42. 스프링 데이터 JPA: Auditing

Entity가 변화가 됨을 감지하여 언제, 누구에 의해 발생했는지를 기록하는 것

스프링 데이터 JPA의 Auditing

```java
@CreatedDate
private Date created;

@LastModifiedDate 
private Date updated;

@CreatedBy
@ManyToOne
private Account createdBy;

@LastModifiedBy
@ManyToOne
private Account updatedBy;
```

***엔티티의 변경 시점에 언제, 누가 변경했는지에 대한 정보를 기록하는 기능.***

아쉽지만 이 기능은 스프링 부트가 자동 설정 해주지 않습니다. 

1. 메인 애플리케이션 위에 @EnableJpaAuditing 추가 
2. 엔티티 클래스 위에 @EntityListeners(AuditingEntityListener.class) 추가 
3. AuditorAware 구현체 만들기 
4. @EnableJpaAuditing에 AuditorAware 빈 이름 설정하기. 

## JPA의 라이프 사이클 이벤트

- https://docs.jboss.org/hibernate/orm/4.0/hem/en-US/html/listeners.html
- @PrePersist
-  @PreUpdate
-  ...



---

```java
@Enumerated(value= EnumType.STRING)
private CommentStatus commentStatus;
```