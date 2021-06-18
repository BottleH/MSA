# Spring Gateway


### 1. API Gateway 소개

___

- 다수의 서비스로 구성된 MicroService 들에서 각 서비스들의 IP와 PORT 번호들에 대한 단일화된 엔드포인트 제공
- 각 서비스들에서 필요한 인증/인가, 사용량 제어, 요청/응답 변조 등의 기능을 대신 담당

![Spring_Gateway](/Img/Spring_Gateway.png)



### 2. Spring Gateway 기능

___

- Service Request 라우팅 기능
- Service Load Balancing
- Service Request에 대한 Endpoint 단일화 기능
- Service Mesh와 연계를 통한 장애 대응 기능
- Service Filtering 기능
- Authentication/ Authorizing 기능
- Logging/Monitoring 기능 제공



### 3. Spring Gateway 적용

___

1. dependency 추가 
   - `implementation 'org.springframework.cloud:spring-cloud-starter-gateway’` // spring-boot-starter는 추가 할 필요 없음 `compile('org.springframework.cloud:spring-cloud-starter-netflix-eureka-client’)` // Spring Gateway가Eureka에 등록할 수 있도록 eureka-client 추가
2. Application Main에 `@EnableEurekaClient` 추가
3. eureka 관련 설정 추가([Eureka 설정](/Eureka.md)) 및 아래 spring cloud gateway 관련 항목 추가

- routing 정보 입력 
  - id: routing에 사용할 id
  - url : [LOAD BALANCING]://[EUREKA에 등록한 서비스명] 
  - -Path=[API Gateway주소 뒤에 올 서비스명]
