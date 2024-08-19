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