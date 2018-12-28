 

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
                for (ConsumerRecord<String, String> record : records) {
                    System.out.printf("Topic: %s, Parition: %s, Offset: %d, Key: %s, Value: %s\n"
                            , record.topic(), record.partition(), record.offset(), record.key(), record.value());
                }
            }
        } catch (Exception e) {
            e.toString();
        } finally {
            consumer.close();
        }
    }
}
 
// javac -classpath .:/home1/irteam/apps/kafka/libs/kafka-clients-1.0.0.jar KafkaBookConsumer1.java 
// java -classpath .:/home1/irteam/apps/kafka/libs/kafka-clients-1.0.0.jar:/home1/irteam/apps/kafka/libs/log4j-1.2.17.jar:/home1/irteam/apps/kafka/libs/slf4j-api-1.7.25.jar:/home1/irteam/apps/kafka/libs/slf4j-log4j12-1.7.25.jar KafkaBookConsumer1
```





