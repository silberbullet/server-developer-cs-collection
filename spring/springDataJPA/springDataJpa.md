# Spring Data JPA CS

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

`'org.springframework.boot:spring-boot-starter-data-jpa'` 라이브러리를 가져오게 되면 Spring Boot AutoConfigure을 통해 `HibernateJpaAutoConfiguration` 설정 파일을 읽게 된다. 해당 클래스도 JDBC를 사용하기 때문에 dataSource 프로퍼티를 applicatioin.yml에서 읽어오게 되고, JPA 프로퍼티스 설정도 읽게 된다. **설정이 안되면 빌드 실패가 일어난다.**

```yaml
    // 설정 예시
    spring:
        // DB 연결 설정
        datasource:
            url: jdbc:mysql://localhost:3306/transaction
            username: root
            password: root1234
            driver-class-name: com.mysql.cj.jdbc.Driver
        // JPA 설정
        jpa:
            show-sql: true
            generate-ddl: false
            hibernate:
                ddl-auto: none
                naming:
                    physical-strategy: org.hibernate.boot.model.naming.PhysicalNamingStrategyStandardImpl
            properties:
                hibernate:
                    dialect: org.hibernate.dialect.MySQL8Dialect

```
