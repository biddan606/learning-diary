# RabbitMQ vs Kafka 간단한 비교

이벤트 기반 아키텍쳐가 필요할 때, 고민하는 2가지 기술이 있습니다. `RabbitMQ` vs `Kafka` 입니다.   
2 기술을 비교하고 어떤 경우 사용하는 것이 좋을지 생각해보겠습니다.   
이 글을 쓰는 시점 이벤트 기반 아키텍쳐를 사용한 적이 없어, 경험을 통한 비교보다는 각 기술의 동작방식에 따른 비교를 해보겠습니다.   
이를 통해 이벤트 기반 아키텍쳐에 대한 이해를 높이고, 각 어떤 경우 사용하면 좋을지 알게될 것입니다.   

비교하기 전 간단하게 각 기술에 대해 이해해보겠습니다.

# RabbitMQ

![alt text](<image/RabbitMQ vs Kafka/RabbitMQ.png>)

`RabbitMQ`는 메시지 브로커의 역할을 합니다. 
1. `producer`로부터 온 메시지(요청)을 메시지 큐에 저장합니다.
2. 메시지를 `consumer`에게 전달합니다.

`RabbitMQ`의 동작방식은 단순합니다. 자료구조 `Queue` 라고 볼 수 있습니다. 작업 큐, 퍼블리시/구독 패턴등이 필요할 때 사용할 수 있을 것 같습니다.   

## Kafka

![alt text](<image/RabbitMQ vs Kafka/kafka-record.png>)

`kafka`는 메시지를 레코드라고 부릅니다. 또한 이 레코드를 디스크에 먼저 저장합니다.(`RabbitMQ`도 디스크 저장이 가능하지만, 기본은 메모리)   

![alt text](<image/RabbitMQ vs Kafka/kafka-topic.png>)

레코드들은 토픽에서 관리가 되고, 이 토픽은 여러 개의 연관성 있는 레코드들이 묶여있는 집합입니다.(주문이라 하면 주문 생성, 주문 취소, 주문 업데이트 등)   

![alt text](<image/RabbitMQ vs Kafka/kafka-cluster.png>)

전체 구조를 보게 된다면, 위와 같이 됩니다.   
`kafka`는 `cluster`처럼 여러 개의 `broker`를 둘 수 있습니다. 이로 인해 장애 대응과 작업 부하 확장성이 뛰어납니다.

## RabbitMQ, Kafka 동작방식 비교

`RabbitMQ`와 `Kafka`의 큰 차이는 `consumer`로 요청을 보내는 방식에 있다고 생각합니다.

### Consumer로 요청을 보내는 방식

`RabbitMQ`는 `push-based` 방식, `Kafka`는 `pull-based` 방식을 채택합니다. 그림으로 보자면, 먼저 `RabbitMQ` 그림입니다.

![alt text](<image/RabbitMQ vs Kafka/rabbitmq-based.png>)

1. `App1 Producer`로부터 `event`를 받습니다.
2. `biding 규칙`에 따라 `queue`를 선택합니다.
  - `queue`는 `Consumer`와 연결되어 있습니다.(`Queue` = `Consumer`가 메시지를 가져가는 대상, 하나의 큐에 여러 컨슈머가 매칭될 수 있다)
3. `queue`에서 `push-based` 방식으로 `consumer`에게 메시지를 보낸다.

여기서 `push-based` 방식이라는 건 `consumer`가 메시지를 달라고 요청하는 것이 아닌 `RabbitMQ`에서 주도적으로 전달하는 것을 의미합니다.   
이 방식은 빠르지만, `Consumer`의 상태를 알 수 없어 메시지를 받지 못할 수 있습니다.(과부하 등)

`Kafka`는 이러한 방식을 해결한 `pull-based`를 사용합니다.   

![alt text](<image/RabbitMQ vs Kafka/kafka-based.png>)

1. `Producer App1` -> `event1`, `event2`을 취급하는 `Topic` 이라는 채널에 메시지를 발행합니다.
2. 메시지가 `kafka`에 저장됩니다.
3. `App2 또는 App3 Consumer`는 `kafka`로부터 메시지를 가져옵니다.
4. `kafka`에서 메시지를 보내줍니다.

여기서 `RabbitMQ`와의 차이점은 토픽이라는 기준으로 관리하는 것과 `consumer`가 먼저 메시지를 요청하는 것입니다.   

- `consumer`가 메시지를 받을 수 있는 상태일 때, 메시지를 달라고 요청하여 과부하 같은 문제를 안 겪을 수 있습니다.   
-  `Topic`과 같이 연관성이 있는 `event`들을 묶어서 관리할 수 있습니다.

`consumer`가 요청을 한 뒤, 메시지를 보내 `RabbitMQ` 비해 느릴 수 있습니다. 다만 `consumer`의 상태에 따라 보내 안정적이며 대용량 데이터일 때는 여러 `event`를 한번에 보낼 수 있어 빠를 수 있습니다.

## 샘플 케이스를 통한 비교

대략적인 동작방식을 살펴보았으니, 샘플 케이스를 두고 기술을 선택해보겠습니다. 케이스는 아래와 같습니다.

![alt text](<image/RabbitMQ vs Kafka/sample-case.png>)
![alt text](<image/RabbitMQ vs Kafka/sample-case2.png>)

**RabbitMQ일 경우**

![alt text](<image/RabbitMQ vs Kafka/rabbitmq-sample.png>)

**Kafka일 경우**

![alt text](<image/RabbitMQ vs Kafka/kafka-sample.png>)   

`event`들의 연관관계가 정립되지 않았을 때는 `RabbitMQ`가 좋아보입니다.   
`event`들의 정립관계가 명확해져 도메인으로 분리될 때는 `kafka`로 구성하는 것이 깔끔해보입니다.

## 결론

- 두 기술은 메시지를 전달하는 방식에 있어 차이가 있다. 이 차이 때문에 간단한 구성과 복잡하지만 안정적인 구성이 있다.   
`RabbitMQ`가 전통적인 큐 방식이라면, `kafka`는 대규모 분산 시스템에 적합한 큐 느낌이 든다.
- 간단하게 구현하고 초기 단계에는 `RabbitMQ`가 적합해보인다. 시스템이 거대해져 `event`가 도메인별로 분리가 되거나, 대용량의 데이터의 이동이 필요할 경우 `Kafka`로의 전환을 고려해볼만하다.

## 참조

- [RabbitMQ in 100 Seconds - Fireship Youtube](https://www.youtube.com/watch?v=NQ3fZtyXji0)

- [Kafka in 100 Seconds](https://www.youtube.com/watch?v=uvb00oaa3k8)

- [RabbitMQ vs Kafka | Trade-off's to choose one over other | Tech Primers - Youtube](https://www.youtube.com/watch?v=GMmRtSFQ5Z0)