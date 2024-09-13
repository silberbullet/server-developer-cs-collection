# 좋은 코딩을 위한 배경 지식

## 1. 객체지향 개발 5대 원리:SOLID

[참조](https://www.nextree.co.kr/p6960/)

1. SRP (단일책임의 원칙 : Single Responsibility Principle)

- 정의

작성된 클래스는 하나의 기능만 가지며 클래스가 제공하는 모든 서비스는 그 하나의 책임을 수행하는 데 집중되어 있어야 한다는 원칙이다. 어떤 변화에 의해 클래스를 변경해야 하는 이유는 오직 하나뿐이어야 함을 의미한다.
SRP원리를 적용하면 무엇보다도 책임 영역이 확실해지기 때문에 코드의 가독성, 유지보수 용이라는 이점까지 누릴 수 있다.

- 적용방법

`리팩토링`에서 소개하는 대부분의 위험상황에 대한 해결 방법은 직/간접적으로 SPR원리와 관련이 있으며, 이는 항상 코드를 최상으로 유지한다는 리팩토링 근본 정신도 항상 객체들의 책임을 최상의 상태로 분배한다는 것이다.

즉 여러 원인에 의한 변경을 통해 혼재된 각 `책임`을 각각의 개별 클래스로 분할하여 클래스 당 하나의 책임만을 맡도록 하는 것이다. `더 나아가 책임만 분리하는 것이 아니라 분리된 두 클래스간의 관계의 복잡도를 줄이는 것`이 중요하다. `복잡도`를 줄인다는 의미는 `클래스들 간의 상호 의존성과 그에 따른 유지보수의 어려움을`을 줄이는 것이다. 예를 들어 **공통적으로 공유되는 요소를 부모 클래스로 정의하여 부모 클래스에 위임하는 기법이다.** 또한 책임을 기존의 어떤 클래스로 모으거나, 이럴만한 클래스가 없다면 새로운 클래스를 만드는 방법도 해결 방법이다. 산발적으로 여러 곳에 분포된 책임을 한곳으로 모으며 설계도 방법이다.(산탄총 수술)

예를들어 [고유정보 + 변화하는 정보가 한 클래스]에 있다면 [ 고유정보 클래스 , 변화하는 정보 클래스] 로 분리해보자 [ 변화하는 정보 클래스 ]는 [ 고유정보 클래스 ]를 상속 받음으로써, 클래스의 용도를 유지한채 확장성의 열려있게 된다.

2. OCP (개방폐쇄의 원칙 : Open Close Principle)

- 정의

소프트웨어의 구성요소(컴포넌트, 클래스, 모듈, 함수)는 확장에는 열러있고, 변경에는 닫혀있어야 한다는 원칙이다. 변경을 위한 비용은 줄이고 확장을 위한 비용은 가능한 극대화 해야 하며, 요구사항의 변경이나 추가사항이 발생하더라도, 기존 구성요소는 수정이 일어나지 말아야 한다.
기존 구성요소를 쉽게 확장해서 재사용할 수 있어야 한다. 재사용 가능한 코드를 만드는 기반은 **추상화**, **다형성**이다. 

- 적용방법

`변경(확장) 될 것과 변하지 않을 것을 엄격히 구분`부터 시작이다. 정의한 인터페이스에 의존하도록 코드를 작성한다.


```java
// 기존 기능
public interface PaymentProcessor {
    void processPayment(double amount);
}

public class CreditCardProcessor implements PaymentProcessor {
    @Override
    public void processPayment(double amount) {
        System.out.println("Processing credit card payment of $" + amount);
    }
}

// 새로운 기능을 추가할 때, 기존 코드를 수정하지 않고 새로운 클래스를 구현함
public class PaypalProcessor implements PaymentProcessor {
    @Override
    public void processPayment(double amount) {
        System.out.println("Processing PayPal payment of $" + amount);
    }
}
```

OCP의 궁극적인 목적은 **기존 코드를 건드리지 않고도 새로운 기능을 추가할 수 있어야 한다는 의미이다.** 다음 예시로 우리는 기존 코드를 굳이 건드리지 말아야 한다는 이유를 알 수 있다.

```java
public class ReportService {
    public void generateReport(String type) {
        if (type.equals("PDF")) {
            System.out.println("Generating PDF report");
        } else if (type.equals("HTML")) {
            System.out.println("Generating HTML report");
        } else if (type.equals("CSV")) {
            System.out.println("Generating CSV report");
        }
    }
}
```
레포트의 타입 별로 처리해야하는 로직이 있다. 요구사항이 XML 등 파일 타입에 추가에 따른 기능을 요구하게 된다면 `generateReport` 메소드에 다음 과 같이 추가가 일어난다.

```java
else if (type.equals("XML")) {
        System.out.println("Generating XML report");
}
```

새로운 if-else문 추가에 따른 기존 코드를 계속 수정해 나간다면, **버그 발생 가능성을 높이고, 코드 유지보수가 어려워진다.** 다음과 같이 바꿔보자

```java

public  interface  ReportGenerator{
    void generateReport();
}

public class PdfGenerator implements ReportGenerator{
    @Override
    public void generateReport(){
        System.out.println("Generating PDF report");
    }
}

public class ReportService {
    
    private ReportGenerator reportGenerator;
    
    public ReportService(ReportGenerator reportGenerator){
        this.reportGenerator = reportGenerator;
    }
    
    public void generateReport(){
        reportGenerator.generateReport();
    }
    
}

// 이제 ReportServices는 파일 타입을 별로 주입을 받아 사용이 가능하다.
```

이제 PDF 파일이 아닌 HTML 생성에 요구사항을 받게 된다면 **HtmlGenerator를 추가하고 generateReport()** 메소드를 작성해주면 된다.

- 주의사항
    - **확장되는 것과 변경되지 않는 모듈을 분리하는 과정에서 크기 조절에 실패**하면 오히려 더 복잡해짐
    - **인터페이스는 가능하면 변경되어 서는 안됨**
    - **모든 설계에서 적당한 추상화 레벨이 필요함**
      - 추상화는 다른 모든 종류의 객체로 부터 식별될 수 있는 객체의 본질적인 특징
      - 행위에 대한 본질적인 정의를 통해 식별

- 인터페이스 외에 다른 OCP 구현
  - 상속
  - 전략 패턴
    - ReportService에 setReportGenerator()을 함으로써 전략적으로 ReportGenerator을 처리
      - **시스템 동작 중에 언제든 ReportGenerator이 바뀔 수 있어, 불변성이 깨짐**
      - 해당 업무 혹은 논리가 불변성이 강해야 한다면 `비추천`
  - 함수형 프로그래밍
    - java.util.function.Consumer 의 accept를 통해 기존 클래스를 정의하지 않고 새로운 동작을 정의

3. LSP (리스코브 치환의 원칙 : The Liskov Substitution Principle)

- 정의

서브 타입은 언제나 기반 타입이 약속한 규약 (public 인터페이스, 물론 메소드가 저니느 예외까지 포함)을 지켜야 한다. 상속은 구현상속이든 인터페이스 상속이든
궁극적으로 다형성을 통한 확장성 획득을 목표로 한다. 

```java
// LSP 이해도를 높이기 위한 예시
// 다음은 LSP 원칙을 위반한 코드 이다.

    public class Bird {
    
        void fly (){
            System.out.println("I can fly");
        }
    }

    public class Eagle extends  Bird{
    
        @Override
        void fly (){
            throw new RuntimeException("hi i'm error bird");
        }
    }
```

서브 클래스 `Eagle`은 `Bird`클래스가 정의한 fly()에서 에러가 없는 print 문이 출력이 되어야 하지만 이를 어겼다.

- 적용방법
**부모 클래스 행동 규약을 자식 클래스가 위반하지 않도록 설계**가 중요하다. `행동 규약`을 위반한다는 것은 자식 클래스가 `오버라이딩`을 할때, 잘못되게 재정의하면 리스코프 치환 원칙을 위배할 수 있다는 의미다.

  - 계약 관계 잘 잘 지키기
  - 확장에도 열려 있기
  - 서브 클래스가 슈퍼 클래스의 일부 메소드만 사용하면 의미가 없다
  - 서브 클래스가 정말로 슈퍼 클래스와 공통점이 없다면 위임으로 변경하기 전에 상속을 제거
  - 추상 팩토리 패턴을 쓰는 방법도 좋다.
    - 여러 제품군 중 하나를 선택해서 시스템을 설정 한다던지, 제품에 대한 클래스 라이브러리를 제공하고, 그들의 구현이 아닌 인터페이스를 노출시키고 싶을 때

```java
// 추상 팩토리 패턴 예
interface Payment {
  // 인증
  void authenticate();
  // 승인
  void authorization();
}

interface PaymentMethod {
  void process(Payment payment);
  void pay();
}

public class Main {
  public static void main(String[] args) {
    // 신용카드 결제
    Payment creditCardPayment = new CreditCardPayment();
    PaymentMethod creditCardMethod = new CreditCardMethod();
    creditCardMethod.process(creditCardPayment);

    // PayPal 결제
    Payment payPalPayment = new PayPalPayment();
    PaymentMethod payPalMethod = new PayPalMethod();
    payPalMethod.process(payPalPayment);
  }
}
```
4. ISP ( 인터페이스 분리의 원칙 : Interface Segreation Principle)

- 정의

한 클래스는 자신이 사용하지 않는 인터페이스는 구현하지 말아야 한다는 원리이다. 즉, **어떤 클래스가 다른 클래스에 종속될 때에는 가능한 최소한의 인터페이스만을 사용해야 한다.**

ISP는 `하나의 일반적인 인터페이스보다는, 여러 개의 구체적인 인터페이스가 낫다`라고 정의한다.

SRP가 클래스의 단일책임을 강조한다면 ISP는 인터페이스의 단일 책임을 강조한다. 하지만 ISP는 어떤 클래스 혹은 인터페이스가 여러 책임 혹은 역할을 갖는 것을 인정

```java
// SRP는 한 클래스는 하나의 책임을 인정하지만
// ISP는 한 클래스의 여러 역할을 인정한다. 다만 인터페이스는 하나의 역할을 해야 한다.

interface Printer {
  void print();
}

interface Scanner {
  void scan();
}

class MultiFunctionDevice implements Printer, Scanner {
  @Override
  public void print() {
    System.out.println("Printing document...");
  }

  @Override
  public void scan() {
    System.out.println("Scanning document...");
  }
}

```

- 적용 방법

**클래스의 상속을 이용하여 인터페이스를 나누기**
- Device라는 공통 부모 클래스를 상속받고, 각 자식 클래스인 PrinterDevice와 ScannerDevice가 각기 다른 인터페이스(Printer, Scanner)를 구현

```java
interface Printer {
    void print();
}

interface Scanner {
    void scan();
}

// 공통된 기능을 제공하는 부모 클래스
abstract class Device {
    String model;
    public Device(String model) {
        this.model = model;
    }
    
    public String getModel() {
        return model;
    }
}

// Printer만 구현하는 클래스
class PrinterDevice extends Device implements Printer {
    public PrinterDevice(String model) {
        super(model);
    }
    
    @Override
    public void print() {
        System.out.println("Printing from " + getModel());
    }
}

// Scanner만 구현하는 클래스
class ScannerDevice extends Device implements Scanner {
    public ScannerDevice(String model) {
        super(model);
    }
    
    @Override
    public void scan() {
        System.out.println("Scanning from " + getModel());
    }
}
```

**객체 인터페이스를 통한 분리**

- MultiFunctionDevice가 Printer와 Scanner 두 가지 인터페이스를 구현해 다기능 장치로 동작하지만, 클라이언트는 Printer만 필요할 경우 Printer 인터페이스만 구현하는 PrintOnlyDevice를 사용

**다만 서로 다른 성격의 인터페이스를 명백히 분리한다.**

5. DIP ( 의존성역전의 원칙 : Dependency Inversion Principle )

- 정의

구조적 디자인에서 발생하던 하위 레벨 모듈의 변경이 상위 레벨 모듈의 변경을 요구하는 위계 관계를 끊는 의미의역전이다.

실제 사용 관계는 바뀌지 않으며, 추상을 매개로 메시를 주고 받아 관계를 최대한 느슨하게 만든다.



## 2. CQRS 패턴

[참조](https://mslim8803.tistory.com/73)

- 정의

`CQRS`란 Command and Query Responsibility Segregation 의 약자로, 데이터 저장소로부터 `읽기`와 `업데이트 작업을 분리`하는 패턴을 말한다. CQRS를 사용하면, 어플리케이션의 퍼포먼스, 확장성, 보안성을 극대화할 수 있다. 또한 CQRS 패턴을 통해 만들어진 시스템의 유연성을 바탕으로, 시간이 지나면서 지속적으로 시스템을 발전하며 여러 요청으로부터 들어온 복수의 업데이트 명령들에 대한 충돌 방지가 가능하다.

- 왜? 사용하게 됐을까?

전통적인 아키텍처는 DB에서 데이터 조회, 업데이트 하는데 같은 데이터 모델이 사용되었다. 간단한 CRUD 작업에 대해서라면 문제 없지만 복잡한 어플리케이션에서 이러한 접근 방식은 `유지 보수를 어렵게 만들 수 있다.` 어플리케이션에서 데이터 조회 시, 각기 다른 형태의 DTO를 반환하는 매우 다양한 쿼리들을 수행할 수 있다.

    - 읽기와 쓰기 작업에서 사용되는 데이터 표현들이 서로 일치 않는 경우, 일부 작업에서는 필요하지 않는 추가적이 컬럼이나 속성의 업데이트가 이뤄져야 한다.
    - 동일한 데이터 세트에 대해 병렬로 작업이 수행될 때 데이터 경합이 발생할 수 있다.
    - 정보 조회를 위해 요구되는 복잡한 쿼리로 인해 성능에 부정적인 영향을 줄 수 있다.
    - 하나의 데이터 모델이 읽기와 쓰기를 모두 수행 시, 보안 관리가 복잡해진다.

`CQRS`는 읽기와 쓰기를 각각 다른 모델로 분리한다. `명령(Command)`을 통해 데이터를 쓰고, `쿼리(Query)`를 통해 데이터를 읽는다.

    `Command(명령)`은 **수행할 작업 중심**이 되어야 한다. ex) '호텔 룸 예약', '호텔 룸 취소'
     또한 비동기적으로 큐에 쌓인 후 수행된다.

    `쿼리(Query)`는 데이터 베이스를 결코 수정하지 않으며, 어떠한 도메인 로직도 캡슐화하지 않은 DTO만을 반환한다.

`읽기/쓰기` 모델들을 서로 격리시키며 **어플리케이션 디자인과 구현을 더욱 간단한게 만들어준다.**
하지만 CQRS 코드는 ORM 툴을 통해 DB 스키마로부터 자동으로 생성되도록 할 수 없다는 단점이 있다. 

`읽기 DB`의 경우, 복잡한 조인문이나 ORM 매핑을 방지하기 위해 조회에 최적화된 별도의 DB 스키마를 가질 수 있다.

    - materialized view
        - 일반 뷰와는 달리 쿼리의 결과를 실제 테이블 형태로 물리적으로 저장하는 것, 쿼리의 결과를 DB 테이블 형태로 저장해 두고, 필요할 때마다 저장된 데이터를 빠르게 조회할 수 있도록 함. (REFRESH 명령문을 통해 주기적인 갱신이 필요함)

즉, 단지 다른 DB 스키마가 아니라 아예 다른 타입의 데이터 저장소를 사용할 수도 있다. 예를 들면 쓰기는 RDBMS를 사용하고, 읽기의 경우 몽고DB와 같은 NoSql을 사용하는 것이다.

별도의 읽기/쓰기 데이터 저장소가 사용된다면, 반드시 동기화가 이뤄져야 한다. 보통 이는 쓰기 모델이 DB에 수정사항이 발생할 때마다 이벤트를 발행함으로써 이뤄진다.
**이때 DB 업데이트와 이벤트 발행은 반드시 하나의 트랜잭션 안에서 이뤄져야한다.**

`CQRS`의 장점은 다음과 같다.
    
    - 읽기와 쓰기 각각에 대해 독립적으로 스케일링을 하는 것을 가능하게 해준다. (Lock 경합이 최소화가 된다.)
      Lock 경합은 데이터 수정인 상태에서 읽기 작업이 온다면 대기상태에 놓인다. 분리함으로써 더욱 최소화가 가능하다.

    - 읽기와 쓰기를 분리함으로 보안 관리가 용이해진다.
    - 읽기와 쓰기에 대한 관심사 분리로 시스템의 유지 보수가 더욱 쉬워 진다.
    - Materialized view를 통해 복잡한 조인문을 사용하지 않을 수 있다.


`CQRS`의 단점은 다음과 같다.

    - 읽기/쓰기 저장소가 분리된다면, 읽기 데이터가 최신이 데이터가 아닐 수도 있다. 읽기 저장소에는 쓰기 저장소의 변경 사항들이 반영되어야 하는데, 이에는 딜레이가 생긴다.
    
`CQRS의 이벤트 소싱`
이벤트소싱이란 모든 명령(Command)을 이벤트 형태로 별도의 이벤트 저장소에 저장한다. 이 명령들은 자연스럽게 다시 사용할 수 있습니다. 강한 일관성 및 실시간 업데이트가 필요한 시스템에는 적합하지 않을 수 있습니다.

## 3. SSH 터널링

SSH는 원격 호스트로 접속하기 위해 쓰이는 프로토콜, 그 중 SSH 터널링은 **SSH 연결을 통해서 방화벽을 우회, 앱 서버에 정보를 전달하고 데이터를 받을 수 있다. 일종의 프록시 역할을 한다.**

SSH 터널리을 이용하면 기존에 서비스 중이던 앱 서버를 재구동하지 않고도 방화벽을 우회 접근할 수 있다는 장점이 있다. 또, SSH 연결을 안전하게 암호화되기 때문에 앱 클라이언트와 서버가 암호화된 통신을 지원하지 않는 경우 SSH 터널링을 이용해서 암호화된 통신을 사용할 수 있다.

물론 방화벽을 우회하기 때문에 방화멱의 목적을 훼손하는 방법이며 각종 보안 홀을 야기할 수 있다.

- SSH 터널링 서버 구성

```shell

sudo vi /etc/ssh/sshd_config

#AllowTcpForarding, GatewayPorts 옵션 yes로 설정

sudo systemctl restart sshd 
# 혹은
sudo service sshd restart
```

SSH 터널링 종류

### SSH 터널링 종류

#### 1. Local Port Forwarding (로컬 포트 포워딩)

로컬 클라이언트(Host A)에서 원격 서버(Host B)로 특정 포트의 트래픽을 SSH 연결을 통해 전달하는 방법입니다. 이를 통해 Host A에서 직접 접근할 수 없는 포트를 우회하여 접속할 수 있습니다.

##### 사용법
# 로컬 포트 <local_port>에서 원격 서버 <remote_host>의 <remote_port>로 연결
ssh -L <local_port>:<remote_host>:<remote_port> <user>@<remote_host>

##### 예시
# 로컬 포트 1234에서 HOSTB의 3036 포트로 연결
ssh -L 1234:HOSTB:3036 user@hostB

- Host A에서 `localhost:1234`로 접속하면 Host B의 `3036` 포트에 연결됩니다.
- **유용한 경우**: Host B의 특정 포트(예: 3036)가 방화벽 등으로 막혀 있을 때.

---

#### 2. Remote Port Forwarding (원격 포트 포워딩)

원격 서버(Host B)의 특정 포트로 들어오는 트래픽을 SSH 연결을 통해 로컬 클라이언트(Host A)로 전달하는 방법입니다. 원격 서버가 외부에서 접근하기 어려운 경우 유용합니다.

##### 사용법
**원격 서버 <remote_host>의 <remote_port>로 들어오는 트래픽을 로컬 <local_port>로 전달**
ssh -R <remote_port>:localhost:<local_port> <user>@<remote_host>

##### 예시
**원격 서버의 1234 포트에서 로컬의 8080 포트로 트래픽 전달**
ssh -R 1234:localhost:8080 user@hostB

- Host B에서 `localhost:1234`로 접속하면 Host A의 `8080` 포트에 연결됩니다.
- **유용한 경우**: 외부에서 Host A의 포트(예: 8080)에 접근하고 싶을 때.

---

#### 3. Dynamic Port Forwarding (동적 포트 포워딩)

동적 포트 포워딩은 SSH 서버를 통해 다양한 목적지로의 트래픽을 전달하며, SSH 서버를 SOCKS 프록시로 사용할 수 있습니다. 이를 통해 여러 네트워크 요청을 유연하게 처리할 수 있습니다.

##### 사용법
**로컬 <local_socks_port>에서 SSH 서버를 통해 다양한 목적지로 트래픽을 전달 (SOCKS 프록시)**
ssh -D <local_socks_port> <user>@<remote_host>

##### 예시
**로컬 포트 1080에서 SSH 서버를 통해 트래픽을 SOCKS 프록시로 전달**
ssh -D 1080 user@hostB

- Host A에서 `localhost:1080`을 통해 SOCKS 프록시를 사용하여 외부 네트워크에 연결할 수 있습니다.
- **유용한 경우**: Host A가 Host B를 통해 여러 목적지로 트래픽을 라우팅해야 할 때.

---

각 포워딩 방식은 방화벽 우회, 보안 트래픽 전송, 또는 프록시 설정 등 다양한 상황에서 유용하게 사용할 수 있습니다.
