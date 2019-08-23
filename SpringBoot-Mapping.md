# SpringBoot 에서 매핑 구현을 살펴보자.



## 00.엔티티와 밸류 기본 매핑 구현



- 애그리거트 루트는 엔티티이므로 @Entitiy 로 매핑 설정
- 한 테이블에 엔티티와 벨류 데이터가 같이 있다면,
  - 밸류는 @Embeddable로 매핑 설정
  - 밸류 타입 프로퍼티는 @Embedded로 매핑 설정





## 01. AttributeConverter를 이용한 밸류 매핑 처리

int, long, String, LocalDate와 같은 타입은 DB 테이블의 한 개 컬럼과 매핑. 이와 비슷하게 밸류 타입의 프로퍼티를 한 개 컬럼에 매핑해야 할 때는?

```java
import com.myshop.common.model.Money;

import javax.persistence.AttributeConverter;
import javax.persistence.Converter;

@Converter(autoApply = true)
public class MoneyConverter implements AttributeConverter<Money, Integer> {

    @Override
    public Integer convertToDatabaseColumn(Money money) {
        if (money == null)
            return null;
        else
            return money.getValue();
    }

    @Override
    public Money convertToEntityAttribute(Integer value) {
        if (value == null) return null;
        else return new Money(value);
    }
}
```

위 코드는 DB에 Money을 저장 될 때는 Integer로 들어가고, DB에서 꺼낼때는 Money로 변환되서 들어온다.



해당 코드를 쓰는 코드는 다음과 같다.

```java
@Entity
@Table(name = "product")
public class Product {
    @EmbeddedId
    private ProductId id;

    @ElementCollection
    @CollectionTable(name = "product_category",
            joinColumns = @JoinColumn(name = "product_id"))
    private Set<CategoryId> categoryIds;

    private String name;

    @Convert(converter = MoneyConverter.class)
    private Money price;
    private String detail;

    @OneToMany(cascade = {CascadeType.PERSIST, CascadeType.REMOVE},
            orphanRemoval = true, fetch = FetchType.EAGER)
    @JoinColumn(name = "product_id")
  
  ... 생략
}
```



## 02. Value Collection - 별도 테이블 매핑

가정: Order 엔티티는 한 개 이상의 OrderLine을 가질 수 있다.

OrderLine의 순서가 있다면 다음과 같이 List 타입을 이용해서 OrderLine 타입의 컬렉션을 프로퍼티로 갖게 된다.

```java
public class Order {
  private List<OrderLine> orderLines;
  ...
}
```

Value 타입의 컬렉션은 별도 테이블에 보관.

Order와 OrderLine을 위한 테이블은 다음과 같이 매칭.

![image-20190816231310623](http://ww1.sinaimg.cn/large/006tNc79gy1g61vlgg8osj31770u0kjq.jpg)



Value 컬렉션을 저장하는 ORDER_LINE는 Index가 필요할 것이다. 그것을 line_idx라고 가정하자.



밸류 컬렉션을 별도 테이블로 매핑할 때는 @ElementCollection 과  @CollectionTable을 함께 사용한다.

```java
import javax.persistence.*;
import java.util.Date;
import java.util.List;

@Entity
@Table(name = "purchase_order")
@Access(AccessType.FIELD)
public class Order {
  
 ...
   
    @ElementCollection
    @CollectionTable(name = "order_line", joinColumns = @JoinColumn(name = "order_number"))
    @OrderColumn(name = "line_idx")
    private List<OrderLine> orderLines;

    @Column(name = "total_amounts")
    private Money totalAmounts;
  
  ...
}
```



```java
@Embeddable
public class OrderLine {
    @Embedded
    private ProductId productId;

    @Column(name = "price")
    private Money price;

    @Column(name = "quantity")
    private int quantity;

    @Column(name = "amounts")
    private Money amounts;
...
}
```



OrderLine 의 매핑을 함께 표시했는데 OrderLine에는 List의 인덱스 값을 저장하기 위한 프로퍼티가 존재 X, 그 이유는 List타입 자체가 인덱스를 갖고 있기 때문. 위에서는 @OrderColumn 으로 인덱스 값을 저장했다.



`@CollectionTable` 은 밸류를 저장할 테이블을 지정할 때 사용, name 속성으로 테이블 이름을 지정하고 `joinColumns ` 속성은 외부키로 사용하는 칼럼을 지정한다.



위에서는 외부키가 한 개인데, 두 개 이상인 경우 @JoinColumn의 배열을 이용해 외부키 목록을 지정



## 03. 한 개 컬럼 매핑

Value Collection을 별도 테이블이 아닌 한 개 칼럼에 저장해야 할 때가 있다. 이 때는 위에서 설명한 AttributeConverter를 사용하자!



## 04. Value를 이용한 Id 매핑

식별자는 최종적으로 문자열이나 숫자와 같은 기본 타입이기 때문에 다음과 같이 String이나 Long 타입을 이용해서 식별자를 매핑한다.

```java
@Entity
public class Order {
  
    @Id
    private String number;
  
...
}

//

@Entity
public class Article {
  @Id
  private Long id;
  ...
}
```



기본 타입을 사용하는 것이 나쁘진 않지만 식별자라는 의미를 부각시키기 위해 식별자 자체를 별도 ValueType 으로 만들 수 있다.

ValueType을 식별자로 인식하기 위해서는 `@EmbeddedId` 어노테이션을 사용하기!

단, 주의 할 점이 식별자로 사용될 별류 타입은 `Serializable` 인터페이스를 상속받아야 한다.



*식별자를 ValueType로 할 경우 주어지는 장점은 식별자에 기능을 더할 수 있다는 것!*



## 05. 별도 테이블에 저장하는 Value Mapping



Tips) 애그리거트를 판단하는 방법으로 자신만의 독자적인 라이프사이클을 갖는다면 다른 애그리거트일 가능성이 높다.



우리가 흔히 저지르는 실수 중 하나가 바로 아래와 같은 맵핑관계를 갖는 것이다.

![image-20190816233906848](http://ww3.sinaimg.cn/large/006tNc79gy1g61wccsqvzj31l20u0e84.jpg)



ArticleContent의 ID가 Article의 식별자로 갖는 마치 1대1 맵핑 관계를 갖는 것처럼 보인다는 것이 문제다.



정확하게는, Article의 내용을 ArticleContent가 가지고 있는 것이 맞는 것이다.

그럼 다음과 같은 맵핑 관계가 나타날 것이다.

![image-20190816234120615](http://ww1.sinaimg.cn/large/006tNc79gy1g61werz1aoj32br0pjkjm.jpg)



`ArticleContent`는 `Value` 이므로 `@Embeddable` 로 매핑한다. `ArticleContent` 와 매핑되는 테이블은 `Article`과 매핑되는 테이블과 다른데, 이때 밸류를 매핑한 테이블을 지정하기 위해 `@SecondaryTable`과 `@AttributeOverride` 를 사용한다.



```java

@Entity
@Table(name = "article")
@SecondaryTable(
        name = "article_content",
        pkJoinColumns = @PrimaryKeyJoinColumn(name = "id")
)
public class Article {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String title;

    @AttributeOverrides({
            @AttributeOverride(name = "content",
                    column = @Column(table = "article_content")),
            @AttributeOverride(name = "contentType",
                    column = @Column(table = "article_content"))
    })
    @Embedded
    private ArticleContent content;

    protected Article() {
    }
...
}

```



`@SecondaryTable` 의 name 속성은 Value 를 저장할 테이블을 지정한다.

 `pkJoinColumns` 속성은 Value 테이블에서 Entity 테이블로 조인할 때 사용할 컬럼을 지정.

content 필드에 @AttributeOverride를 적용했는데 이 애노테이션을 사용해서 해당 밸류 데이터가 저장된 테이블 이름을 지정한다.



*@SecondaryTable을 이용하면 아래 코드를 실행할 때 두 테이블을 조인해서 데이터를 조회하게 된다.*

```java
//@SecondaryTable로 매핑된 article_content 테이블을 조인
Article article = entityManager.find(Article.class, 1L);
```





## 06. Value Collection을 @Entity로 매핑하기.

개념적으로 Value 인데 구현 기술의 한계나 팀 표준 때문에 @Entity 를 사용해야 할 때도 있다.

 예를 들어 제품의 이미지 업로드 방식에 따라 이미지 경로와 썸네일 이미지 제공 여부가 달라진다고 해보자. 이를 위해 Image는 다음과 같은 계층 구조로 설계할 수 있다.



![image-20190816235504913](http://ww4.sinaimg.cn/large/006tNc79gy1g61wt1xl9qj31io0u01l1.jpg)





JPA는 @Embeddable 타입의 클래스 상속 매핑을 지원하지 않는다. 따라서 상속 구조를 갖는 밸류 타입을 사용하려면 @Embeddable 대신 @Entity 를 이용한 상속 매핑으로 처리해야 한다. Value 타입을 @Entity 로 매핑하므로 식별자 매핑을 위한 필드도 추가해야 한다. 또한 구현 클래스를 구분하기 위한 타입 식별(discriminator) 칼럼을 추가해야 한다. 이를 위한 테이블 설계는 아래와 같다.



![telegram-cloud-photo-size-5-6249273657763408474-y](http://ww3.sinaimg.cn/large/006tNc79gy1g61wxc57hmj30zk0g3jt0.jpg)



한 테이블에 Image 및 하위 클래스를 매핑하므로 Image 클래스에 @Inheritance를 적용하고 strategy 값으로 SINGLE_TABLE을 사용하고 @DiscriminatorColumn을 이용해서 타입을 구분하는 용도로 사용할 칼럼을 지정한다. 

Image를 @Entity로 매핑했지만 모델에서 Image는 엔티티가 아니라 밸류이므로 상태를 변경하는 기능은 추가하지 않는다.



```java
@Entity
@Inheritance(strategy = InheritanceType.SINGLE_TABLE)
@DiscriminatorColumn(name = "image_type")
@Table(name = "image")
public abstract class Image {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "image_id")
    private Long id;

    @Column(name = "image_path")
    private String path;

    @Temporal(TemporalType.TIMESTAMP)
    @Column(name = "upload_time")
    private Date uploadTime;

    protected Image() {}
    public Image(String path) {
        this.path = path;
        this.uploadTime = new Date();
    }

    protected String getPath() {
        return path;
    }

    public Date getUploadTime() {
        return uploadTime;
    }

    public abstract String getUrl();
    public abstract boolean hasThumbnail();
    public abstract String getThumbnailUrl();

}
```



*Image를 상속받은 클래스는 다음과 같이 @Entity 와 @DisCriminator 를 사용해서 매핑한다!!!*



```java
@Entity
@DiscriminatorValue("II")
public class InternalImage extends Image {
    protected InternalImage() {}
    public InternalImage(String path) {
        super(path);
    }
  ...
}

---
@Entity
@DiscriminatorValue("EI")
public class ExternalImage extends Image {
    private ExternalImage() {}
    public ExternalImage(String path) {
        super(path);
    }
}
```

같이 설정한다.



Image가 @Entity이므로 목록을 담고 있는 Product는 @OneToMany를 이용해서 매핑을 처리.

Image는 밸류이므로 독자적인 라이프사이클을 갖지 않고 Product에 완전히 의존한다. 따라서 cascade 속성을 이용해서 Product를 저장할 때 함께 저장되고, Product를 삭제할 때 함께 삭제되도록 설정한다. 리스트에서 Image객체를 제거하면 DB에서 함께 삭제되도록 orphanRemoval 도 True로 설정한다.

```java
@Entity
@Table(name = "product")
public class Product {
    @EmbeddedId
    private ProductId id;

    @ElementCollection
    @CollectionTable(name = "product_category",
            joinColumns = @JoinColumn(name = "product_id"))
    private Set<CategoryId> categoryIds;

    private String name;

    @Convert(converter = MoneyConverter.class)
    private Money price;
    private String detail;
**********************************************************************
    @OneToMany(cascade = {CascadeType.PERSIST, CascadeType.REMOVE},
            orphanRemoval = true, fetch = FetchType.EAGER)
    @JoinColumn(name = "product_id")
    @OrderColumn(name = "list_idx")
    private List<Image> images = new ArrayList<>();
**********************************************************************
    protected Product() {
    }

    public Product(ProductId id, String name, Money price, String detail, List<Image> images) {
        this.id = id;
        this.name = name;
        this.price = price;
        this.detail = detail;
        this.images.addAll(images);
    }
**********************************************************************
  public void changeImages(List<Image> newImages) {
        images.clear();
        images.addAll(newImages);
    }
**********************************************************************
...
}
```



ChangeImages() 를 보면 clear 후 addAll 하고 있는 모습을 볼 수 있다. 여기서 만약 clear가 동작된다면 @OneToMany 매핑에서 네번의 쿼리가 동작한다. 이는 상속 때문에 발생하는 문제이다. 그러므로 상속을 제거하고 단일 클래스로 구현하는 것이 더 좋다고 판단된다.



## 07. ID참조와 조인 테이블을 이용한 단방향 M:N 매핑

애그리거트 간 집합 연산은 성능상의 이유로 피해야 한다고 했다. 그럼에도 불구하고 요구사항을 구현하는 데 집합 연관을 사용하는 것이 유리하다면 ID참조를 이용해 단방향 집합 연관을 적용해보자.

```java
@Entity
@Table(name = "product")
public class Product {
    @EmbeddedId
    private ProductId id;

    @ElementCollection
    @CollectionTable(name = "product_category",
            joinColumns = @JoinColumn(name = "product_id"))
    private Set<CategoryId> categoryIds;
		...

```

이 코드는 Product에서 Category 로의 단방향 M:N 연관을 ID 참조 방식으로 구현한 것.



ID참조를 이용한 애그리거트 간 단방향 M:N 연관은 Value Collection Mapping과 동일한 방식으로 설정한 것을 알 수 있다. 차이점이 있다면, 집합의 값에 밸류 대신 연관을 맺는 식별자가 온다는 점이다.



@ElementCollection 을 이용하기 때문에 Product 를 삭제할 때 매핑에 사용한 조인테이블의 데이터도 함께 삭제된다. 애그리거트를 직접 참조하는 방식을 사용했다면 영속성 전파나 로딩 전략을 고민해야 하는데 ID 참조 방식을 사용함으로써 이런 고민을 할 필요가 없다.



# 애그리거트 로딩 전략

JPA 매핑을 설정할 때 항상 기억해야 할 점은 애그리거트에 속한 객체가 모두 모여야 완전한 하나가 된다는 것. 즉, 다음과 같이 애그리거트 루트를 로딩하면 루트에 속한 모든 객체가 완전한 상태여 함을 의미

```java
//Product는 완전한 하나여야 한다.
Product product = productRepository.findById(id);
```



*조회 시점에서 애그리거트를 완전한 상태가 되도록 하려면* 애그리거트 루트에서 연관 매핑의 조회 방식을 ***즉시 로딩(FetchType.EAGER)*** 으로 설정. 즉, 다음과 같이 컬렉션이나 `@Entity` 에 대한 매핑의 fetch 속성을 즉시 로딩(FetchType.EAGER) 으로 설정하면 EntitiyManager#find() 메서드로 애그리거트 루트를 구할 때 연관된 구성요소를 DB에서 함께 읽어 온다.



```java
// @Entity 컬렉션에 대한 즉시 로딩 설정
@OneToMany(cascade = {CascadeType.PERSIST, CascadeType.REMOVE},
            orphanRemoval = true, fetch = FetchType.EAGER)
@JoinColumn(name = "product_id")
@OrderColumn(name = "list_idx")
private List<Image> images = new ArrayList<>();

// @Enbeddable 컬렉션에 대한 즉시 로딩 설정
@ElementCollection(fetch = FetchType.EAGER)
@CollectionTable(name = "order_line", joinColumns = @JoinColumn(name = "order_number"))
@OrderColumn(name = "line_idx")
private List<OrderLine> orderLines;
```



**즉시 로딩 방식**으로 설정하면 애그리거트 루트를 로딩하는 시점에 애그리거트에 속한 모든 객체를 함께 로딩할 수 있는 장점이 있지만, 이 장점이 항상 좋은 것은 아니다. 

특히, 컬렉션에 대한 로딩 전략을 FetchType.EAGER로 설정하면 오히려 즉시 로딩 방식이 문제가 될 수 있다.



**로딩전략에 따른 성능 은 튜트리얼로 좀 더 배워야겠다!**

