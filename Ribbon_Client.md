# Ribbon Client


### 1. Ribbon Client 소개

___

- Netflix OSS 라이브러리 중 Hardware적인Load Balancer를 대신해 L7 Layer에서 Client Side Load Balacer 역할을 담당함. 최근 RestTemplate 대신 사용하는 FeignClient의 경우 이미 Ribbon 기능이 포함되어 있음.

  ![Ribbon](/Img/Ribbon.png)



### 2. Ribbon Client 장점

___

- REST API를 호출하는 서비스에 탑제되는 SW 모듈
- 주어진 서버 목록에 대해 Load Balancing 수행
- 매우 다양한 설정이 가능(서버선택, 실패시 Skip시간, Ping체크 등등)
- Ribbon에는 Retry 기능이 내장되어 있음
- Eureka와함께 사용되어 매우 강력한 기능 발휘



### 3. Ribbon Client 적용

___

1. Ribbon Library Dependency 추가
   - `org.springframework.cloud:spring-cloud-starter-netflix-ribbon` 
2. `application.properties`에 호출 될 서비스들 추가
3. 연결 요청을 수행할 RestTemplate에 `@LoadBalaced Annotation` 추가(Feign 사용시 불필요)
4. RestTemplate의 호출 URL을 `application.properties`에 지정한 이름으로 변경
5. 기타 load balancing 관련 추가 옵션을 `application.yml` 에 지정



### 4. Ribbon Client Dependency Matrix

___

| 구분                 | Eureka | Spring Cloud Kubernetes |
| -------------------- | ------ | ----------------------- |
| Ribbon               | O      | O                       |
| FeignClient + Ribbon | X      | O                       |

