# 2024SummerTIL
## 2024 하계 모각소를 위한 레포지토리입니다.

### Week 1.SpringBoot - Swagger / Logback
##### Swagger
REST API를 설계, 빌드, 문서화 및 사용하는데 도움이 되는 오픈소스

##### Swagger 사용
1. 의존성 추가
2. 설정 코드 작성
```JAVA
@Configuration
@EnableSwagger2 //Swagger Ver.2를 활성화 시키는 어노테이션
public class SwaggerConfiguration{

	@Bean
	public Docket api(){ //Docket 객체 : Swagger 설정을 할 수 있게 도와주는 클래스 
		return new Docket(DocumentationType.SWAGGER_2)
							.apiInfo(apiInfo())
							.select()
		/*(1)*/		.apis(RequestHandlerSelectors.basePackage("com.springboot.api"))
							.paths(PathSelectors.any())
							.build();
	}
	
	private ApiInfo apiInfo(){
		
		return new ApiInfoBuilder()
								.title("Spring Boot Open API Test with Swagger")
								.description("설명 내용")
								.version("1.0.0")
								.build();
								// ApiInfoBuilder가 아닌 직접 ApiInfo 객체를 생성하여도 됨
								// ApiInfo는 정보 설정용 객체임
	}
}

//스캔될 메서드에 명세 정보 설정
@ApiOperation(value="메서드 이름", notes="메서드 설명")
@GetMapping(매핑 주소)
public String MappingController{
	/ * * @ApiParam
		*  
		*  매개변수에 대한 설명 및 설정을 위한 어노테이션
		*  매개변수 뿐 아니라 VO(DTO) 객체를 매개변수로 사용할 경우 
		*  DTO 클래스 내의 매개변수에도 정의 가능
		*/
	@ApiParam(value="이름" , required=true) @RequestParam String name,
	@ApiParam(value="이메일" , required=true) @RequestParam String email,
	@ApiParam(value="전화번호" , required=true) @RequestParam String phone,
}

```

##### Logback
- log4j 이후 출시된 로깅 프레임워크(slf4j 기반)
- spring-boot-starter-web 라이브러리 내부에 내장 되어 있음
- 특징
    1. 5개의 로그 레벨 설정(TRACE, DEBUG, INFO, WARN, ERROR)
    2. 운영환경과 개발 환경에 각자 다른 출력 레벨 설정 가능
    3. 설정 파일을 일정 시간마다 스캔
    ⇒ 애플리케이션 재가동하지 않아도 설정 변경 가능


### Week 2. 정보처리기사 문제풀이
CBT 모의고사 3회 풀이 진행

### Week 3. 부트캠프 내용 정리

##### Filter & Interceptor

**Interceptor**
- 컨트롤러로 들어오는 Request와 나가는 Response를 가로채는 역할
- InterCeptor의 간섭시점
  - View <--> Filter <--> DispatcherServlet <--> Interceptor <--> Controller
- 요청 전 (preHandle) : Contoroller 전 선처리 내용 정의
- 요청 후 (postHandle) : Response 의 오류 및 로깅 등 후처리 내용 정의
- 구현 방식
  - HandlerInterceptor 인터페이스의 구현체 생성
  - HandlerInterceptorAdapter 클래스를 상속


### Week 4.SpringBoot - Actuator
##### 액추에이터
SpringBoot는 Actuator를 통해 HTTP 엔드포인트 및 JMX를 활용하여 애플리케이션을 모니터링할 수 있는 기능을 제공

##### Spring Boot HealthCheck
- 서비스의 고가용성, 고성능을 위한 부하 분산등의 이유로 서버의 이중화를 할 경우 서버로 요청을 보내는 Load balancer를 둠
- Load Balancer에서 각 서버의 헬스체크 진행 후 해당 서버가 서비스 가능할 경우에 라우팅함
- 헬스 체크 시 각 코드레벨에서 Indicator의 사용
  1. HelathEndPointSupport 클래스의 getAggregateContribution 메서드 : 각 HealthContributor를 순회하면서 헬스 체크
  2. HealthEndpointSupport 클래스의 getCompsiteHealth 메서드 : 각 HealthIndicator로부터 수집한 상태를 바탕으로 현재 서버의 상태를 진단
  3. SimpleStatusAggregator클래스의 getAggregateStatus 메서드 : 각 상태를 수집하여 하나의 Status로 반환(이때 StatausComparator 사용)
      Status는 DOWN / OUT_OF_SERVICE / UP / UNKNOWN 의 순서로 가장 순서가 빠른 것 하나를 return


##### Dependency
```XML
<-- pom.xml-->
<dependencies>
	..생략..
	<dependency>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-actuator</artifactId>
	<dependency>
	..생략..
<dependencies>
```

##### 엔드포인트 리스트
1. /beans : 애플리케이션에 있는 모든 스프링 빈 리스트를 표시
2. coditions : 자동 구성 조건 내역을 생성함
3. configprops : @ConfigurationProperties의 속성 리스트를 표시함
4. env : 애플리케이션에서 사용할 수 있는 환경 속성을 표시
5. health : 애플리케이션의 상태 정보 표시
6. httpexchanges : Http 호출 응답 정보를 보여줌. HttpExchangeRepository를 구현한 빈을 별도로 등록해야 함
7. info : 애플리케이션의 정보 표시
8. loggers : 애플리케이션의 로거 설정을 표시하고 수정
9. metrics : 애플리케이션의 매트릭 정보 표시
10. mappings : 모든 @RequestMapping의 매핑 정보를 표시
11. threaddump : 스레드 덤프를 수행
12. shutdown : 애플리케이션을 정상적으로 종료(default값은 비활성화 상태)

##### 엔드포인트 노출 여부
- application.properties
```
<!--각 HTTP/JMX 에서 엔드포인트를 전체적으로 노출, 스레드 덤프와 힙덤프는 제외-->
<!-- HTTP 설정 -->
management.endpoints.web.exposure.include=*
management.endpoints.web.exposure.exclude=threaddump,heapdump

<!-- JMX 설정 -->
management.endpoints.jmx.exposure.include=*
management.endpoints.jmx.exposure.exclude=threaddump,heapdump
```

##### Actuator 커스텀
- 커스텀은 두 가지 방식으로 진행
1. 기존 기능에 내용 추가
    -> 별도의 구현체 클래스 작성하여 내용을 추가(InfoContributor Interface)
3. 새로운 엔드포인트 개발
    -> @Endpoint어노테이션을 통해 새로운 어노테이션을 빈에 등록




### Week.5.SpringBoot - RestTemplate & WebClient
##### RestTemplate
- HTTP 통신 기능 사용을 위한 Spring의 Template
- 기본적으로 동기 방식으로 처리, 비동기 방식을 사용하고 싶다면 AsyncRestTemplate 사용
- ** Rest Template는 deprecated(지원 중단)되었음**
- **특징**
  - spring-boot-starter-web 모듈에 포함되어 따로 의존성 추가할 필요 없음
  - HTTP 메서드에 맞는 여러 메서드 제공
  - RESTful 형식을 갖춘 템플릿
  - Blocking I/O 기반의 동기 방식 사용
- **동작 원리**
  - 애플리케이션에서 URI,HTTP Method등 설정 후 API로 RequestEntity 보냄
  - RestTemplate에서 HttpMessageConverter를 통해 Request 메세지로 변환
  - 변환된 메세지를 ClientHttpRequestFactory를 통해 ClientHttpRequest로 가져온 후 외부 API에 요청을 보냄
  - 요청에 따른 응답은 ResponseErrorHandler에서 오류 확인 후 오류가 잇다면 ClientHttpResponse에서, 없다면 HttpMessageConverter 에서 처리됨
- **커스텀**
  - RestTemplate은 HTTPClient를 추상화하고 있으며 Connection Pool은 지원하지 않음
    -> 호출마다 포트를 열어 커넥션을 생성하며 TIME_WAIT 된 소켓을 재사용하지 못함
    -> 따라서 Apache의 HTTPClient로 커스텀하여 사용하는 것이 보편적임
  


##### WebClient
- deprecated(지원 중단)된 Rest Template을 대체하기 위한 SpringBoot의 HTTP 통신 인터페이스
- WebClient 생성 방법은 두가지(WebClient는 객체 생성 후 요청 전달하는 방식으로 동작)
  - create 메서드 사용
    ```
    WebClient wc = webClient.create();
    ```
  - builder()를 이용한 생성
    ```
    WebClient wc = WebClient.builder()
		    .baseUrl("http://localhost:8081/actuator")
		    .defaultHeader(HttpHeaders.CONTENT_TYPE, MediaType.APPLICATION_JSON_VALUE)
		    .build();
    ```


### 출처
"스프링부트 핵심 가이드", 장정우, 위키북스, 2022
[Spring Boot Actuator의 헬스 체크 작동 원리](https://toss.tech/article/how-to-work-health-check-in-spring-boot-actuator) - 토스 기술 블로그 (접근 날짜: 2024년 8월 06일).

