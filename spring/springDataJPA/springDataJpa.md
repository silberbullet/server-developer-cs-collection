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
