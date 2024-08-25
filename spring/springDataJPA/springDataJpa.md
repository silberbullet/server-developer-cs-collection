# Spring Data JPA CS

## 0. SQL의 문제점 

SQL에 모든 것을 의존하는 상황에서 개발자들은 DAO 계층에서 DBMS에 SQL을 전달한다. 유지보수 관점으로 본다면 일일히 DAO를 열어서 어떤 SQL이 실행되고 어떤 객체들이 함께 조회되는 확인해야한다.

이것은 진정한 의미의 계층 분할이 아니다. 무리적으로 SQL과 JDBC API를 데이터 접근 계층에 숨기는 데 성공했을지는 몰라도 논리적으로는 엔티티와 아주 강한 의존 관계를 맺고 있다.
이 때문에 만약 회원을 조회할 때 물론이고 회원 객체에 필드를 하나 추가할 때도 DAO의 CRUD코드와 SQL 대부분을 변경해야 하는 불편함이 발생한다.

즉, SQL을 사용하는 우리들의 문제점을 이렇다

    - 진정한 의미의 계층 분할이 어렵다.
    - 엔티티를 신뢰할 수 없다
    - SQL에 의존적이 개발을 피하기 어렵다

**`패러다임 불일치`**

자바 객체는 속성(필드)과 기능(메소드)을 가진다. 객체의 기능은 클래스에 정의되어 있으므로 객체 인스턴스의 상태인 속성만 저장했다가 필요할 때 불러와서 복구한다.
객체가 단순하면 객체의 모든 속성 값을 꺼내고 파일이나 DB에 저장혐 되지만, 부모 객체 상속이나 다른 객체를 참조하고 있다면 객체의 상태를 저장하기는 쉽지 않다.

관계형 DB는 데이터 중심으로 구조화되어 있고, 집합적인 사고를 요구한다. 그리고 객체지향에서 이야기하는 추상화, 상속, 다형성 가은 개념이 없다.

객체와 관계형 DB는 지향하는 목적이 서로 다르므로 둘의 기능과 표현 방법도 다르다. 이것을 객체와 관계형 DB의 패러다임 불일치 문제 라 한다. 따라서 객체 구조를 테이블 구조에 저장하는 데는 한계가 있다.

이를 JPA가 해결했다.

## 1. JPA 발자취

JPA를 쓰기 전 우리의 JDBC 프로그래밍을 해왔다.

    0.DBMS 접속
    1.SQL 작성
    2.SQL 실행, DBMS 안에서 실행
    3.ResultSet으로 한건한거 읽어와서 처리.

**객체지향적으로 프로그래밍(ORM)으로 한다는 것은 무슨 의미일까?**

SQL을 알아야 할까? SQL DBMS마다 다를 거다. Oracle, Mysql 등등 으로 DBMS마다 SQL 문법이 다르다. 우리는 SQL을 쓰지 않고, 저장소에서 자동으로 객체 가지고 매핑을 해주는 기술을 찾기 시작했다.

`EJB Entity Bean`이라는 기술이를 시작을 하였다. 하지만 복잡성이 너무 강하여 Hibernate가 2001년에 등장하였다.
`Hibernate (ORM) 프레임워크` SQL 없이도 객체를 가지고 관계를 매핑해주는 기술이 만들어 졌지만, 그 밖에 다양한 기술들이 만들어졌다. 이런 비슷한 프레임워크 및 기술들이 개발이 되니 결국 `JPA(Java Persistence API)`라는 표준이 2006년도에 발표가 됐다. JPA는 결국 인터페이스 이며, ORM 표준 API이다. 이를 구현한게 Hibernate가 대표적이다.

JPA를 구현하고 있는 것이 Hibernate 같은 기술이다. 그런데 Spring은 JPA와 관련된 객체를 손쉽게 Bean으로 생성해서 관리해주려 한다.

Data를 관리하기 위해서 관계형 DB(MySql, Oracle)를 쓰고, NoSql(MongoDB), 메로리에 저장하는 (Redis) 등의 저장소가 존재한다. 이를 사용하기 위해서 사용법을 일일히 다 알아야 한다. Spring Data 프레임워크는 데이터를 저장하는 기술들을 통일하기 위한 추상화한 프레임워크이다. **Spring Data JPA가 익숙해지면 MongoDB나 Redis 사용법이 익숙해지게 되는거다.**
하지만 어디까지나 Spring Data JPA는 JPA의 껍데기 이기 때문에 내부 동작을 이해하기 위해서는 JPA의 구현체인 Hibernate를 공부하고 ORM의 동작원리를 알아야 한다.

## 2.Spring Data JPA 라이브러리

```gradle
    dependencies {
        implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
        implementation 'org.springframework.boot:spring-boot-starter-web'

        compileOnly 'org.projectlombok:lombok'
        annotationProcessor 'org.projectlombok:lombok'

        runtimeOnly 'com.mysql:mysql-connector-j'

    }
```

`'org.springframework.boot:spring-boot-starter-data-jpa'` 라이브러리를 가져오게 되면 Spring Boot AutoConfigure을 통해 `HibernateJpaAutoConfiguration` 설정 파일을 읽게 된다. 해당 클래스도 JDBC를 사용하기 때문에 dataSource 프로퍼티를 applicatioin.yml에서 읽어오게 되고, JPA 프로퍼티스 설정도 읽게 된다.
**설정이 안되면 빌드 실패가 일어난다.**

```yaml
    # 설정 예시
    spring:
        # DB 연결 설정
        datasource:
            url: jdbc:mysql://localhost:3306/transaction
            username: root
            password: root1234
            driver-class-name: com.mysql.cj.jdbc.Driver
        # JPA 설정
        jpa:
            generate-ddl: false  # Spring Data JPA가 DDL을 생성하지 않도록 설정
            hibernate:
              ddl-auto: update  # Hibernate의 스키마 자동 처리 전략 설정 (올바른 값으로 설정 필요)
            properties:
              hibernate:
                show_sql: true  # SQL 쿼리를 로그에 출력
                format_sql: true  # SQL 쿼리를 보기 좋게 포맷팅
                use_sql_comments: false  # SQL 주석 사용 여부
                id.new_generator_mappings: true  # 새로운 Identifier Generator 전략 사용
                dialect: org.hibernate.dialect.MySQL8Dialect  # MySQL 8을 위한 Hibernate 방언 설정
                naming.physical-strategy: org.hibernate.boot.model.naming.PhysicalNamingStrategyStandardImpl  # 물리적 네이밍 전략 설정
```
각각 JPA 기본적인 프로퍼티스 설정 내용은 이렇다. 여기서 `ddl-auto`의 특성은 이렇다.
 
    none: 아무 작업도 하지 않음.
    validate: 엔티티 클래스와 데이터베이스 스키마를 비교하여 일치하는지 검증만 함.
    update: 엔티티 클래스의 변경 사항에 따라 데이터베이스 스키마를 업데이트함.
    create: 애플리케이션 시작 시 데이터베이스 스키마를 새로 생성함.
    create-drop: 애플리케이션 시작 시 스키마를 생성하고, 애플리케이션 종료 시 이를 삭제함.

만약 테스트용으로 사용하고 싶다면 update를 추천 드리며, 운영 환경에서는 얘기치 않는 결과를 방지하기 위해 `validate`나 `none`으로 설정하고, 스키마 변경은 수동으로 관리하는 것이 좋다.
`generate-ddl`도 Spring Data JPA가 스키마를 생성할지 여부를 결정하는 용도로 쓰인다. 하지만 `ddl-auto`가 더 세밀하기 때문에 `ddl-auto`를 권장한다.
**`jpa.propeties.hibernate.dialect`** 는 JPA가 해당 DBMS에 문법을 사용하기 위한 설정이다. 

## 3.EntityManger

**JPA에서 가장 중요한 객체는 `EntityManager`라고 말한다.** Spring boot (jpa 라이브러리 포함) 를 실행하게 된다면 EntityManagerFactory 빈이 자동으로 생성 된다. `JPA는 본래 EntityManger를 사용하여 프로그래밍이 시작된다.`
그걸 가능하게 해주기 위해 Spring Data JPA는 EntityManager를 제공하는 것이다. 

다음 JPA 기본 코딩 방식을 보자.

```java
    @SpringBootApplication
    public class TransactionApplication implements CommandLineRunner {

        public static void main(String[] args) {
            SpringApplication.run(TransactionApplication.class, args);
        }

        @Autowired
        EntityManagerFactory entityManagerFactory;

        @Override
        public void run(String... args) throws Exception {
            
            EntityManager entityManager = entityManagerFactory.createEntityManger();
            
            EntityTransaction transaction = entityManager.getTransaction();
            
            try{
                transaction.begin();
                // JPA 관련된 코드
                transaction.commit();
            } catch (Exception e) {
                transaction.rollback();
            } finally {
                entityManager.close();
            }
        }
    }
```
> 엔티티는 ORM과 DB 설계에서 중요한 개념으로 DB 테이블과 매핑되는 클래스 또는 그 클래스를 기반으로 한 객체를 의미한다.

`EntityManger`는 엔티티를 저장하고, 수정하고, 삭제하고, 조회하는 등 엔티티와 관련된 모든 일을 처리한다. 이름 그대로 엔티티를 관리하는 관리자다. 

## 4. 엔티티 매핑

JPA를 사용해서 테이블과 매핑할 클래스는 `@Entity` 어노테이션을 필수로 붙여야 한다. @Entity가 붙은 클래스는 JPA가 관리하는 것으로, 엔티티라 부른다. 
**@Entity 사용 시 주의 상황은 다음과 같다**

    - 기본 생성자는 필수다. (파라미터가 없는 public 또는 protected 생성자) // 그래서 기본적으로 @NoArgsConstructor 을 붙인다.
    - final 클래스, enum, interface, inner 클래스에는 사용할 수 없다. 
    - 저장할 필드에 final을 사용하면 안된다.

`@Table`은 엔티티와 매핑할 테이블을 지정한다. 생략하면 매핑한 엔티티 이름을 테이블 이름으로 사용한다.
@Table 속성은 다음과 같다. 
    
    - name : 매핑할 테이블 이름
    - catalog : catalog 기능이 있는 데이터베이스에서 catalog를 매핑한다.
    - schema : schema 기능이 있는 데이터베이스에서 schema를 매핑한다.
    - uniqueConstraints : DDL 생성 시에 유니트 제약 조건을 만든다. 2개 이상의 복합 유니크 제약 조건도 만들 수 있다. 참고로 이기능은 스키마 자동 생성 기능을 사용해서 DDL을 만들 때만 사용된다.

@Entity가 붙은 클래스는 키 값이 필수이다. `@Id` 어노테이션을 붙인 필드명이 필요하다. 각 필드가 테이블 컬럼에 매핑할 수 있도록 해주는 어노테이션은 `@Column`이다. 
아래 회원 도메인을 예로 들어보자.

```java
import java.time.temporal.Temporal;

@Entity
@Table(name = "member")
@Getter
@Setter
public class Member {

    @Id
    @Column(name = "user_id")
    private String userId;

    @Column(name = "user_name")
    private String userName;

    @Column(name = "user_age")
    private Integer userAge;

    @Column(name = "role_Type")
    @Enumerated(EnumType.String)
    /**
     * 자바의 enum을 사용해서 회원의 타입을 구분했다
     * 일반 회원은 USER, 관리자는 ADMIN이다. enum으로 매핑 시 
     * @Enumerated 어노테이션 매핑 필요
     */
    private RoleType roleType;

    @Column(name = "reg_date")
    @Temporal(TemporalType.TIMESTAMP)
    /**
     * 날짜 타입은 @Temporal을 사용하여 매핑한다.
     */
    private Date regDate;
    
    @Lob
    /**
     *  회원을 설명하는 필드는 길이 제한이 없다. 따라서 DB에 CLOB 타입으로 저장해야 할 경우 @Lob을 사용한다.
     */
    private String descriptioin;
}

public enum RoleType {
    ADMIN, USER
}
```
regDate를 대신 다음 같이 사용 할 수도 있다.

```java

    // 현재시간이 저장 될 때 자동으로 생성
    @CreationTimeStamp
    private LocalDateTime regdate;

```

그렇다면 이 Member를 가지고 EntityManger를 이용해서 트랜잭션을 만들어보면 다음과 같다. 

```java
    @SpringBootApplication
    public class TransactionApplication implements CommandLineRunner {

        public static void main(String[] args) {
            SpringApplication.run(TransactionApplication.class, args);
        }

        @Autowired
        EntityManagerFactory entityManagerFactory;

        @Override
        public void run(String... args) throws Exception {
            
            EntityManager entityManager = entityManagerFactory.createEntityManger();
            
            EntityTransaction transaction = entityManager.getTransaction();
            
            try{
                transaction.begin();
                // JPA 관련된 코드
                Member member = new Member();
                
                member.setUserId("user");
                member.setUserName("경우");
                member.setuserAge(29);
                member.setRoleType("USER");
                
                // insert 문이 실행하게 된다.
                entityManager.persist(member);
                
                transaction.commit();
            } catch (Exception e) {
                transaction.rollback();
            } finally {
                entityManager.close();
            }
        }
    }
```
> 앞으로 DB 설계 시 Drawio 사이트 추천한다. [Draw.io](https://app.diagrams.net/)

## 5. Persistence Context (영속성 컨텍스트)

JPA를 이애하는 데 가장 중요한 용어는 `영속성 컨텍스트`이다 . `엔티티를 영구 저장하는 환경`이라는 뜻으로써 엔티티 매니저로 엔티티를 저장하거나 조회하면 엔티티 매니저는 영속성 컨텍스트에 엔티티를 보관하고 관리한다.
다음 그림을 확인해보자.

![persistenceContext drawio](https://github.com/user-attachments/assets/fd48ae17-a80e-403a-ade2-ae8a75de9b89)


`EntityManager`는 find() 명령을 실행 시, DB에서 찾아온 값을 영속성 컨테스트에 저장한다. Entity는 영속성 컨테스트에 저장된 `원본 엔티티`를 참조하게 된다. **User1과 User2는 같은 User를 참조하고 있기 때문에 두개는 같은 객체이다.**

여기서 User1이 `setPassword('5678')`을 통해 비밀번호를 변경했다고 가정해보자. User2 또한 getPassword시 `5678` 값을 가져오게 된다. 

비밀번호가 변경된 시점에서 commit() 시, 중요한 상황이 있다. **find()시 영속성 컨테스트에는 `원본 엔티티`의 복사본인 `스냅샷`을 생성한다. **commit시 `원본 엔티티`와 `스냅샷`을 비교하여 차이가 있다면 `원본 엔티티`를 update 처리한다**
우리는 setPassword를 통해서 `원본 엔티티`의 비밀번호를 바꾸었고, 영속성 컨테스트는 `스냅샷`과 비교하여 변화를 감지하고 `UPDATE`처리가 일어나게 된다.

## 6. JPA 키 생성 전략

Entity에 @Id 어노테이션을 붙인 필드는 그 테이블에 key를 설정한다. JPA가 제공하는 DB 기본 키 생성 전략은 다음과 같다.

1. 직접 할당 : 기본 키를 애플리케이션에서 직접 할당한다.

2. 자동 생성 : 대리 키 사용 방식
   
   1) IDENTITY : 기본 키 생성을 DB에 위임한다.
      - 주로 MySQL, PostgreSQL, SQL server, DB2에서 사용.
      - IDENTITY 전략은 MySQL의 AUTO_INCREAMENT 처럼 DB에 값을 저장하고 나서야 기본 키 값을 구할 수 있을 때 사용.
      - @GenerateValue(strategy = GenerationType.IDENTITY) 어노테이션 설정
      - **IDENTITY 생성 전략은 DB에 엔티티를 `즉시 저장`하고 JPA에서 조회 후 식별자를 정함. 그래서 `쓰기 지연`이 동작을 안함**
      - JDBC3의 Statement.getGeneratedKeys()를 사용하면 데이터 저장과 동시에 기본 키 값을 할당 받음 ( Hibernate에서 사용 중 )
   
   2) SEQUENCE : DB 시퀀스르 사용해서 기본 키를 할당한다.
      - 주로 Oracle, PostgreSQL, DB2, H2에서 사용
      - `@SequenceGernerator` 어노테이션으로 시퀀스 생성기를 생성. 
      - @GenerateValue(strategy = GenerationType.SEQUNCE , generator = "시퀀스 생성기 이름") 어노테이션 설정
      - 시퀀스 증가 값은 기본적으로 50 임, `allocationSize = 50` ( 성능 최적화를 위해 )
      - **SEQUENCE 전략은 시퀀스를 통해 식별자를 조회하는 추가 작업이 필요하여 DB와 `2번` 통신함**
      - **JPA는 이를 해결하기 위해 allocationSize값을 50을 기본값으로 설정함, JVM이 동시에 동작해도 기본 값이 충돌하지 않음**

   3) TABLE : 키 생성 테이블을 사용한다.
      - 키 생성 전용 테이블 하나 만들고 여기에 이름과 값으로 사용할 컬럼을 만들어 DB 시퀀스를 흉내내는 것.
   
   4) AUTO : @GenerateValue(strategy = GenerationType.AUTO) 는 선택한 DBMS에 따라 전략을 알아서 설정.

김영한 님이 말아주는 권장하는 식별자 선택 전략

1. DB 기본키는 3가지 조건을 만족할 것 
    - NULL 값 허용 X
    - 유일성
    - 변해선 안됨
   
2. 테이블의 기본 키를 선택하는 전략은 크게 2가지
    - 자연키
      - 비즈니스에 의미가 있는 키 ( 주민번호, 이메일, 전화번호 )
    - 대리 키
      - 비즈니스와 관련 없은 임의로 만들어진 키, 대체 키로도 불림 ( 오라클 시퀀스, auto_increment)

3. 자연 키보다는 `대리 키`를 권장

4. 비즈니스 환경은 언젠간 변한다.
   - 기본키의 조건을 현재는 물론이고 **미래까지 충족하는 자연 키를 찾기는 쉽지 않다.**

5. JPA는 모든 엔티티에 일괄된 방식으로 대리 키 사용을 권장한다.

## 7. JPA Entity 매핑 

만약 상품 테이블과 주문 테이블이 존재한다고 가정해보자. 상품 하나의 주문이 여러개 일 수 있으니, 상품과 주문은 1:N ( 혹은 N:1 ) 관계를 가진다.
RDBS 기준으로 본다면 주문에는 상품 코드가 외래키로 쓰인다. **두 개의 엔티티에는 외래키 참조 형식이 필요하다**

`N:1` 관계로 엔티티 객체를 매핑 할 때는 JPA는 다음과 같이 쓰인다.

```java

@Entity
@Table(name = "order_item")
@NoArgsConstructor
@Getter
public class OrderItem extends BaseEntity {

    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "ord_id")
    private long id;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "item_id")
    private Item item;

    @Column(name="ord_cnt", nullable = false)
    private Integer ordCnt;

    @Column(name="ord_prc", nullable = false)
    private Integer ordPrc;
}

```

   - @ManyToOne : 다대일 관계라는 매핑 정보, 연관계를 매핑할 때 다중성을 나타내는 어노테이션 필수
     - fetch 
       - FetchType.LAZY: 연관된 엔티티를 실제로 접근할 때 로드. 메모리 효율적이지만, 잘못 사용하면 예상치 못한 쿼리 발생으로 성능 문제가 발생 (N+1 문제)
         ```java
               Order order = orderRepository.findById(orderId).get();  // 이 시점에서는 customer가 로드되지 않음
               Item item = order.getItem();  // 이 시점에서야 customer가 로드됨 (쿼리 발생)
         ```
         - **N+1 발생하는 예제** 
            ```java
                // Orders를 한 번의 쿼리로 조회
               List<Order> orders = orderRepository.findAll();  // (1)번 쿼리 발생
               
               for (Order order : orders) {
               System.out.println(order.getItem().getName());  // N번의 쿼리 발생 (각 주문마다 상품 조회)
               }
            ```

       - FetchType.EAGER: 연관된 엔티티를 즉시 로드. 이후의 데이터베이스 접근을 줄일 수 있지만, 필요 없는 데이터를 로드함으로써 성능 저하가 발생

   - @JoinColumn(name = "item_id") : 외래 키를 매핑할 때 사용, 주문은 상품코드를 외래키로 쓰인다.