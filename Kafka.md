# Kafka


### 1. Apache Kafka란

___

- 아파치 카프카(Apache Kafka)는 분산 스트리밍 플랫폼이며 데이터 파이프 라인을 만들 때 주로 사용되는 오픈소스 솔루션
- 카프카는 대용량의 실시간 로그처리에 특화되어 있는 솔루션이며 데이터를 유실 없이 안전하게 전달하는 것이 주목적인 메세지 시스템 에서 Fault-Tolerant(결함 감내)한 안정적인 아키텍처와 빠른 퍼포먼스로 데이터를 처리하는 용도로 사용



### 2. Apache Kafka 특징

___

- Pub/Sub 모델 : Publishe/Subscriber 모델은 데이터 큐를 중간에 두고 서로 간 독립적으로 데이터를 생산하고 소비하는 모델. 이런 느 슨한 결합을 통해 Publisher나 Subscriber가 변경이나 장애시에도 서로 간에 의존성이 없으므로 안정적인 데이터 처리 가능.
- 고가용성(High availability) 및 확장성(Scalability) : 카프카는 클러스터로서 작동함. 클러스터로서 작동하므로 Fault-tolerant 한 고가용성 서비스를 제공할 수 있고 분산 처리를 통해 빠른 데이터 처리를 가능하게 한다. 또한 서버를 수평적으로 늘려 안정성 및 성능을 향상시키는 Scale-out이 가능
- 디스크 순차 저장 및 처리(Sequential Store and Process in Disk) : 메세지를 메모리 큐에 적재하는 기존 메세지 시스템과 다르게 카프카는 메세지를 **디스크**에 순차적으로 저장
- 서버에 장애가 나도 메세지가 디스크에 저장되어 있으므로 유실걱정이 상대적으로 적음
- 디스크가 순차적으로 저장되어 있으므로 디스크 I/O가줄어들어 성능이 향상됨
- 분산 처리(Distributed Processing) : 카프카는 파티션(Partition)이란 개념을 도입하여 여러 개의 파티션을 서버들에 분산시켜 나누어 처리할 수 있음. 이로서 메세지를 상황에 맞추어 빠르게 처리할 수 있음.



### 3. Pub/Sub 구조

___

![pubsub구조](/Img/pubsub구조.png)

**파티션**: 토픽을 병렬 및 분산처리하기 위함.

- 토픽 하나에 파티션이 여러개 있을 수 있음!

**off-set**: 파티션 내에서 Record에 대한 고유한 식별자임.

- 데이터의 안정성을 보장해줌.



### 4. 설정

___

1. dependency 추가 
   - `compile('org.springframework.kafka:spring-kafka')` 
2. application.properties에 BootStrapAddress 추가(Kafka 서버 주소)
3. `KafkaProducerConfig` 구현

```java
@Configuration
public class KafkaProducerConfig {
    @Value(value = "${kafka.bootstrapAddress}")
    private String bootstrapAddress;

    @Bean
    public ProducerFactory<String, TransferHistory> transferProducerFactory() {
        Map<String, Object> configProps = new HashMap<>();
        configProps.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, bootstrapAddress);
        configProps.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class);
        configProps.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, JsonSerializer.class);
        return new DefaultKafkaProducerFactory<>(configProps);
    }

    @Bean
    public KafkaTemplate<String, TransferHistory> transferKafkaTemplate() {
        return new KafkaTemplate<>(transferProducerFactory());
    }
}
```

4. **KafkaConsumerConfig** 구현

```java
@EnableKafka
@Configuration
public class KafkaConsumerConfig {
    @Value(value = "${kafka.bootstrapAddress}")
    private String bootstrapAddress;

    public ConsumerFactory<String, TransferHistory> b2bTransferResultConsumerFactory() {
        Map<String, Object> props = new HashMap<>();
        props.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, bootstrapAddress);
        props.put(ConsumerConfig.GROUP_ID_CONFIG, "b2btransfer");
        props.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
        props.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
        props.put(ConsumerConfig.ENABLE_AUTO_COMMIT_CONFIG, "false ");
        return new DefaultKafkaConsumerFactory<>(props, new StringDeserializer(), new JsonDeserializer<>(TransferHistory.class, false));
    }

    @Bean
    public ConcurrentKafkaListenerContainerFactory<String, TransferHistory> b2bTransferResultKafkaListenerContainerFactory() {
        ConcurrentKafkaListenerContainerFactory<String, TransferHistory> factory = new ConcurrentKafkaListenerContainerFactory<>();
        factory.setConsumerFactory(b2bTransferResultConsumerFactory());
        factory.getContainerProperties().setAckMode(AckMode.MANUAL_IMMEDIATE);
        factory.setErrorHandler(new SeekToCurrentErrorHandler());
        return factory;
    }
}
```

5. Producer, Consumer 클래스 구현

**Producer는 데이터를 제공해야하므로 직렬화를 시켜 보내고, Consumer는 직렬화된 데이터를 다시 비직렬화시킴!**
