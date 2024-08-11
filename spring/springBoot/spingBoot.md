# Spring Boot CS

Spring Boot는 Spring Framework을 쉽게 사용하기 위해 나온 기술</br>

## 목차

1. [스프링 부트란?](#1-스프링-부트란)
2. [스프링 부트의 중요 목적](#2-스프링-부트의-중요-목적)
3. [build.gradle의 plugins란?](#3-buildgradle의-plugins란)
4. [Spring의 AutoConfiguration](#4-spring-boot의-autoconfiguration)
5. [Spring Boot의 run](#5-spring-boot의-run)
6. [Spring Boot는 HikariCP를 지향한다](#6-spring-boot는-HikariCP를-지향한다.)
7. [Spring Boot JdbcTemplate](#7-spring-boot-jdbctemplate)

### 1. 스프링 부트란?

    - 독립적이고, 실제 제품 수준의 애플리케이션을 쉽게 만들어준다.
    - 특정 버전의 서드파티 라이브러리들을 특정한 방법으로 사용할 수 있도록 확장점을 제공한다.
    - 최소한의 노력으로 시작할 수 있도록 틀을 만들어 놓았다.
    - 스프링 부트는 최소한의 설정만 필요하다. 그러나, 보안 관련 설정 등 추작적인 설정이 추가로 될 가능성이 크다.
    - .jar을 이용하여 쉽게 실행이 가능하다.
    - war로 묶어서 배포 할수 있다.
    - 커맨드 라인 툴도 제공한다.

### 2. 스프링 부트의 중요 목적

    - 아주 손쉽게 프로젝트를 시작할 수 있도록 한다.
    - 주관을 가지고 만들어진 제품이지만 요구사항이 다양해짐으로써 주어진 틀에서도 얼마든지 벗어 날 수 있도록 한다. `커스텀마이징 가능` (최대 장점이라 생각함)
    - 프로덕션을 위한 모니터링, 헬스 체크등의 기능도 제공한다.
    - 코드제네레이션을 제공하는 도구가 아니다. XML 설정은 필요 없음

### 3. build.gradle의 plugins란?

    ```
    plugins {
    id 'java'
    id 'org.springframework.boot' version '3.3.2'
    id 'io.spring.dependency-management' version '1.1.6'
    }
    ```

`org.springframework.boot`는 플러그인을 단독으로 적용하면 프로젝트는 아무런 변화가 없다. 대신 다른 플러그인이 적용되는 시점을 감지하고 반응한다.

`java`가 플러그인이 적용되면 실행 가능한 jar빌드 작업이 자동으로 구성된다.

`io.spring.dependency-management` 이 플러그인이 **핵심**이라고 생각한다. 이 플러그인으로 통해 현재 사용하고 있는 spring boot 버전에 맞게 호환이 가능한 dependencies의 라이브러리 버전들을 가져올 수 있게 된다 dependencies의 Spring boot 문서에 굳이 버전을 안 붙여도 된다는 이유기도 하다.

### 4. Spring Boot의 AutoConfiguration

Spring Boot는 dependencies의 라이브러리들의 자동 설정을 지원한다. gradle 빌드 시 Gradle에 `org.springframework.boot:srping-boot-autoconfigure` 라이브러리를 가져온다. 해당 라이브러리의 META-INF를 들어가게 된다면 Spring가 자동으로 읽을 클래스들 목록으로 정해둔 `spring.factories`파일이 존재한다. 정확히는 `org.springframework.boot.autoconfigure.admin.SpringApplicationAdminJmxAutoConfiguration`키 값의 Value들을 읽는다.

예시

만약 jdbc 라이브러리를 설정하게 된다면 다음 같은 클래스 를 읽는다.

```java
    @AutoConfiguration(before = SqlInitializationAutoConfiguration.class)
    @ConditionalOnClass({ DataSource.class, EmbeddedDatabaseType.class })
    @ConditionalOnMissingBean(type = "io.r2dbc.spi.ConnectionFactory")
    @EnableConfigurationProperties(DataSourceProperties.class)
    @Import({ DataSourcePoolMetadataProvidersConfiguration.class, DataSourceCheckpointRestoreConfiguration.class })
    public class DataSourceAutoConfiguration {

    ...
    }
```

@EnableConfigurationProperties(DataSourceProperties.class)의 DataSourceProperties을 확인하면

```java
   @ConfigurationProperties(prefix = "spring.datasource")
   public class DataSourceProperties implements BeanClassLoaderAware, InitializingBean {

       private ClassLoader classLoader;

       /**
        * Whether to generate a random datasource name.
        */
       private boolean generateUniqueName = true;

       /**
        * Datasource name to use if "generate-unique-name" is false. Defaults to "testdb"
        * when using an embedded database, otherwise null.
        */
       private String name;

       /**
        * Fully qualified name of the connection pool implementation to use. By default, it
        * is auto-detected from the classpath.
        */
       private Class<? extends DataSource> type;

       //...

       }
```

JDBC 설정들을 확인 할수가 있다. 우리가 왜 `application.yml` 파일에 spring.datasource를 작성하는 알 수가 있다.

### 5. Spring Boot의 run

Spring Boot를 Init 하면 @SrpingBootApplication 어노테이션이 붙인 클래스가 생긴다. 메소드를 보면 `public static void main(String[] args)`가 보인다.

```java
    @SpringBootApplication
    public class TransactionApplication {

        public static void main(String[] args) {
            SpringApplication.run(TransactionApplication.class, args);
        }

    }
```

main 메소드 안에는 SpringApplication의 static 메소드 run이 실행이 된다. run 안에는 어노테이션을 붙였던 해당 파일.class가 파라미터로 들어간다. `이 때 부터 Spring Boot빈이 생성하게 된다.` 해당 빈은 @SpringBootApplication 어노테이션에 따라 Bean (Component)를 탐색하고 필요시 빈을 생성하고 관리하게 되는 것이다.👍

```java
    @SpringBootApplication
    public class TransactionApplication implements CommandLineRunner {

        public static void main(String[] args) {
            SpringApplication.run(TransactionApplication.class, args);
        }

        @Autowired
        List<Object> objects;

        @Override
        public void run(String... args) throws Exception {
            for (Object object : objects) {
                System.out.println(object.getClass().getName());
            }
        }

    }
```

다음을 통해서 Spring Boot가 자동으로 빈으로 만들어주는 객체를 확인이 가능하다.

### 6. Spring Boot는 HikariCP를 지향한다.

Spring Boot 문서를 참조해보면 다양한 Conneection Pool중 HikariCP를 되도록 사용할 것이라고 한다. Spring Boot가 자동으로 생성되는 Bean 목록을 살펴보면
`com.zaxxer.hikari.HikariDataSoucre`를 가져온다. HikariCP는 타 CP 중에 가장 성능이 좋은 걸로 확인이 됐다.

**Connection Pool 이란?**

    Connection Pool 인터페이스는 DataSoure를 통해서 사용할 수 있도록 한다. Connection Pool의 구현체가 HikariCp이다.

    Connection Pool은 미리 DB와 연결을 맺어 놓는다. 이를 connection이라 한다. Connection Pool은 기본적으로 여러 connection을 가지고 있으며, 사용자가 직접 connection 수를 조절 할 수 있다.

    사용자가 DB를 이용하고 싶을 때는 DataSource를 통해서 Connection Pool에서connection을 빌려온다. 사용자가 connection을 완료 시키면 DataSource 쪽에서는 Connection Pool에게 사용 완료한 connection을 되돌려준다.

**무한정 Connection을 요청하기만 하면?**

```java
    @SpringBootApplication
    public class TransactionApplication implements CommandLineRunner {

        public static void main(String[] args) {
            SpringApplication.run(TransactionApplication.class, args);
        }

        @Autowired
        DataSource datasource;

        @Override
        public void run(String... args) throws Exception {

            List<Connection> list = new ArrayList<>();

            while (true) {
                Connection conn = datasource.getConnection();

                list.add(conn);

                Thread.sleep(100);
            }
        }
    }
```

위와 같이 지속적으로 Connection을 반환하지 않고 가져오게 된다면 Hikari는 더 이상 가져올 connection이 없어 서버가 죽어버리게 된다.

### 7. Spring Boot JdbcTemplate.

여기까지 정리하면 Spring Boot의 JDBC 생태계는 이렇다.

- `JDBC`: 자바에서 데이터베이스와 상호작용하기 위한 표준 API입니다. JDBC는 데이터베이스와 연결하고 SQL 쿼리를 실행하는 방법을 정의하는 인터페이스와 클래스들을 제공합니다. 여기서 주의할 점은 JDBC랑 Connection Pool이 엄연히 다른 개념이라는 것입니다. DB 상호작용할 때 쓰이지만 JDBC는 자바 애플리케이션이 DB에 접근하고, SQL 쿼리를 실행하며, 그 결과를 처리하기 위한 API입니다. `Connection Pool`은 DB 연결을 효율적으로 관리하기 위한 메커니즘이라고 보면 됩니다.

- `HikariCP`: JDBC를 사용하여 데이터베이스 연결을 효율적으로 관리하는 커넥션 풀링 라이브러리입니다. HikariCP는 JDBC를 기반으로 데이터베이스 연결을 재사용하고, 성능을 최적화하는 기능을 제공합니다. 자주 사용되는 데이터베이스 연결을 미리 생성해두고, 필요할 때마다 이 연결을 재사용할 수 있도록 해줍니다.

- `DataSource` : 자바의 JDBC API에서 DB 연결을 관리하는데 사용되는 인터페이스 이다.

그렇다면 `JdbcTemplate`은 결국 Spring 프레임워크에서 제공하는 클래스로, JDBC를 보다 쉽게 사용할 수 있도록 도와주는 유틸리티 클래스 이다. JDBC 코드의 양을 줄이고 DB 작업을 간편하게 처리할 수 있도록 도와주는 클래스이다. JDBC를 생으로 쓰면 반복적인 연결 생서, SQL 실행, 예외 처리, 리소스 관리등을 자동화 하여 개발자가 더 간단하게 DB와 상호작용할 수 있게 해준다.

```java

    // 기존의 Connection을 이용한 DB 접근

    String sql = "INSERT INTO MEMBER(member_id, money) VALUES (?, ?)";

    Connection con = null;
    PreparedStatement pstmt = null;

    try {
        con = getConnection();
        pstmt = con.prepareStatement(sql);
        pstmt.setString(1, member.getMemberId());
        pstmt.setInt(2, member.getMoney());
        pstmt.executeUpdate();
        return member;
    } catch (SQLException e) {
        throw exceptionTranslator.translate("save", sql, e);
    } finally {
        // TCP/IP 커넥션으로 연결되기 때문에 자원 해제해줘야 함
        close(con, pstmt, null); // 현재 코드에는 없지만 자원을 해제해주는 사용자 정의 메서드
    }

```

```java

    // JdbcTemplate 사용할 때 이용한 DB 접근
    // 불필요한 Connect, 예외처리, statement가 사라진걸 확인 할 수 있다.
    ...
    private final JdbcTemplate template;
    public MemberRepository(DataSource dataSource) {
        this.template = new JdbcTemplate(dataSource);
    }
    ...

    @Override
    public Member save(Member member) {
        String sql = "INSERT INTO MEMBER(member_id, money) VALUES (?, ?)";
        template.update(sql, member.getMemberId(), member.getMoney());
    return member;

```

하지만 아직 까지도 우리는 SQL문을 작성하고 해야한다. 아직 수작업이 더 필요한 상황이다. 이를 더 개선하기 위한 Spring Data JPA를 확인해보도록 하자. [Spring Data JPA CS](#7-spring-boot-jdbctemplate)
