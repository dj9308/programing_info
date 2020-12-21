# Eureka(Spring Cloud)

## 정의 및 특징

- 클라이언트가 요청(request)했을 때, 동적으로 IP가 변경되는 Cloud 환경에서 정확한 IP위치를 찾아내 응답(response)해주는 서비스.
- Eureka Server와 Eureka Client로 나뉜다.

## Eureka Server

- 등록된 정보는 일반적으로 서비스 ID와 URL이 포함된다. 이렇게 Server registry에 등록된 서비스는 서비스 ID를 통해 접근할 수 있다.
- Eureka Client에 해당하는 Micro Services의 상태 정보가 등록되어 있는 registry를 갖는다.
- Middle-tier server(비즈니스 로직이 위치한 애플리케이션 서버단)의 로드밸런스와 Failover를 위해 서비스를 배치해 주는 REST 기반 서비스이다.
- Spring Boot에서 지원하는 프로젝트이며 주로 AWS Cloud에서 사용되고, 이를 Eureka Server라 부른다.

## Eureka Client

- 유레카 클라이언트는 기본적으로 JAVA Application이다. 
- Non-Java application은 Side-car로 유레카 서비스에 등록해 Polyglot을 지원한다.
- 유레카 클라이언트로 등록되면 유레카 서버에서는 30초 간격으로 ping을 요청해 등록된 서비스의 health를 체크한다. 이 ping 요청이 전송되지 않으면 이 서비스는 죽은 것으로 간주해 레지스트리에서 제외된다.
- 유레카 클라이언트는 서버로부터 레지스트리 정보를 읽어와 로컬에 캐시한다.
- 이 클라이언트는 로컬에 캐시된 레지스트리 정보를 이용해 필요한 다른 서비스를 찾을 수 있게 된다.
- 이 정보는 기본적으로 30초마다 주기적으로 갱신되며, 최근에 가져온 정보와 현재 레지스트리 정보의 차이를 가져오는 방식으로 갱신된다.

## Eureka Server 설치

1. Eureka Server dependency가 추가된 Spring Boot Project를 생성한다.

2. application.properites 파일에 eureka 서버 관련 설정을 한다.

   - ```properties
     ## properties 파일일 경우
     # application settings
     spring.application.name=Eureka Server
     server.port=${PORT:8080}
     
     # Eureka Configuration
     eureka.client.serviceUrl.defaultZone=http://localhost:8080/eureka/
     eureka.client.register-with-eureka=false
     eureka.client.fetch-registry=false
     eureka.server.enable-self-preservation=true
     
     
     ## yml 파일일 경우 (properties 파일 또는 yml 파일 둘 중 하나만 설정하면 됨).
     # application settings
     spring:
       application:
         name: Eureka Server
     server:
       port: 8080
     
     # -- Eureka
     eureka:
       instance:
         hostname: localhost
       client:
         serviceUrl:
           defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/
         register-with-eureka: false
         fetch-registry: false
     ```

3. 해당 프로젝트의 Application.java 파일에 @EnableEurekaServer 어노테이션을 추가한다.

   - ```java
     import org.springframework.boot.SpringApplication;
     import org.springframework.boot.autoconfigure.SpringBootApplication;
     import org.springframework.cloud.netflix.eureka.server.EnableEurekaServer;
     
     @EnableEurekaServer
     @SpringBootApplication
     public class EurekaServerApplication {
     
     	public static void main(String[] args) {
     		SpringApplication.run(EurekaServerApplication.class, args);
     	}
     }
     ```

4. Project를 실행하면 해당 웹 콘솔을 확인할 수 있다.

![Console](https://ssipflow.github.io/assets/images/static/180926/Eureka_server_console.png)

## Eureka Client 설치

1. 다음 Dependency로 프로젝트를 생성한다.

   - Eureka Discovery Client
   - Actuator
   - Spring Web 
   - DevTools

2. application.properties에 eureka client 관련 설정을 추가한다.

   ```properties
   ################## Application Settings ####################
   
   ##### 여기서 명시한 어플리케이션 이름이 유레카 서버에 Service ID로 등록된다.
   ## properties 파일
   # 당연히 server와 port는 달라야 한다.
   spring.application.name=Client1
   server.port=${PORT:8081}
   
   ##### Eureka Server의 url이 들어간다.
   eureka.client.serviceUrl.defaultZone=http://localhost:8080/eureka/
   
   ## yml 파일 (properties 파일 과 yml 파일 중 하나만 설정하면 됨)
   # -- server port
   server:
     port: 8081
   
   # -- Default spring configuration & Thymeleaf
   spring:
     thymeleaf:
       enabled: true
       encoding: UTF-8
       check-template-location: true
       prefix: classpath:/templates/
       suffix: .html
     application:  
       name: Client1
   
   # -- Eureka client
   eureka:
     client:
       serviceUrl:
         defaultZone: ${EUREKA_URL:http://localhost:8080/eureka/}
   ```

3. controller 및 view를 새로 만든다.(Thymeleaf 템플릿 엔진 이용)

   - ```java
     @Controller
     public class SampleController {
     
         @RequestMapping(value = "/test", method = RequestMethod.GET)
         ResponseEntity<Map<String, Object>> sample() {
             ResponseEntity<Map<String, Object>> response = null;
     
             Map<String, Object> resMap = new HashMap<String, Object>();
             resMap.put("type", "First eureka client!");
             resMap.put("message", 1);
     
             response = new ResponseEntity<Map<String, Object>>(resMap, HttpStatus.OK);
     
             return response;
         }
     
         @RequestMapping(value = "/test2", method = RequestMethod.GET)
         public String test2(Model model, HttpSession session){
             session.setAttribute("id", "seol");
             String id = (String) session.getAttribute("id");
             model.addAttribute("id", id);
             model.addAttribute("test", "Hello World!");
             return "index";
         }
     }
     ```

   - ```html
     <!DOCTYPE html>
     <html xmlns="http://www.w3.org/1999/xhtml"
     	xmlns:th="http://www.thymeleaf.org"
     	xmlns:layout="http://www.ultraq.net.nz/thymeleaf/layout"
     	layout:decorate="~{layout/base}">
     <head>
       <title>test</title>
     </head>
     <body>
         <h1 th:text="${test}"></h1>
         <h2 th:text="${id}"></h2>
     </body>
     </html>
     ```

4. Project 실행 후 Eureka Server의 콘솔 창을 확인해보면 해당 Eureka Client의 Service가 등록된 걸 확인할 수 있다.

   - ![Eureka Server Registered Console](https://ssipflow.github.io/assets/images/static/180926/Eureka_server_registered_console.png)