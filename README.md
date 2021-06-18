# MSA
MSA를 적용한 프로젝트 진행 중 **공부했던 것 / 궁금했던 것 / 교육받은 내용** 등을 정리한다.



### 1. MicroService Architecture 개요

___

#### 1-1. MSA 등장배경

2007년 심각한 데이터베이스 손상으로 서비스 장애를 겪은 넷플릭스는 신뢰성 높고 수평 확장이 가능한 클라우드 시스템으로 이전할 필요성을 느꼈습니다. 단순히 플랫폼 이전만으로는 기존의 문제점과 한계를 탈피할 수 없다고 판단한 넷플릭스는 고가용성, 유연한 스케일링, 빠르고 쉬운 배포를 위해 MSA를 선택했다.

- 넷플릭스는 **살아남기 위해서** MSA를 선택했다고 한다. MSA를 적용하기 전에 충분한 검증이 필요하지 않을까?



#### 1-2. MSA의 전반적인 구조

![전반적인구조](/Img/전반적인구조.png)



### 2. 개발환경

___

#### 2-1. Spring Cloud

- Spring Cloud는 분산 시스템 상에서 공통된 개발 패턴을 통해 개발 효율을 제공해 주기 위해 개발
- Spring Cloud를 통해 분산 시스템 상에서 필요한 여러 패턴들을 Boiler Plate(표준 문안 혹은 패턴)화 시켜 손쉽게 개발 할 수 있도록 지원

**Spring Cloud 구성**

- Distributed/versioned configuration
- Service registration and discovery
- Routing
- Service-to-service calls
- Load balancing
- Circuit Breakers
- Global locks
- Leadership election and cluster state
- Distrubuted Messaging

![SpringCloud](/Img/SpringCloud.png)



#### 2-2. Circuit Breaker - Hystrix

- 서킷브레이커는 전기 기기에서 과부하나 과전류가 들어왔을 때 메인 기기를 보호하기 위해 흔히 쓰는 회로 차단기를 의미함.
- 또한, 주가 등락 폭이 심하게 요동칠 때도 시장 과열을 방지하기 위해 서킷 브레이커를 가동해 거래를 중지 시킬때도 사용
- **MSA에서 서킷브레이커는 특정MSA 서비스의 장애로 인해 다른 MSA 서비스에도 장애를 일으킬 수 있는 가능성을 방지하기 위해 사용**

![Hystrix](/Img/Hystrix.png)

[Hystrix 정리](/Hystrix.md)



#### 2-3. Load Balancer – Ribbon Client

Netflix OSS 라이브러리 중 Hardware적인Load Balancer를 대신해 L7 Layer에서 Client Side Load Balacer 역할을 담당함. 최근 RestTemplate 대신 사용하는 FeignClient의 경우 이미 Ribbon 기능이 포함되어 있음.

![Ribbon](/Img/Ribbon.png)

[Ribbon_Client 정리](/Ribbon_Client.md)



#### 2-4. Service Discovery – Eureka

- MicrosService를 구성하는 서비스들의 목록과 위치(IP, Port)가 동적으로 변하는 환경 하에서 서비스들을 효율적으로 관리하기 위해 Netflix OSS 기반으로 개발한 Service Discovery Server와 Client
- Java로 개발된 Netflix OSS 기반 라이브러리는 Spring Cloud의 Library와 통합되어 Spring Cloud – Eureka로 적용됨

![Eureka](/Img/Eureka.png)

[Eureka 정리](/Eureka.md)



#### 2-5. API Gateway – Spring Gateway

- 다수의 서비스로 구성된 MicroService 들에서 각 서비스들의 IP와 PORT 번호들에 대한 단일화된 엔드포인트 제공
- 각 서비스들에서 필요한 인증/인가, 사용량 제어, 요청/응답 변조 등의 기능을 대신 담당

![Spring_Gateway](/Img/Spring_Gateway.png)

[Spring_Gateway 정리](/Spring_Gateway.md)



#### 2-6. REST Call을위한 OpenFeign

- REST Call을 위해 호출하는 클라이언트를 보다 쉽게 작성할 수 있도록 도와주는 라이브러리
- OpenFeign은 동일한 기능을 하는 RestTemplate 대비 interface를 작성하고 annotation을 붙여주면 세부적인 내용 없이 사용할 수 있기 편리한 기능 제공
- Timeout 같은 간단한 기능은 Hystrix 연동없이 실패에 대한 Callback 함수 구현 가능
- spring-cloud-starter-openfeign 라이브러리 추가로 손쉽게 사용 가능

![OpenFeign](/Img/OpenFeign.png)

[OpenFeign 정리](/OpenFeign.md)



#### 2-7. Kafka

- 카프카(Apache Kafka)는 분산 스트리밍 플랫폼이며 데이터 파이프 라인을 만들 때 주로 사용되는 오픈소스 솔루션

![pubsub구조](/Img/pubsub구조.png)

[Kafka 정리](/Kafka.md)



### 3. Transaction in MSA

___

![Transaction](/Img/Transaction.png)

❗ RuntimeException: 주로 프로그램 에러에 의해 발생하는 Exception

- e.g: `NullPointException`, `IllegalArgumentException`…
- Spring의Tranactional annotation은RuntimeException 기준



#### 3-1. REST call, Publishing이 혼합된 Service Layer에서 Transaction 처리

![Transaction2](/Img/Transaction2.png)

**REST update 와 Q publish 전략필요** 

- REST update는DB 다음에 처리 
- Q를 제일 마지막에 처리 
  - Compensation(보상) 최소화



#### 3-2. Listener(Subscribe) 처리 고려사항

![Subscribe처리](/Img/Subscribe처리.png)

1. KafkaListener의 Ack 모드를 수동으로 지정

```java
ConcurrentKafkaListenerContainerFactory<String, Account> factory = new ConcurrentKafkaListenerContainerFactory<>(); 
// Listener의 AckMode를 수동으로 지정
factory.getContainerProperties().setAckMode(AckMode. MANUAL_IMMEDIATE );
```

2. 각 ConsumerFactory별 `ENABLE_AUTO_COMMIT_CONFIG = false` 속성추가

```java
props.put(ConsumerConfig. ENABLE_AUTO_COMMIT_CONFIG,"false");
```



### 4. API Composition

___

#### 기존

![Api기존](/Img/Api기존.png)

- 웹 브라우저에서 3개의 서비스를 호출해야 하며 Sequential한 호출로 인해 전체호출시간 증가

#### API Composition

![API_Composition](/Img/API_Composition.png)

- 웹 브라우저에서API Composite Service하나만 호출 하고 API Composite에서 나머지 서비스들을 Thread를 이용해 Parallel 하게 호출
  - 단, 업무의 성격상 순차 처리해야하는 경우는 제외



### 5. CQRS

___

#### 기존

![cqrs기존](/Img/cqrs기존.png)

- 웹 브라우저에서 표시해야 할 데이터가 Service A,B,C의 데이터를 조인(Join)해야 할 경우, 데이터 조회를 위해 Composite 해야 하는 시간이 오래 걸림.



#### CQRS

![cqrs](/Img/cqrs.png)

- 화면에서각 서비스 간에 데이터 조인(Join)을 통해 데이터 표시를 해야 할 경우 큐(Queue)를 이용해 필요한 데이터를 미리 고속의 CQRS(Read Only) 데이터베이스로 복제해 서비스 시간을 줄이는 방식
