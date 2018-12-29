 

# 컨슈머 주요 옵션

- 컨슈머 종류 (주키퍼 사용 여부에 따라 구분)
  - 올드 컨슈머 (Old Consumer) : 컨슈머의 오프셋을 주키퍼의 지노드에 저장
  - 뉴 컨슈머 (New Consumer) : 컨슈머의 오프셋을 카파카의 토픽에 저장 (카프카 0.9버전부터 변경)
- 뉴 컨슈머 기준의 설정
  - bootstrap.servers 
    - 카프카 클러스터에 연결하기 위한 호스트와 포트 정보로 구성된 목록
    - 콤마로 구분해서 "호스트명:포트,호스트명:포트" 형태로 입력
    - 가능하면 대상 호스트 전체를 입력하도록 권장
  - fetch.min.bytes
    - 한번에 가져올수 있는 최소 데이터 크기
    - 지정한 크기보다 작다면 데이터를 누적해서 해당 크기만큼 기다림
  - group.id
    - 컨슈머가 속한 컨슈머 그룹 식별자
  - enable.auto.commit
    - 백그라운드로 주기적으로 오프셋을 커밋
  - auto.offset.reset
    - 카프카에서 초기 오프셋이 없거나 현재 오프셋이 더 이상 존재하지 않은 경우 다음 옵션으로 리셋
      - earliest : 가장 초기의 오프셋값으로 설정
      - latest : 가장 마지막의 오프셋값으로 설정
      - none : 이전 오프셋값을 찾지 못하면 에러
  - fetch.max.bytes
    - 한번에 가져올수 있는 최대 데이터 크기
  - request.timeout.ms
    - 요청에 대해 응답을 기다리는 최대 시간
  - session.timeout.ms
    - 컨슈머와 브로커 사이 세션 타임 아웃 시간. 브로커가 컨슈머가 살아있는 것으로 판단하는 시간(기본값 10초)
    - 컨슈머가 이 시간내 하트비트(heartbeat)를 보내지 않으면 컨슈머 그룹은 리밸런스 시도
    - heartbeat.internal.ms 와 관련있음
  - hearbeat.interval.ms
    - 그룹 코디네이터에게 얼마나 자주 KafkaConsumer poll() 메소드로 하트비트를 보낼 것인지 조정
    - session.timeout.ms 보다는 낮아야 하고 일반적으로 1/3 수준으로 설정
  - max.poll.records
    - 단일 호출 poll()에 대한 최대 레코드 수를 조정
    - 애플리케이션이 폴링 루프에서 데이터 양을 조정가능
  - max.poll.interval.ms
    - 데이터는 가져가지 않고 하트비트만 보내는지 체크
    - 주기적으로 poll을 호출하는지 체크해서 장애 체크
  - auto.commit.interval.ms
    - 주기적으로 오프셋을 커밋하는 시간
  - fetch.max.wait.ms
    - fetch.min.bytes 에 의해 설정된 데이터보다 적은 경우 요청에 응답을 기다리는 최대 시간

# 콘솔 컨슈머로 메시지 가져오기
컨슈머는 토픽에서 메시지를 가져옴

```
./kafka-consumer-groups.sh --bootstrap-server dev-dongguk-zk001-ncl:9092,dev-dongguk-zk002-ncl:9092,dev-dongguk-zk003-ncl:9092 --list
```

```
./kafka-console-consumer.sh --bootstrap-server dev-dongguk-zk001-ncl:9092,dev-dongguk-zk002-ncl:9092,dev-dongguk-zk003-ncl:9092 --topic dongguk-topic --group dongguk-consumer-group --from-beginning
```

# 자바와 파이썬을 이용한 컨슈머

```
import org.apache.kafka.clients.producer.Callback;
import org.apache.kafka.clients.producer.RecordMetadata;

public class KafkaCallback implements Callback {
    @Override
    public void onCompletion(RecordMetadata metadata, Exception exception) {
        if (metadata != null) {
            System.out.println("Partition: " + metadata.partition() +", offset :" + metadata.offset());
        } else {
            exception.printStackTrace();
        }
    }
}
```

```
// https://github.com/yookeun/kafka-java/blob/master/src/main/java/com/example/kafka/KafkaBookProducer1.java
import org.apache.kafka.clients.producer.KafkaProducer;
import org.apache.kafka.clients.producer.Producer;
import org.apache.kafka.clients.producer.ProducerRecord;

import java.util.Properties;

public class KafkaBookProducer1 {
    public static void main(String[] args) {
        Properties props = new Properties();
        props.put("bootstrap.servers", "dev-dongguk-zk001-ncl:9092,dev-dongguk-zk002-ncl:9092,dev-dongguk-zk003-ncl:9092");
        props.put("key.serializer", "org.apache.kafka.common.serialization.StringSerializer");
        props.put("value.serializer", "org.apache.kafka.common.serialization.StringSerializer");

        Producer<String, String> producer = new KafkaProducer<>(props);
        try {
            producer.send(new ProducerRecord<String,String>("dongguk-topic", "Apache Kafka3!"), new KafkaCallback());
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            producer.close();
        }
    }
}

// javac -classpath .:/home1/irteam/apps/kafka/libs/kafka-clients-1.0.0.jar KafkaBookProducer1.java 
// java -classpath .:/home1/irteam/apps/kafka/libs/kafka-clients-1.0.0.jar:/home1/irteam/apps/kafka/libs/log4j-1.2.17.jar:/home1/irteam/apps/kafka/libs/slf4j-api-1.7.25.jar:/home1/irteam/apps/kafka/libs/slf4j-log4j12-1.7.25.jar KafkaBookProducer1
```

```
// https://github.com/yookeun/kafka-java/blob/master/src/main/java/com/example/kafka/KafkaBookConsumer1.java
import org.apache.kafka.clients.consumer.ConsumerRecord;
import org.apache.kafka.clients.consumer.ConsumerRecords;
import org.apache.kafka.clients.consumer.KafkaConsumer;

import java.util.Arrays;
import java.util.Properties;

public class KafkaBookConsumer1 {

    public static void main(String[] args) {
        Properties props = new Properties();
        // 브로커 목록
        props.put("bootstrap.servers", "dev-dongguk-zk001-ncl:9092,dev-dongguk-zk002-ncl:9092,dev-dongguk-zk003-ncl:9092");
        // 그룹아이디. 5.5에서 계속 다룸
        props.put("group.id", "dongguk-consumer-group");        
        // 자동커밋으로 5.6에서 계속 다룸
        props.put("enable.auto.commit", "true");
        // 오프셋 리셋값을 지정
        props.put("auto.offset.reset", "latest");
        props.put("key.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
        props.put("value.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");

        // 컨슈머 생성
        KafkaConsumer<String, String>  consumer = new KafkaConsumer<String, String>(props);
        // 메시지를 가져올 토픽을 지정
        consumer.subscribe(Arrays.asList("dongguk-topic"));

        try {
            while(true) {
                // 무한루프 형태로 계속 폴링처리
                // https://kafka.apache.org/0101/javadoc/org/apache/kafka/clients/consumer/KafkaConsumer.html#poll(long)
                ConsumerRecords<String, String> records = consumer.poll(100);
                // poll 메소드는 레코드 전체 (토픽, 파티션, 오프셋, 키, 값 등)를 리턴 
                // 실제 N개의 메시지를 반복문을 통해 처리해야 함
                for (ConsumerRecord<String, String> record : records) {
                    System.out.printf("Topic: %s, Parition: %s, Offset: %d, Key: %s, Value: %s\n"
                            , record.topic(), record.partition(), record.offset(), record.key(), record.value());
                }
            }
        } catch (Exception e) {
            e.toString();
        } finally {
            // 컨슈머를 종료하기 전에 close()메소드를 호출
            consumer.close();
        }
    }
}
 
// javac -classpath .:/home1/irteam/apps/kafka/libs/kafka-clients-1.0.0.jar KafkaBookConsumer1.java 
// java -classpath .:/home1/irteam/apps/kafka/libs/kafka-clients-1.0.0.jar:/home1/irteam/apps/kafka/libs/log4j-1.2.17.jar:/home1/irteam/apps/kafka/libs/slf4j-api-1.7.25.jar:/home1/irteam/apps/kafka/libs/slf4j-log4j12-1.7.25.jar KafkaBookConsumer1
```

# 파티션과 메시지 순서

> [Kafka 운영자가 말하는 처음 접하는 Kafka](https://www.popit.kr/kafka-%EC%9A%B4%EC%98%81%EC%9E%90%EA%B0%80-%EB%A7%90%ED%95%98%EB%8A%94-%EC%B2%98%EC%9D%8C-%EC%A0%91%ED%95%98%EB%8A%94-kafka/) 참고
> - "파티션 수에 따른 메시지 순서" 절에도 동일한 설명이 있음

- 리플리케이션 팩터는 다르게 설정해도 되지만 파티션 수는 동일하게 토픽 생성해야 함

#### 테스트로 사용할 토픽을 만들어보자

```
./kafka-topics.sh --zookeeper dev-dongguk-zk001-ncl:2181,dev-dongguk-zk002-ncl:2181,dev-dongguk-zk003-ncl:2181/dongguk-kafka --topic dongguk-01 --partitions 3 --replication-factor 1 --create

# 출력결과
# Created topic "dongguk-01".
```

## 파티션 3개로 구성한 dongguk토픽과 메시지 순서

- dongguk-01 토픽
  - 파티션 수가 3
  - 리플리케이션 팩터는 1

#### dongguk-01토픽을 메시지를 보내보자

```
./kafka-console-producer.sh --broker-list dev-dongguk-zk001-ncl:9092,dev-dongguk-zk002-ncl:9092,dev-dongguk-zk003-ncl:9092 --topic dongguk-01

# 출력결과
# > a
# ... 중략
# > e
```

#### dongguk-01토픽에 보낸 메시지를 받아보자. 

```
./kafka-console-consumer.sh --bootstrap-server dev-dongguk-zk001-ncl:9092,dev-dongguk-zk002-ncl:9092,dev-dongguk-zk003-ncl:9092 --topic dongguk-01 --from-beginning

# 출력결과
# a
# d
# b
# e
# c
```

- 메시지 전송시 a부터 e까지 순서대로 보냈지만 수신된 순서는 다소 다를수 있음

#### 토픽의 파티션별로 저장된 메시지를 확인해보자. 

- kafka-console-consumer.sh명령어에 --partition파라미터를 추가하면 파티션별 메시지를 확인할수 있음 

```
./kafka-console-consumer.sh --bootstrap-server dev-dongguk-zk001-ncl:9092,dev-dongguk-zk002-ncl:9092,dev-dongguk-zk003-ncl:9092 --topic dongguk-01 --partition 0 --from-beginning

# 출력결과
# c
# f
# i
# l
# o
# r
```

```
./kafka-console-consumer.sh --bootstrap-server dev-dongguk-zk001-ncl:9092,dev-dongguk-zk002-ncl:9092,dev-dongguk-zk003-ncl:9092 --topic dongguk-01 --partition 1 --from-beginning

# 출력결과
# b
# e
# h
# k
# n
# q
# t
```

```
./kafka-console-consumer.sh --bootstrap-server dev-dongguk-zk001-ncl:9092,dev-dongguk-zk002-ncl:9092,dev-dongguk-zk003-ncl:9092 --topic dongguk-01 --partition 2 --from-beginning

# 출력결과
# a
# d
# g
# j
# m
# p
# s
```

- 동일 파티션내에서는 프로듀서가 생성한 순서와 동일하게 처리, 파티션과 파티션 사이에서는 순서 보장안됨

## 파티션 1개로 구성한 dongguk-02토픽과 메시지 순서

메시지 순서를 정확히 보장하기 위해서는 파티션 수를 1로 지정해서 사용해야 함

```
./kafka-topics.sh --zookeeper dev-dongguk-zk001-ncl:2181,dev-dongguk-zk002-ncl:2181,dev-dongguk-zk003-ncl:2181/dongguk-kafka --topic dongguk-02 --partitions 1 --replication-factor 1 --create

# 출력결과
# Created topic "dongguk-02".
```

```
./kafka-console-producer.sh --broker-list dev-dongguk-zk001-ncl:9092,dev-dongguk-zk002-ncl:9092,dev-dongguk-zk003-ncl:9092 --topic dongguk-02

# 출력결과
# > a
# ... 중략
# > e
```

```
./kafka-console-consumer.sh --bootstrap-server dev-dongguk-zk001-ncl:9092,dev-dongguk-zk002-ncl:9092,dev-dongguk-zk003-ncl:9092 --topic dongguk-02 --from-beginning

# 출력결과
# a
# b
# c
# d
# e
```

- 파티션 3인 dongguk-01 의 결과와 달리 메시지가 순서대로 출력됨

```
./kafka-console-consumer.sh --bootstrap-server dev-dongguk-zk001-ncl:9092,dev-dongguk-zk002-ncl:9092,dev-dongguk-zk003-ncl:9092 --topic dongguk-02 --partition 0 --from-beginning

# 출력결과
# a
# b
# c
# d
# e
```

- 파티션이 1개라 0번째 파티션의 메시지 내용을 확인해보더라도 출력된 결과와 동일하게 순서대로 들어있음
- 메시지를 순서대로 처리하기 위해서는 파티션을 1개만 사용해야 하지만 성능이 떨어짐
- 성능을 위해서는 대부분 파티션을 여러개 사용하지만 이 경우 메시지의 완전한 순서 보장은 어려움

# 컨슈머 그룹

- 컨슈머 그룹은 하나의 토픽에 여러 컨슈머 그룹이 동시에 접속해 메시지를 가져올수 있음
- 컨슈머 그룹은 컨슈머를 확장하는게 가능
- 프로듀서가 메시지를 더 빨리 생성하고 컨슈머가 그 만큼 처리를 못하는 경우 컨슈머 그룹을 통해 컨슈머를 확장하면 처리량이 증가함

**그림5-4**

- dongguk-01 토픽의 파티션 수는 3으로 구성된 토픽
- 메시지는 컨슈머 01이 처리 중
- 메시지 생성이 많아 dongguk-01 토픽에 메시지가 계속 쌓인다면

**그림5-5**

- 컨슈머 그룹내에서 컨슈머들은 메시지를 가져오고 있는 토픽의 파티션에 소유권을 공유
- 컨슈머가 부족해 처리가 늦다면 동일 컨슈머 그룹 아이디를 지정해서 컨슈머를 추가해야 함
- 기존 컨슈머 01이 파티션 1,2,3의 소유권을 가지고 있었음
- 컨슈머 02, 03이 추가되면서 컨슈머 02는 파티션 2, 컨슈머 03은 파티션 3을 가져가서 처리하는데 이렇게 소유권이 이동하는 것을 리밸런스(rebalance)라고 함
- 컨슈머 그룹내에서는 리밸런스를 통해 컨슈머를 안전하게 추가및 삭제가 가능함

- 리밸런스의 단점
  - 리밸런스하는 동안 일시적으로 컨슈머가 메시지를 가져가지 못함
  - 즉 리밸런스 시간동안 컨슈머 그룹 전체를 일시적으로 사용할수 없을수 있음

- 토픽내 파티션 수보다 컨슈머 그룹내 컨슈머 수가 많을 경우 (**그림 5-6**)
  - 토픽의 파티션에는 하나의 컨슈머만 연결가능
  - 즉 컨슈머 그룹내 컨슈머가 더 많은 경우 나머지는 동작하지 않을 수 있음

- 컨슈머 그룹내 컨슈머를 추가해도 메시지 처리가 늦다면
  - 컨슈머 추가와 함께 파티션도 추가해야 함


















