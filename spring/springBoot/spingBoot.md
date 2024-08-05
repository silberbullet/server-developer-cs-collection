# Spring Boot CS

Spring Boot는 Spring Framework을 쉽게 사용하기 위해 나온 기술</br>

## 목차

[1.스프링 부트란?](1.스프링-부트란?)
[2.스프링 부트의 중요 목적](2.스프링-부트의-중요-목적)
[3.build.gradle의 pulgins란](3.build.gradle의-pulgins란?)
[4.Spring의 AutoConfiguration](4.Spring의-AutoConfiguration?)
[5.Spring Boot의 run](5.Spring-Boot의-run)

### 1.스프링 부트란?

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

### 3. build.gradle의 pulgins란?

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

### 4. Spring Boot의 AutoConfiguration?

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
