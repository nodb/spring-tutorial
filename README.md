# 1차시
## spring-tutorial를 완료하자!
![Image](https://github-production-user-asset-6210df.s3.amazonaws.com/71473708/437928886-8aa784ef-eaa1-4f7d-aba1-3e6130f0139a.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAVCODYLSA53PQK4ZA%2F20250427%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20250427T071921Z&X-Amz-Expires=300&X-Amz-Signature=547a369bf4158ee99e75a981abe84f888e0c9e008f9f3fbff52ca634fccfc6eb&X-Amz-SignedHeaders=host)

## spring이 지원하는 기술들(IoC/DI, AOP, PSA 등)을 자유롭게 조사해요 & 스프링 어노테이션을 심층 분석해요
**프레임워크(Framework)** : **비지니스 로직 X**, 뼈대만 갖춰진 반제품 형태의 애플리케이션

- 비지니스 로직과 무관한 귀찮고 어려우며 **모듈화** 할 수 있는 일들.
- 개발자 : 비지니스 로직 집중

**Spring Framework** : **자바 애플리케이션 개발**을 위한 경량 framework

- Spring
    - DB 관리
    - 선언적 Tx 처리
    - 템플릿 제공
    - 범용 예외 처리

- 개발자
    - 비즈니스 로직
- **IoC**(Inversion Of Control; 제어의 역전) : 생성/소멸/의존성 관리 : 사용자 코드→컨테이너(프레임워크)
- **Spring IoC 컨테이너**
    - **빈 관리**
        - **빈(Bean)** : 스프링 프레임워크에 의해 생성되고 관리되는 자바 객체
    - **의존성 주입**(DI)
        - 객체 간의 결합도 ↓
        - 코드 재사용성, 유지보수성 ↑
    - **라이프 사이클 관리**
        - 빈의 초기화와 소멸 시점에 콜백 메서드를 호출
    - **설정 관리**
        - 애플리케이션 설정 관리
        - XML, Java 설정 클래스, 어노테이션 기반 설정 지원
- **POJO** 기반의 **DI**, **AOP**, **PSA**
    - **POJO(Plain Old Java Object)** : 자바 객체
---
### **DI(Dependency Injection; 의존성 주입)**

- 프레임워크가 객체 관리를 컨트롤
- **의존성**(멤버 변수)를 **외부에서 주입**
- 장점
    - 의존하는 객체는 변경될 필요가 없음
    - 코드 유지보수성 ↑
- 방법
  - **생성자 주입**
    
    ```java
    public class WasherUser {
    	private final Washer washer; ****// has a 관계 - 의존성
    
    	public WasherUser(Washer washer){ // 생성자의 인자로 washer를 받는다
    		this.washer = washer;
    	}
    }
    
    // 외부에서 의존성 주입
    Washer washer = new SWasher();
    WasherUser washerUser = new WasherUser(washer); // 생성자 주입
    ```
    
    - WasherUser 입장에서는 어떤 Washer을 사용할지 고민 X → 책임을 외부에 위임
    - WasherUser 클래스와 Washer 클래스의 결합 느슨해짐
    - 의존성 주입 코드가 명시적
    - **빈의 순환 의존성 문제** → 빈 생성 시점에 즉시 발견 가능
        - lombok **@RequiredArgsConstructor** 활용
    
  - **setter 주입**
    
    ```java
    public class WasherUser {
    	private Washer washer; ****// has a 관계 - 의존
    	
    	@Autowired
    	public void setWasher(Washer washer) {
    		this.washer = washer;
    	}
    }
    
    // 외부에서 의존성 주입
    Washer washer = new SWasher();
    WasherUser washerUser = new WasherUser();
    washerUser.setWasher(washer); // setter 주입
    ```
    
    - 선택적 의존성 주입 가능
    - 주입 누락시 오류 발생
    
  - **field 주입**
    
    ```java
    public class WasherUser {
        @Autowired
        private Washer washer;  // 리플렉션으로 바로 주입
    }
    ```
    
    - 비추
    - 리플렉션을 써야만 의존성 주입 가능
    - 캡슐화 위반

- 스프링의 빈 관리 라이프사이클
    1. 개발자는 POJO로 빈 구성
    2. 메타 정보(빈의 생성 방법 및 관계 설정 정보)를 스프링 컨테이너에게 전달
    3. 런타임에
    스프링은 메타 정보를 보고 빈 객체 생성 → 싱글턴 형태로 관리
    빈 관계에 정보에 따라 빈 주입 처리

- **명시적 DI**
    - 빈을 생성하고 의존성을 주입하는 코드를 별도의 파일에 명시

| 어노테이션 | 내용 |
| --- | --- |
| @Configuration | java 기반으로 설정 파일 만들기(설정 정보)<br>class 레벨 |
| @Bean | 빈 선언, 빈의 이름 : method 이름(name 속성으로 재정의)<br>method 레벨 |

```java
@Configuration // 설정 정보
public class WasherConfig {
	@Bean // 메서드를 통해서 빈 생성
	public SWasher sWasher(){ // SWasher 타입의 빈 생성, 빈 이름: SWasher
		return new SWasher(); // 실제로 사용될 빈 객체 반환
	}
	
	@Bean (name = "myWasheruser") // 빈 이름을 myWasheruser로 설정
	public WasherUser washerUser(){
		return new WasherUser(sWasher()); // WasherUser 생성자에 SWasher 주입
	}
}
```

- 빈 조회

```java
Washer swasher = ctx.getBean(SWasher.class); // SWasher 타입의 빈
Washer myWasher = ctx.getBean("sWasher", Washer.class); // 동일 타입이 여럿일 경우
																												// 타입 + 이름 기반
System.out.println(sWasher == myWasher);
```

- **묵시적 DI**
    - @Configuration, @Bean을 사용 X

| 어노테이션 | 내용 |
| --- | --- |
| @Autowired | 빈을 주입<br>타입 기반으로 빈 자동 주입<br>생성자가 1개일 경우 생략가능 |
| @Qualifier | 이름 기반으로 주입될 빈을 한정<br>@Autowired와 함께 사용 |
| @ComponentScan | @Conponent 표시된 빈을 등록 |
| @Component | 빈으로 사용할 각각의 클래스들에 표시<br>빈의 이름 : Pascal case인 경우 → camel case로 변경됨(value 속성으로 재정의) |
| @Repository | 스테레오타입, 내부적으로 @Component를 포함<br>MVC Repository(DAO) 계열 |
| @Service | 스테레오타입, 내부적으로 @Component를 포함<br>MVC Service 계열 |
| @Controller | 스테레오타입, 내부적으로 @Component를 포함<br>MVC controller 계열 |
| @Configuration | 스테레오타입, 내부적으로 @Component를 포함<br>java 기반의 메타정보 클래스 |

```java
@Component
public class Bag {
	Hat hat;
	Watch watch;
	SmartWatch smartWatch;

	@Autowired // 생성자 주입 - 생략 가능
	public Bag(Hat hat, @Qualifier("watch") Watch watch) 
}

@Component
public class Hat {}	

@Configuration
@ComponentScan("com.ssafy.live.accessory")
public class BagConfig {}
```

**Spring Boot** : 스프링 애플리케이션 개발 템플릿(**설정 자동화**, **단위 테스트 강화**)

- application.properties (혹은 .yml)
    - `property = value`
    - logging.level.* : 패키지별 로그 레벨 지정
    - server.port : 서버의 HTTP 포트
    - server.servlet.context-path : 서버의 context root path

| 어노테이션 | 내용 |
| --- | --- |
| @Value | application.properties의 value 가져오기<br>@Value("${property_name}") |
| @SpringBootApplication | SpringBoot 애플리케이션의 시작점인 클래스에 선언<br> **@SpringBootConfiguration** <br> **@EnableAutoConfiguration** <br> **@ComponentScan** |
| @SpringBootConfiguration | 빈 등록<br>@Configuration |
| @EnableAutoConfiguration | 의존성에 근거해 자동 설정 지원 |
| @SpringBootTest | 스프링 애플리케이션의 통합 테스트<br>통합 테스트는 애플리케이션 전체를 구동 → 단위 테스트에는 부적합 |
| @Scope | 빈의 스코프<br>묵시적 빈 등록시 : 클래스 레벨<br>명시적 빈 등록시 : 메서드 레벨<br>singleton : 디폴트값, 비지니스 로직을 재사용하기 위해 빈을 관리<br>prototype : 요청 할 때 마다 매번 새로운 빈 객체 생성<br>Request, Session : 웹 환경에서 사용 |

**lombok**

| 어노테이션 | 내용 |
| --- | --- |
| @Getter @Setter |  |
| @ToString |  |
| @EqualsAndHashCode |  |
| @AllArgsConstructor | 모든 멤버 변수를 파라미터로 하는 생성자 생성 |
| @NoArgsConstructor | 기본 생성자 생성 |
| @RequiredArgsConstructor | final 및 @NonNull 멤버 변수를 이용한 생성자 생성 |
| @Data | @ToString : 순환 참조 문제 발생 가능 <br> - @ToString.**Exclude** 해결 <br> @EqualsAndHashCode <br> @Getter <br> @Setter <br> @RequiredArgsConstructor : 다른 생성자가 없을 때만 |
| @NonNull |  |
| @Builder | 빌더 패턴을 통한 객체 생성 지원<br>내부적으로 모든 field의 값을 default 초기화<br>@Builder. Default : 기본값을 유지하기 위해서 활용 |
| @Slf4j | Slf4j의 Logger 타입 변수 log 생성 |
---
### **AOP(Aspect Oriented Programming; 관점 지향 프로그래밍)**

- 메서드 : 핵심 관심사항(비지니스 로직)과 횡단 관심 사항(부가 로직)
    - 횡단 관심 사항 : 로깅, 성능, 보안, 암호화, 일괄 예외 처리, 트랜잭션 처리
    - 횡단 관심의 모듈화 → 재사용
    - 개발자 : 핵심 관심사에 집중
- AOP 동작 원리
    - **Proxy** : 대신 서버에 요청
    - CGLib(Code Generator Library)를 이용해 Class 기반 Proxy 생성
    - 빈 설정 분석
    → 빈 생성(**target** 반환)
    → pointcut 기반 AOP 적용 검토
    → Proxy 생성
    → 노출 대상 변경(**proxy** 반환)
- target : 핵심 비지니스 로직을 포함한 빈 객체, AOP 적용 대상
- join point(어디서) : advice가 동작하는 지점
- **pointcut** : advice를 적용할 대상 결정하는 **표현식**,  join point 필터링
    - return_type, parameter에 스칼라 타입을 제외 레퍼런스 타입 사용시 패키지 이름까지 사용
        - @Before("execution(public java.util.List<com.example.MyClass> *(..))")
    - 여러 개의 pointcut을 연결 : &&, II, !, and, or, not
        - @Before("execution(* com..*(..)) || execution(* org..*(..))")

| 어노테이션 | 내용 |
| --- | --- |
| execution | signature 기반, 정교<br>@Before("execution( * com.ssafy..dao.*.*(..))")<br>@Before("execution(int pointcut..Bean.set(..))")<br>**return_type**(필수) : 메서드 리턴타입, *(타입 무관), !(타입 부정)<br>class : 패키지를 포함하는 클래스 이름, ..(0개 이상 하위 패키지, 맨 처음 불가), *(0개 이상 문자열)<br>**method_name**(필수) : 메서드 이름, *(0개 이상 문자열)<br>parameter : 메서드 파라미터, *(1개의 파라미터 대체), ..(0개 이상 파라미터 대체) |
| within | 빈 클래스 기반, execution의 class 부분<br>@Before("within(com.ssafy..dao)") |
| bean | 빈의 이름을 기반<br>@Before("bean(basicMemberDao)") |
| @within | 빈에 적용된 애너테이션을 기준<br>@Before("@within(org.springframework.stereotype.Service)") |
| @annotation | 특정 애너테이션이 선언된 메서드를 기준<br>@Before("@annotation(org.springframework.transaction.annotation.Transactional)") |

- **advice**(언제, 무엇을) : 횡단 관심 코드를 모듈화 한 메서드

| 어노테이션 | 내용 |
| --- | --- |
| @Before | target method 호출 전에 advice 실행<br>전달받은 argument 조작 가능 |
| @After | target method가 종료되면 언제나 실행 |
| @AfterReturning | target method가 정상적으로 종료한 경우만 실행<br>반환 받은 return 값 조작 가능 |
| @AfterThrowing | target method에서 예외가 던져졌을 때만 실행 |
| @Around | advice 내부에서 target의 메서드 호출 → target method의 모든 것을 제어 가능<br>파라미터, 리턴 값에 대한 완전한 대체 가능<br>proceed : 타겟의 메서드 호출 |

- **aspect** : 모듈화 된 횡단 관심사(관점)들에 대한 추상적인 명칭, **하나 이상**의 advice

| 어노테이션 | 내용 |
| --- | --- |
| @Aspect | 하나 이상의 advice를 포함 |

- weaving : aspect 비즈니스 로직 코드에 적용하는 행위

```java
@Component // 하나의 빈으로 관리
@Aspect // 하나 이상의 advice를 포함
@Slf4j
public class LoggingAspect {
	@Before("execution( * com.ssafy..dao..*(..))") // pointcut
	public void loggingDao(JoinPoint jp) {
		log.trace("method call: {}, {}", jp.getSignature(), Arrays.toString(jp.getArgs()));
	}
}
```

- advice type: 메서드 실행 전
- advice code: 횡단 관심사
---
### **PSA(Portable Service Abstraction; 이식 가능한 서비스 추상화)**

- JPA를 쓰건 MyBatis를 쓰건 스프링에서 T.X 처리하는 방법은 동일!
- 추상화된 레이어 → 특정 환경에 종속되지 않고 쉽게 사용

## Spring Bean이 무엇이고, Bean의 라이프사이클은 어떻게 되는지 조사해요
**Spring MVC**

- MVC 기반 Web Application을 작성하기 위한 Spring Framework
- **DispatcherServlet**
    - Front Controller Pattern의 적용으로 client의 모든 request 접수
    - **Infrastructure Component**에게 위임하여 처리
        - **HandlerMapping** : 이런 request → 어떤 Handler가 처리?
        - **HandlerAdapter** : XXHandler에게 request 좀 처리~
        - **ViewResolver** : 이런 뷰가 필요~

![image](https://github.com/user-attachments/assets/7d824db2-cc13-48ca-9925-1d9f83a33a7b)

| 어노테이션 | 내용 |
| --- | --- |
| @Controller |  |
| @RequestMapping | 클래스 레벨 : prefix<br>메서드 레벨 : 해당 메서드<br>value :URL<br>method : GET/POST |
| @GetMapping | @RequestMapping(method=RequestMethod.GET) 축약 |
| @PostMapping | @RequestMapping(method=RequestMethod.POST) 축약 |

- **View Resolver** 설정

```java
@Bean
public ViewResolver internalResourceViewResolver() {
	InternalResourceViewResolver resolver = new InternalResourceViewResolver();
	resolver.setPrefix("/WEB-INF/views/");
	resolver.setSuffix(".jsp");
	return resolver;
}
```

```xml
// application.properties
spring.mvc.view.prefix=/WEB-INF/views/
spring.mvc.view.suffix=.jsp
```

- 처리
    - **forward**: logical view name 전달
    - **redirect**: **redirect:** 를 추가
    - **json 방식** 응답: **@ResponseBody** 사용

```java
@RequestMapping("/")
public String welcome (Model model) }
	model.addAttribute("message", service.sayHello());
	return **"index"**; // **forward**
}

@RequestMapping("/redir")
public String redir(Model model) {
	return **"redirect:/"**; // **redirect**
}

@GetMapping("/json")
public **@ResponseBody** Map<String, Object> json() {
	return **Map.of**("name", "hong gil dong","age", 10);
}
```

| 어노테이션 | 내용 |
| --- | --- |
| @RequestParam | HttpServletRequest 객체의 parameter 조회에 사용<br>name 속성에 파라미터 이름 지정 |
| @ModelAttribute | 전달된 파라미터들을 DTO의 property 이름 기반으로 setter와 자동 연결해서 DTO 생성 |
| @CookieValue | cookie의 name 값에 매핑된 value를 얻어올 때 |

**Flash Scope**

- Redirect 가 완료될 때까지만 저장되는 Spring 지원 scope
- request < flash < session
- 한 번의 request 후 사라지는 session scope

**Handler Interceptor**

- DispatcherServlet이 HandlerAdapter를 호출하기 전/후 등에 거치는 일종의 필터

```java
boolean preHandle() // true이면 다음 단계로 진행
void postHandle()
void afterCompletion()
```

- 하나의 controller에 여러 개의 interceptor가 등록될 수 있음
    - preHandle은 등록된 순서로 호출
    - postHandle과 afterCompletion은 역순으로 호출 됨

![image 1](https://github.com/user-attachments/assets/a9001d11-70fa-4fc8-89d8-a00e6ad2352e)

- HandlerInterceptor 적용
    - ANT 경로 패턴
        - ? : 1개의 글자
        - * : 0개 이상의 글자
        - ** : 0개 이상의 디렉터리

```java
@Override
public void addInterceptors(InterceptorRegistry registry) {
	registry.addInterceptor(pi)
					.addPathPatterns("/some", "/check/**")
					.excludePathPatterns("/check/*Help");
}
```

**Filter, Interceptor, AOP 차이**

- **Servlet Filter**
    - spring과 무관하며 DispatcherServlet 호출 이전에 동작
    - 스프링과의 협업은 어렵고 단지 ServletRequest, ServletResponse에 대한 전/후 처리
- **HandlerInterceptor**
    - Spring에서 관리하며 DispatcherServlet 이후 동작
    - Spring의 빈으로 injection 등 모든 기능 사용 가능
    - c**ontroller에 대한 부가 작업이 필요한 경우**
- **AOP**
    - HTTP 관련 작업을 처리하기 위한 처리하기 어려움
    - 일반적으로 **Service, DAO에 적합**

## 단위 테스트와 통합 테스트 탐구
**Slf4j(Simple Logging Façade for Java)** : java에서 logging을 위한 façade

- 로그 레벨 : **trace < debug < info < warn < error**
    - 예) 개발 과정 : trace → trace, debug, info, warn, error 표시
    - 예) 운영 과정 : info → info, warn, error 만 표시
- 레이아웃 설정 가능
- 의존성 : logback을 slf4j 기본 구현체로 사용

```xml
<dependency>
	<groupId>**ch.qos.logback**</groupId>
	<artifactId>logback-classic</artifactId>
	<version>1.5.16</version>
	<scope>compile</scope>
</dependency>
```

- logger 사용

```java
private static final Logger log = LoggerFactory.getLogger(LoggerTest.class);
public class LoggerTest {
	public static void main(String[] args) {
		log.trace("trace: {}", "trace level")
		log.debug("debug: {}", "debug level");
		log.info("info: {}", "info level");
		log.warn("warn: {}", "warn level");
		log.error("error: {}", "error level");
	}
}
```

- 로그 설정에서 레벨 올리기, logback.xml

```xml
<configuration>
  <!-- 전체 로거 레벨을 TRACE로 설정 -->
  <root level="TRACE">
    <appender-ref ref="STDOUT"/>
  </root>
</configuration>
```

**junit** : Java 코드의 단위 테스트 자동화를 위한 프레임워크

- 의존성

```xml
<dependency>
	<groupId>org.junit.jupiter</groupId>
	<artifactId>junit-jupiter-api</artifactId>
	<version>5.12.0</version>
	<scope>**test**</scope> **<!-- Test 영역에서만 사용 -->**
</dependency>
```
| 어노테이션 | 내용 |
| --- | --- |
| @Test | 대상 메서드가 단위테스트를 수행하는 테스트 케이스로 인식 |
| @DisplayName  | 테스트 리포트나 IDE 실행 화면에 표시될 내용(테스트의 이름) |
| @BeforeAll<br>@AfterAll | 테스트 클래스 내에 선언된 모든 테스트 메서드 @Test보다 단 한 번 가장 먼저/나중에 실행<br>모든 @Test에 거쳐 한번 만 필요한 사전/사후 작업 메서드 정의<br>static 메서드에 작성 |
| @BeforeEach<br>@AfterEach | 각각의 @Test 전/후에 공통적으로 처리할 사전/사후 작업 메서드 정의<br>각 @Test 메서드가 실행되기 직전에 한 번씩 호출 |

**assertion(단정문)** : 메서드 호출 결과가 예상한 값과 동일한지 판단

- 긍정과 부정이 쌍으로 존재

| 어노테이션 | 내용 |
| --- | --- |
| assertEquals(a, b)<br>assertNotEquals(a, b) | a, b 값이 같은지/다른지<br>참조비교(==) |
| assertSame(a, b)<br>assertNotSame(a, b) | a, b 주소가 같은지/다른지 |
| assertTrue(a)<br>assertFalse(a) | a가 true/false 인지 |
| assertNull(a)<br>assertNotNull(a) | a가 null/not null 인지 |
| assertArrayEquals(a, b) | a, b 배열 요소별로 비교 |
| assertThrows | 지정한 예외가 발생하는지 |
| assertAll | 여러 Assertion을 묶어 실행<br>앞의 실패 여부와 상관없이 전부 실행 |

```java
String str1 = new String("Hello");
String str2 = new String("Hello");
assertEquals(str1, str2); // 통과: str1.equals(str2)가 true
assertNotSame(str1, str2); // 통과: str1과 str2는 서로 다른 인스턴스

// 배열
char[] chars1 = "HelloJUnit".toCharArray();
char[] chars2 = "HelloJUnit".toCharArray();
assertNotEquals(chars1, chars2); // 통과: 서로 다른 new로 생성된 별개의 배열
assertArrayEquals(chars1, chars2); // 통과: 원소 내용이 하나하나 같음

// target.divide(10, 0) 실행 중 ArithmeticException 발생해야 통과
ArithmeticException e = assertThrows(
	ArithmeticException.class,
	() -> target.divide(10, 0)
);
assertEquals(e.getMessage(), "/ by zero"); // 통과

// 한 번에
assertAll(() -> { assertEquals(target.divide(10, 5), 100); // 실패: 2 == 100
			 }, () -> { assertNotEquals(target.divide(10, 5), 2); // 실패: 2 != 2
			 }, () -> { assertNotNull(target); // 성공
			 }, () -> { assertTrue(target.divide(10, 5) == 2); // 성공
});
```

Given-When-Then 패턴 : BDD(Behavior Driven Development)

- given: 테스트 준비
- when: 테스트 메서드 실행
- then: 테스트 결과

```java
@Test
public void seccess() {
	Calculator calc = new Calculator();
	assertEquals(calc.divide(10, 5), 10/5);
}
```

```java
@Test
public void seccess() {
	// given
	Calculator calc = new Calculator();
	int a = 10;
	int b = 5;
	// when
	int result = calc.divide(a, b);
	// then
	assertEquals(result, 10/5);
}
```

@Controller 클래스 단위 테스트

- **Mock 환경**: 실제 WAS에서 Spring@MVC와 소통하는 부분을 부분을 **가상**으로 제공

```java
@WebMvcTest(value = SimpleController.class) // 테스트 할 대상 Controller
@Slf4j
public class SimpleControllerTest {
	@Autowired
	MockMvc mock; // 가상의 웹 환경을 위한 MockMvc 객체
	
	@MockitoBean // 실제 서비스를 테스트할 계획은 없다.
	SimpleService sService;

	@Test
	public void forwardTest() throws Exception {
		// given
		when(sService.helloMVC()).thenReturn("Hello"); // Mock 서비스 객체의 동작 흉내내기
		// when
		...
	}
}
```

F.I.R.S.T 원칙

- **Fast**: 단위테스트는 빠르게 동작
- **Independent**: 테스트는 이전 테스트 상태에 의존 X
    - 어떤 순서로 테스트를 하더라도 성공
- **Repeatable**: 여러 번 테스트를 해도 같은 결과
- **Self-Validating**: 테스트 자체만으로 검증 완료
- **Timely**: TDD를 한다면 테스트는 production code 개발 전 진행

