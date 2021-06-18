# Hystrix


### 1. Circuit Breaker란

___

- 서킷브레이커는 전기 기기에서 과부하나 과전류가 들어왔을 때 메인 기기를 보호하기 위해 흔히 쓰는 회로 차단기를 의미함.
- 또한, 주가 등락 폭이 심하게 요동칠 때도 시장 과열을 방지하기 위해 서킷 브레이커를 가동해 거래를 중지 시킬때도 사용
- **MSA에서 서킷브레이커는 특정MSA 서비스의 장애로 인해 다른 MSA 서비스에도 장애를 일으킬 수 있는 가능성을 방지하기 위해 사용**



### 2. Hystrix 소개

___

- MSA(MicroService Architecture)로 가장 유명한 Netflix社가 Amazon AWS上에서 MSA 시스템을 구축 할 때 개발한 Software 기반 Circuit Breaker로 Java로 구성되어 동작
- Spring Cloud Hystrix는 Netflex OSS 기반의 Hystrix Library를 Spring Cloud에 적용 할 수 있는 Library로 변형한 라이브러리

![Hystrix](/Img/Hystrix.png)



### 3. Hystrix 대표 기능

___

- Thread timeout, 장애 대응 등을 설정해 장해시 정해진 루트를 따르도록 처리
- 미리 정해진 임계치를 넘으면 장해가 있는 로직을 실행하지 않고 우회 하도록 처리



### 4. Hystrix 적용

___

1. spring-cloud-starter-netflix-hystrix 라이브러리 추가 
   - gradle e.g.) `implementation('org.springframework.cloud:spring-cloud-starter-netflix-hystrix` 
2. Main Application에 `@EnableCircuitBreaker` annotation 추가
3. Circuit Break를추가하고자 하는 메소드에 `@HystrixCommand` annotation추가

```java
@HystrixCommand(fallbackMethod = "doFallbackProcess")
public String getOtherServiceMessage(String param) {
        return this.restTemplate.getForObject(url, String.class);
    }
...
public String doFallbackProcess() { 
    return “Do your fall back process”; 
	}
```

