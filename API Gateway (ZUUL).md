# API Gateway (ZUUL)

## 정의 및 특징

- ZUUL은 Gateway 서비스 또는 Edge 서비스로서 Micro Service 라우팅, 모니터링, 예외처리, 보안 등을 책임진다.
- ZUUL은 Micro Service를 라우팅하는 과정에서 pre filter, route filter, post filter를 거치고 에러 발생 시, error filter를 거친다.
- 각 filter는 개발자가 자유롭게 커스터마이징이 가능하며 로깅, 인증, 모니터링 등 목적에 맞게 개발 할 수 있다.
- 여러 Service의 Port들을 JUUL이 실행되는 Port 하나로 통일해 하나의 서비스로 보이게 할 수 있다.
- 엄밀히 따지자면 ZUUL 또한 eureka client 중 하나이다.
- ![ZUUL Request Life Cycle](https://ssipflow.github.io/assets/images/static/180930/Request-Lifecycle.png)

## ZUUL 생성하기

1. 다음의 Dependency를 추가해 프로젝트를 생성한다.

   - Eureka Discovery Client
   - Actuator
   - ZUUL (spring boot 버전이 2.4 미만에서 추가할 수 있다,)

2. application.properties에 Eureka Client 및 ZUUL 관련 설정을 추가한다.

   - ```properties
     # 애플리케이션 포트 및 이름 설정
     erver.port=${PORT:9999}
     spring.application.name=ZUUL
     
     # ZUUL 관련
     zuul.ignored-services='*'
     zuul.ignored-patterns=/**/api/**
     
     #Spring Boot 2.0 부터는 actuator의 /actuator/routes uri을 사용하기 위해 management.endpoints.web.exposure.include=* 옵션을 사용해야 한다.
     management.endpoints.web.exposure.include=*
     
     # Eureka Client 설정 관련
     eureka.client.serviceUrl.defaultZone=http://localhost:8080/eureka/
     
     ```

   - Eureka Server에 등록된 서비스들의 정보를 ZUUL을 통해 엔드포인트에 접근하기 위해 ZUUL을 Eureka Client로 설정한다.

3. 해당 Project의 Application.java에  @EnableZuulProxy, @EnableEurekaClient 어노테이션을 추가한다

   - ```java
     @EnableZuulProxy
     @EnableEurekaClient
     @SpringBootApplication
     public class SpringCloudZuulApplication {
     
     	public static void main(String[] args) {
     		SpringApplication.run(SpringCloudZuulApplication.class, args);
     	}
     }
     ```

4. Eureka Server의 콘솔 창을 확인해 보면 ZUUL이 Eureka Client로 등록된 것을 확인할 수 있다.

5. ZUUL에서 /actuator/routes에 접속하면 (http://[ZUUL Project 주소]/actuator/routes) Eureka Server로 부터 캐시한 서비스들을 확인할 수 있다.

6. 이제 ZUUL을 통해 Service ID로 서비스 접근이 가능하다.

7. ZUUL을 통한 Client1의 주소는 (http://[ZUUL Project 주소]/[Client 이름]/[Client의 controller route])이다.

   - ex) : http://localhost:8081/client1/test 
     - 여기서 client1은 application 이름이 아닌 디렉토리명을 따온다.

