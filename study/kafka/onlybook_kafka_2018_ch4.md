# 4.1. 콘솔 프로듀서로 메시지 보내기

```  
./kafka-topics.sh --zookeeper dev-dongguk-zk001-ncl:2181,dev-dongguk-zk002-ncl:2181,dev-dongguk-zk003-ncl:2181/dongguk-kafka --topic dongguk-topic --partitions 1 --replication-factor 3 --create
# 출력결과
# Created topic "dongguk-topic"
```

- 테스트용 토픽을 생성
  - 토픽 이름 : dongguk-topic
  - 파티션 : 1
  - 리플리케이션 팩터 : 3

```
./kafka-topics.sh --zookeeper dev-dongguk-zk001-ncl:2181,dev-dongguk-zk002-ncl:2181,dev-dongguk-zk003-ncl:2181/dongguk-kafka --topic dongguk-topic --describe

# 출력결과
# Topic:dongguk-topic     PartitionCount:1        ReplicationFactor:3     Configs:
#           Topic: dongguk-topic    Partition: 0    Leader: 2       Replicas: 2,1,3 Isr: 2,1,3
```

- 토픽의 정보를 확인
  - 1개의 파티션
  - 리플리케이션 팩터는 3
  - 리더는 2번 브로커의 0번 파티션에 위치
  - 리더와 ISR은 2,1,3에 위치
  - **그림 4-1** 

- 콘솔에서 메시지 전송 테스트

```
./kafka-console-producer.sh --broker-list dongguk-kafka001:9092,dongguk-kafka002:9092,dongguk-kafka003:9092 --topic dongguk-topic

# 출력결과
# > hello
```

```
./kafka-console-consumer.sh --bootstrap-server dongguk-kafka001:9092,dongguk-kafka002:9092,dongguk-kafka003:9092 --topic dongguk-topic --from-beginning

# 출력결과
# hello
```

# 4.2. 자바와 파이썬을 이용한 프로듀서

```
import org.apache.kafka.clients.producer.KafkaProducer;
import org.apache.kafka.clients.producer.Producer;
import org.apache.kafka.clients.producer.ProducerRecord;

import java.util.Properties;

public class KafkaBookProducer1 {
  public static void main(String[] args) {
    Properties props = new Properties();
    // 브로커 목록 선언
    props.put("bootstrap.servers", "dev-dongguk-zk001-ncl:9092,dev-dongguk-zk002-ncl:9092,dev-dongguk-zk003-ncl:9092");
    props.put("acks", "1");
    props.put("compression.type", "gzip");
    // 문자열 타입의 키와 값을 처리
    props.put("key.serializer", "org.apache.kafka.common.serialization.StringSerializer");
    props.put("value.serializer", "org.apache.kafka.common.serialization.StringSerializer");

    Producer<String, String> producer = new KafkaProducer<>(props);
    // dongguk-topic 토픽에 메시지를 전달하도록 ProducerRecord를 생성하고 send() 메소드 호출
    producer.send(new ProducerRecord<String, String>("dongguk-topic", "Apache Kafka is a distributed streaming platform"));
    producer.close();
  }
}
```
** 예제 4-1** 자바를 이용한 프로듀서 producer.java 파일

## 4.2.1. 메시지를 보내고 확인하지 않기 

- 프로듀서에서 서버로 메시지를 보내고 난 후 성공적으로 도착했는지 확인하지 않음
- 일부 메시지 유실 가능함

```
Producer<String, String> producer = new KafkaProducer<>(props);
try {
  // send() 메소드는 자바 Future객체로 RecordMetdata를 리턴받지만 리턴값을 무시하기 때문에 성공여부를 알수 없음
  // 메시지 유실 가능성이 있어서 서비스 환경에서는 사용하지 않음
  producer.send(new ProducerRecord<String, String>("dongguk-topic", "Apache Kafka is a distributed streaming platform"));
} catch (Exception exception) {
  exception.printStackTrace();
} finally {
  producer.close();
}
```

## 4.2.2. 동기전송

```
Producer<String, String> producer = new KafkaProducer<>(props);
try {
  // send() 메소드의 결과에 get() 메소드 호출 후 RecordMetdata를 리턴받음
  RecordMetadata metadata = producer.send(new ProducerRecord<String, String>("dongguk-topic", "Apache Kafka is a distributed streaming platform")).get();
  // RecordMetadata를 사용해서 성공여부 체크
  System.out.println("Partition : %d, Offset : %d", metadata.partition(), metadata.getOffset());
} catch (Exception exception) {
  exception.printStackTrace();
} finally {
  producer.close();
}
```

## 4.2.3. 비동기 전송

- send() 메소드 호출시 콜백을 함께 전달해서 응답을 받으면 콜백이 호출되도록 처리
- 동기전송으로 응답을 기다릴 경우 오랜시간이 걸릴수 있다. 

```
import org.apache.kafka.clients.producer.Callback;

class DonggukCallback implements Callback {
  public void onCompletion(RecordMetadata metadata, Exception exception) {
    if(metadata != null) {
      System.out.println("Partition: " + metadata.partition() + ", Offset : " + metadata.offset() + " ");
    } else {
      exception.printStackTrace();
    }
  }
}
```
 
 ```
 Producer<String, String> producer = new KafkaProducer<>(props);
 try {
   // send() 메소드의 마지막 인자로 DonggukCallback 을 전달
   RecordMetadata metadata = producer.send(new ProducerRecord<String, String>("dongguk-topic", "Apache Kafka is a distributed streaming platform"), new DonggukCallback());
 } catch (Exception exception) {
   exception.printStackTrace();
 } finally {
   producer.close();
 }
 ```

## 4.3. 프로듀서 활용예제

- 프로듀서의 경우 key옵션 사용이 가능함
- key옵션을 사용하지 않으면 라운드 로빈 방식으로 파티션 마다 균등하게 메시지 전송
- key옵션을 사용하면 특정 파티션으로만 메시지 전송가능

```
import org.apache.kafka.clients.producer.KafkaProducer;
import org.apache.kafka.clients.producer.Producer;
import org.apache.kafka.clients.producer.ProducerRecord;

import java.util.Properties;

public class KafkaBookProducerKey {
  public static void main(String[] args) {
    Properties props = new Properties();
    props.put("bootstrap.servers", "dev-dongguk-zk001-ncl:9092,dev-dongguk-zk002-ncl:9092,dev-dongguk-zk003-ncl:9092");
    props.put("acks", "1");
    props.put("compression.type", "gzip");
    props.put("key.serializer", "org.apache.kafka.common.serialization.StringSerializer");
    props.put("value.serializer", "org.apache.kafka.common.serialization.StringSerializer");

    Producer<String, String> producer = new KafkaProducer<>(props);
    String testTopic = "dongguk-topic";
    String oddKey = "1";
    String evenKey = "2";

    for (int i = 1; i < 11; i++) {
      if (i % 2 == 1) {
        producer.send(new ProducerRecord<String, String>(testTopic, oddKey, String.format("%d - Apache Kafka is a distributed streaming platform - key=" + oddKey, i)));
      } else {
        producer.send(new ProducerRecord<String, String>(testTopic, evenKey, String.format("%d - Apache Kafka is a distributed streaming platform - key=" + evenKey, i)));
      }
    }

    producer.close();
  }
}

# 출력결과
# 2 - Apache Kafka is a distributed streaming platform - key=2
# 4 - Apache Kafka is a distributed streaming platform - key=2
# 6 - Apache Kafka is a distributed streaming platform - key=2
# 8 - Apache Kafka is a distributed streaming platform - key=2
# 10 - Apache Kafka is a distributed streaming platform - key=2
# 1 - Apache Kafka is a distributed streaming platform - key=1
# 3 - Apache Kafka is a distributed streaming platform - key=1
# 5 - Apache Kafka is a distributed streaming platform - key=1
# 7 - Apache Kafka is a distributed streaming platform - key=1
# 9 - Apache Kafka is a distributed streaming platform - key=1
```
**예제 4-7** 키를 이용한 프로듀서 producer-key.java 파일

- 파티션 수에 따라 확인이 필요할듯
  - 파티션이 두개일 경우 key가 2인 메시지는 0번 파티션에만 전송, key가 1인 메시지는 1번 파티션에만 전송
  - 파티션이 3개인 경우 ??


## 4.4. 프로듀서 주요 옵션

- bootstrap.servers
  - 카프카 클러스터에 처음 연결하기 위한 호스트와 포트 정보로 구성된 목록
  - 콤마로 구분해서 나열
  - 대상 호스트 전체 입력을 추천
- acks
  - 프로듀서가 카프카 토픽의 리더에게 메시지를 보낸 후 요청 완료 전 acl(승인)의 수
  - 수가 작으면 성능은 좋지만 메시지 손실 가능성이 있고
  - 수가 크면 성능은 좋지 않지만 손실 가능성도 줄어들거나 없어짐
  - ack=0 
    - 프로듀서는 서버로부터 어떤 ack도 기다리지 않음
    - 데이터를 받았는지 보장하지 않고 재요청 설정도 적용되지 않음
    - 메시지가 손실될수 있지만 높은 처리량을 얻을 수 있음
  - ack=1
    - 리더는 데이터를 기록하지만 팔로워는 확인하지 않음
  - ack=all (또는 -1)
    - 리더는 ISR의 팔로워로부터 데이터에 대한 ack를 기다림
    - 데이터 무손실을 강력히 보장함
  - buffer.memory
    - 프로듀서가 카프카 서버로 데이터를 보내기 위해 잠시 대기(배치전송이나 딜레이)할수 있는 전체 메모리 사이즈(바이트 단위)
  - compression.type
    - 프로듀서가 데이터를 압축해서 보낼때의 타입
    - none, gzip, snappy, lz4 등을 선택가능
  - retries
    - 일시적인 오류로 인해 전송에 실패한 데이터를 다시 보내는 횟수
  - batch.size
    - 프로듀서는 동일 파티션으로 보내는 여러 데이터를 함께 배치로 보내려고 함
    - 배치로 보내는 행위는 서버와 클라이언트 양쪽에 성능상 도움이 됨
    - 이 설정값은 배치 크기를 설정하는 것으로 이보다 크다면 배치가 아닌 바로 전송
  - linger.ms
    - 배치 형태의 메시지를 보내기 전에 추가 메시지를 기다리는 시간
    - 배치 사이즈에 도달한 경우 즉시 전송하지만 배치 사이즈에 도달하기 전에는 이 설정값만큼 기다렸다가 전송
  - max.request.size
    - 프로듀서가 보낼수 있는 최대 메시지 크기(바이트 단위)
    - 기본값은 1MB
    
## 4.5. 메시지 전송 방법

- 프로듀서 옵션 중 acks 옵션 설정에 따라 결정되는 사항들
  - 메시지 손실 여부
  - 메시지 전송 속도
  - 처리량 

### 4.5.1. 메시지 손실 가능성이 높지만 빠른 전송이 필요한 경우

**그림 4-2** 프로듀서의 acks=0 메시지 전송 방법

- 카프카 서버에서 응답을 기다리지 않음
- 일반적인 운영환경에서는 메시지 유실이 없음
- 브로커가 다운되는 장애등의 경우에만 메시지 손실 가능성이 높다고 보면 됨


### 4.5.2. 메시지 손실 가능성이 적고 적당한 속도의 전송이 필요한 경우

**그림 4-3** 프로듀서의 acks=1 메시지 전송 방법

- 메시지는 보내는 시간에서 응답을 기다리는 시간만큼 속도가 약간 떨어짐
- 메시지 전송에 1초, 응답을 기다리는데 1초라면 acks=0은 소요시간이 1초, acks=1은 소요시간이 2초가 되는 셈


**그림 4-4** 프로듀서가 acks=1로 메시지 전송 후 카프카 내에서 리더와 팔로워가 메시지를 저장하는 순서

- 

**그림 4-5** 프로듀서가 acks=1로 메시지 전송 후 리더가 그 메시지에 대해 acks를 보낸 후 장애 발생 상황


### 4.5.3. 전송속도는 느리지만 메시지 손실이 없어야 하는 경우







