



# 1부: 핵심 개념 이해

본격적인 스프링 데이터 JPA 활용법을 학습하기에 앞서, ORM과 JPA에 대한 이론적인 배경을 학습합니다. 

**관계형 데이터베이스와 자바**
JDBC(관계형) 데이터베이스와 자바의 연결 고리![img](http://ww4.sinaimg.cn/large/006tNc79gy1g4wx3sxk29j30lb06vt9e.jpg)JDBC

- DataSource / DriverManager
- Connection
- PreparedStatement

SQL

- DDL - 스키마 생성
- DML - 데이터 조작

RAW한 SQL를 활용하는 것이 무엇이 문제인가?

- SQL을 실행하는 비용이 비싸다.
- SQL이 데이터베이스 마다 다르다.
- 스키마를 바꿨더니 코드가 너무 많이 바뀌네...반복적인 코드가 너무 많아.
- 당장은 필요가 없는데 언제 쓸 줄 모르니까 미리 다 읽어와야 하나...  
  **: 성능최적화적인 코드를 구현하기 힘듬**

의존성 추가

```xml
<dependency> 
	<groupId>org.postgresql</groupId>   <artifactId>postgresql</artifactId>
</dependency>
```

PostgreSQL 설치 및 서버 실행 (docker)

```
docker run -p 5432:5432 -e POSTGRES_PASSWORD=pass -e POSTGRES_USER=keesun -e POSTGRES_DB=springdata --name postgres_boot -d postgres

docker exec -i -t postgres_boot bash

su - postgres

psql springdata

데이터베이스 조회
list

테이블 조회
dt

쿼리
SELECT * FROM account;

```



# 04. ORM: Object-Relation Mapping

```java
///JDBC 사용
String url = “jdbc:postresql://...”
String username = “”;
String password = “pass”

//Java 7부터 도입
try(Connection connection = DriverManager.getConnection(url, username, password)) {
            System.out.println("Connection created: " + connection);
            String sql = "INSERT INTO ACCOUNT VALUES(1, 'keesun', 'pass');";
            try(PreparedStatement statement = connection.prepareStatement(sql)) {
                statement.execute();
            }
        }
```

**도메인 모델 사용**

```java
Account account = new Account(“keesun”, “pass”);
accountRepository.save(account);
```

**JDBC 대신 도메인 모델 사용하려는 이유는?**

- 객체 지향 프로그래밍의 장점을 활용하기 좋으니까
- 각종 디자인 패턴
- 코드 재사용
- 비즈니스 로직 구현 및 테스트 편함.



 ***ORM***은 애플리케이션의 클래스와 SQL 데이터베이스의 테이블 사이의 ***맵핑 정보를 기술한 메타데이터***를 사용하여, 자바 애플리케이션의 객체를 SQL 데이터베이스의 테이블에 ***자동으로 (또 깨끗하게) 영속화*** 해주는 기술입니다.

In a nutshell, object/relational mapping is the automated (and **transparent**) persistence of objects in a Java application to the tables in an SQL database, using metadata that describes the mapping between the classes of the application and the schema of the SQL database.

- Java Persistence with Hibernate, Second Edition

|                         장점                          |   단점   |
| :---------------------------------------------------: | :------: |
| 생산성<br />유지보수<br />성능<br />밴더 독립성<br /> | 학습비용 |



JPA와 하이버네이트는 중간에 캐시가 존재해서 변경해야되는 사항에서만 동작된다.

Ex) Insert A, Insert A, Insert B -> 일반적으로 3번의 쿼리를 보내나, 캐시를 통해서 단 한번에 해결한다.

***하이버네이트가 자동으로 SQL를 생성해주기 때문에 SQL를 잘 알아야 한다.***

성능문제에 부딪힐 경우에 문제는 하이버네이트가 문제가 아니라, 개발자 본인이 문제다.


# 5. ORM: 패러다임 불일치

객체를 릴레이션에 맵핑하려니 발생하는 문제들과 해결책

**밀도(Granularity) 문제**

| 객체 |릴레이션 |
| ---- | ---- |
|  다양한 크기의 객체를 만들 수 있음.   <br />커스텀한 타입 만들기 쉬움.  |  테이블  기본 데이터 타입 (UDT는 비추) |

**서브타입(Subtype) 문제 |** 

| 객체                                 | 릴레이션                                                     |
| ------------------------------------ | ------------------------------------------------------------ |
| 상속 구조 만들기 쉬움.  <br/>다형성. | 테이블 상속이라는게 없음.  <br/>상속 기능을 구현했다 하더라도 표준 기술이 아님.  <br/>다형적인 관계를 표현할 방법이 없음. |

**식별성(Identity) 문제**

| 객체                                                         | 릴레이션           |
| ------------------------------------------------------------ | ------------------ |
| 레퍼런스 동일성 (==)  <br />인스턴스 동일성 (equals() 메소드) | 주키 (primary key) |

**관계(Association) 문제**

| 객체                                                         | 릴레이션                                                     |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| 객체 레퍼런스로 관계 표현.  <br />근본적으로 ‘방향'이 존재 한다.  <br />다대다 관계를 가질 수 있음 | 외래키(foreign key)로 관계 표현.  <br />‘방향'이라는 의미가 없음.  그냥 Join으로 아무거나 묶을 수 있음.  <br />태생적으로 다대다 관계를 못만들고, 조인 테이블 또는 링크 테이블을 사용해서 두개의 1대다 관계로 풀어야 함. |

데이터 네비게이션(Navigation)의 문제

| 객체                                                         | 릴레이션                                                     |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| 레퍼런스를 이용해서 다른 객체로 이동 가능.  <br />콜렉션을 순회할 수도 있음. | 하지만 그런 방식은 릴레이션에서 데이터를 조회하는데 있어서 매우 비효율적이다.<br/>데이터베이스에 요청을 적게 할 수록 성능이 좋다. 따라서 Join을 쓴다.<br/>하지만, 너무 많이 한번에 가져오려고 해도 문제다.<br/>그렇다고 lazy loading을 하자니 그것도 문제다. (n+1 select) |

---

# JPA 프로그래밍: 프로젝트 세팅

# 데이터베이스 실행 

-  PostgreSQL 도커 컨테이너 재사용 
- docker start postgres_boot 

스프링 부트 

- 스프링 부트 v2.* 
- 스프링 프레임워크 v5.* 

스프링 부트 스타터 JPA 

- JPA 프로그래밍에 필요한 의존성 추가 

  - ○  JPA v2.* 
  - ○  Hibernate v5.* 

- 자동 설정: HibernateJpaAutoConfiguration 

  - ○  컨테이너가 관리하는 EntityManager (프록시) 빈 설정 

  application.property 가 아래 entitiyManagerFactoryBuilder를 타고 환경설정을 가짐.
  ![img](http://ww4.sinaimg.cn/large/006tNc79gy1g52roe5u81j30mb08rabr.jpg)

  - PlatformTransactionManager 빈 설정 

    JDBC 설정
     jdbc:postgresql://localhost:5432/springdata 



 keesun / pass 

application.properties

> spring.jpa.properties.hibernate.jdbc.lob.non.contextual_creation=true spring.jpa.hibernate.ddl-auto=create 



# 7. 엔티티 맵핑

XML로 맵핑하는 건 거의 보지 못함,.


## @Entity 

-  “엔티티”는 객체 세상에서 부르는 이름. 

- 보통 클래스와 같은 이름을 사용하기 때문에 값을 변경하지 않음. 

- 엔티티의 이름은 JQL에서 쓰임. 

  

> 엔티티 키워드는 다른 필드가 예약키워드로 잡혀 Table 네임이 중복되서 설정 불가능

## @Table 

-  “릴레이션" 세상에서 부르는 이름. 
- @Entity의 이름이 기본값. 
- 테이블의 이름은 SQL에서 쓰임. 



## @Id 

- 엔티티의 주키를 맵핑할 때 사용.   

- 자바의 모든 primitive 타입과 그 랩퍼 타입을 사용할 수 있음. 
  : 그러나 래퍼타입을 권장함, 왜냐하면 Null이니까? Long을 활용함. 

  ○ Date랑 BigDecimal, BigInteger도 사용 가능.

- 복합키를 만드는 맵핑하는 방법도 있지만 그건 논외로.. 



## @GeneratedValue 

- 주키의 생성 방법을 맵핑하는 애노테이션 
   - 생성 전략과 생성기를 설정할 수 있다. 
기본 전략은 AUTO: 사용하는 DB에 따라 적절한 전략 선택
   - TABLE, SEQUENCE, IDENTITY 중 하나.



## @Column

- unique 

- nullable
- length
- columnDefinition
-  ... 



## @Temporal

- 현재 JPA 2.1까지는 Date와 Calendar만 지원.

## @Transient

- 컬럼으로 맵핑하고 싶지 않은 멤버 변수에 사용.

**application.properties**

SQL 을 런타임에서 볼 수 있음.

- spring.jpa.show-sql=true
- spring.jpa.properties.hibernate.format_sql=true

## @Enum

- EnumType=String 으로 할 것!



# 8. Value 타입 맵핑

**엔티티 타입과 Value 타입 구분**  

- 식별자가 있어야 하는가.
- 독립적으로 존재해야 하는가. 

**Value 타입 종류**  

- 기본 타입 (String, Date, Boolean, ...)  
- Composite Value 타입
- Collection Value 타입
	- 기본 타입의 콜렉션
	- 컴포짓 타입의 콜렉션

![image-20190717145703244](http://ww1.sinaimg.cn/large/006tNc79gy1g52snu1srfj30ks04p0t0.jpg)

![image-20190717145716829](http://ww2.sinaimg.cn/large/006tNc79gy1g52so2x9scj306v09c0ta.jpg)

**Composite Value 타입 맵핑**  

- @Embeddable
- @Embedded
- @AttributeOverrides
- @AttributeOverride



## 9. 1대다 맵핑
**관계에는 항상 두 엔티티가 존재 합니다.**

- 둘 중 하나는 그 관계의 주인(owning)이고
- 다른 쪽은 종속된(non-owning) 쪽입니다.
- 해당 관계의 반대쪽 레퍼런스를 가지고 있는 쪽이 주인.
  

**단방향에서의 관계의 주인은 명확합니다.**

- 관계를 정의한 쪽이 그 관계의 주인입니다.
  

**단방향 @ManyToOne**

- 기본값은 FK 생성
  

**단방향 @OneToMany**

- 기본값은 조인 테이블 생성
  

**양방향**

- FK 가지고 있는 쪽이 오너 따라서 기본값은 @ManyToOne 가지고 있는 쪽이 주인.
- 주인이 아닌쪽(@OneToMany쪽)에서 mappedBy 사용해서 관계를 맺고 있는 필드를
  설정해야 합니다.


**양방향**

- @ManyToOne (이쪽이 주인)
- @OneToMany(mappedBy)
  위와 같이 설정할 경우 두 부분 모두 단방향으로 2개 만들어짐. 그러므로, 관계를 적어놓아야 함.
  Ex) @OneToMany(mappedBy = "owner")
  ![image-20190717153315724](http://ww1.sinaimg.cn/large/006tNc79gy1g52tpix4juj30am039glt.jpg)
- 주인한테 관계를 설정해야 DB에 반영이 됩니다.



# 10. Cascade

**엔티티 상태 변화**를 전파 시키는 옵션.
잠깐?  **엔티티 상태**가 뭐지?

- Transient: JPA가 모르는 상태

- Persistent: JPA가 관리중인 상태 (1차 캐시, Dirty Checking, Write Behind, …)  
  `Session.save() 호출되는 부분 `
  `1차 캐시란? PersistentContext에 인덱스를 넣는 그 상태. `  
  ![](http://ww1.sinaimg.cn/large/006tNc79gy1g52u595jwej30eb05faag.jpg)

  이 때 load부분에 `SELECT` 쿼리를 사용하지 않는다. 즉, 캐시가 되어있다는 의미.



- Detached: JPA가 더이상 관리하지 않는 상태.  
  : 트랜잭션이 끝나고 세션 밖으로 나올 때.
-  Removed: JPA가 관리하긴 하지만 삭제하기로 한 상태.



![image-20190717154418694](http://ww2.sinaimg.cn/large/006tNc79gy1g52u10fperj30ff09q40q.jpg)



`Post` / `Comment` 관계를 양방향으로 설정 후 OneToMany에 cascade 설정을 함.

![image-20190717161437732](http://ww1.sinaimg.cn/large/006tNc79gy1g52uwjp3isj30ew02kt91.jpg)



# 11. Fetch

연관 관계의 엔티티를 어떻게 가져올 것이냐... 지금 (Eager)? 나중에(Lazy)?

- @OneToMany의 기본값은 Lazy
- @ManyToOne의 기본값은 Eager

# 12. Query

###  JPQL (HQL) 

1. - Java Persistence Query Language / Hibernate Query Language 
   - 데이터베이스 테이블이 아닌, 엔티티 객체 모델 기반으로 쿼리 작성. 
   - JPA 또는 하이버네이트가 해당 쿼리를 SQL로 변환해서 실행함. 
   - https://docs.jboss.org/hibernate/orm/5.2/userguide/html_single/Hibernate_User_Guide.html#hql 

```java
TypedQuery<Post> query = entityManager.createQuery("SELECT p FROM Post As p", Post.class);
        List<Post> posts = query.getResultList();
```

단점은, 타입세이프하지 않다라는 점.



### Criteria

- 타입 세이프 쿼리
- https://docs.jboss.org/hibernate/orm/5.2/userguide/html_single/Hibernate_User_Guide.html#criteria 

```java
CriteriaBuilder builder = entityManager.​getCriteriaBuilder​();
 
CriteriaQuery<Post> criteria = builder.createQuery(Post.class); Root<Post> root = criteria.from(Post.class);
criteria.select(root);
List<Post> posts = entityManager.​createQuery​(criteria).getResultList();
```



### Native Query

- SQL 쿼리 실행하기 
- https://docs.jboss.org/hibernate/orm/5.2/userguide/html_single/Hibernate_User_Guide.html#sql 







