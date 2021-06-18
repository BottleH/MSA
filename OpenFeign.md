# OpenFeign


### 1. OpenFeign 소개

___

- REST Call을 위해 호출하는 클라이언트를 보다 쉽게 작성할 수 있도록 도와주는 라이브러리
- OpenFeign은 동일한 기능을 하는 RestTemplate 대비 interface를 작성하고 annotation을 붙여주면 세부적인 내용 없이 사용할 수 있기 편리한 기능 제공
- Timeout 같은 간단한 기능은 Hystrix 연동없이 실패에 대한 Callback 함수 구현 가능
- spring-cloud-starter-openfeign 라이브러리 추가로 손쉽게 사용 가능

![OpenFeign](/Img/OpenFeign.png)



### 2. OpenFeign 적용

___

1. OpenFeign Library Dependency 추가 : `compile('org.springframework.cloud:spring-cloud-starter-openfeign’)` 
2. ApplicationMain에 `@EnableFeignClients` 추가 
3. REST call을 할 REST API에 대한 FeignClient Interface 작성
4. Interface call 실패에 대한 대안(fallback) 작성
5. `application.properties(yml)`에 Feign속성 추가
