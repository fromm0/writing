# 6.1. 필수 카프카 명령어

## 6.1.1. 토픽 생성

```
./kafka-topics.sh \ 
--zookeeper dongguk-zk001:2181,dongguk-zk002:2181,dongguk-zk003:2181/dongguk-kafka \ 
--replication-factor 1 \
--partitions 1 \
--topic dongguk-topic \ 
--create

# 출력결과
# Created topic "dongguk-topic".
```

## 6.1.2. 토픽 목록 확인

```
./kafka-topics.sh \
--zookeeper dongguk-zk001:2181,dongguk-zk002:2181,dongguk-zk003:2181/dongguk-kafka \ 
--list

# 출력결과
# __consumer_offsets
# dongguk-topic
```

## 6.1.3. 토픽 상세보기

```
./kafka-topics.sh \ 
--zookeeper dongguk-zk001:2181,dongguk-zk002:2181,dongguk-zk003:2181/dongguk-kafka \ 
--topic dongguk-topic \
--describe

# 출력결과
# Topic:dongguk-topic     PartitionCount:1        ReplicationFactor:1     Configs:
#           Topic: dongguk-topic    Partition: 0    Leader: 1       Replicas: 1 Isr: 1
```

- 첫줄은 토픽의 기본정보
  - 토픽이름, 파티션수, 리플리케이션 팩터를 순서대로 나열
- 두번째 줄은 파티션 기본정보
  - dongguk-topic은 파티션이 1개이기 때문에 파티션 0번에 대한 정보만 보여줌
  - 파티션 0에는 리더가 1번 브로커
  - 리플리케이션은 1번 브로커에 있고 
  - ISR도 1번 브로커에 있음

## 6.1.4. 토픽 설정 변경

- 운영중인 카프카의 디스크 공간을 확보하는 가장 좋은 방법은 디스크 공간을 많이 차지하는 토픽의 보관주기를 줄여주는 방법

```
./kafka-configs.sh \
--zookeeper dongguk-zk001:2181,dongguk-zk002:2181,dongguk-zk003:2181/dongguk-kafka \
--alter \
--entity-type topics \
--entity-name dongguk-topic \
--add-config retention.ms=3600000

# 출력결과
# Completed Updating config for entity: topic 'dongguk-topic'.
```

















