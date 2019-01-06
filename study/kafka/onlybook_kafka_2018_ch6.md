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
- 다음은 보관주기를 1시간으로 줄이는 명령어

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

- --add-config로 추가한 설정을 삭제하려면 --delete-config를 사용

```
./kafka-configs.sh \
--zookeeper dongguk-zk001:2181,dongguk-zk002:2181,dongguk-zk003:2181/dongguk-kafka \
--alter \
--entity-type topics \
--entity-name dongguk-topic \
--delete-config retention.ms

# 출력결과
# Completed Updating config for entity: topic 'dongguk-topic'.
```

### 6.1.5. 토픽의 파티션수 변경

- 토픽의 파티션 수는 증가는 가능하지만 감소는 불가능
- 파티션은 늘리면 메시지 순서에 영향이 있을수 있음

```
./kafka-topics.sh \ 
--zookeeper dongguk-zk001:2181,dongguk-zk002:2181,dongguk-zk003:2181/dongguk-kafka \ 
--alter \
--topic dongguk-topic \
--partitions 2

# 출력결과
# Adding partitions succeeded!
```

- 파티션 추가 후 정보를 출력해보면 1번 파티션 정보가 추가로 출력됨

```
./kafka-topics.sh \ 
--zookeeper dongguk-zk001:2181,dongguk-zk002:2181,dongguk-zk003:2181/dongguk-kafka \ 
--topic dongguk-topic \
--describe

# 출력결과
# Topic:dongguk-topic     PartitionCount:2        ReplicationFactor:1     Configs:
#           Topic: dongguk-topic    Partition: 0    Leader: 1       Replicas: 1 Isr: 1
#           Topic: dongguk-topic    Partition: 1    Leader: 2       Replicas: 2 Isr: 2
```

## 6.1.6. 토픽의 리플리케이션 팩터 변경

- 리플리케이션 팩터를 변경하기 위해 json형식의 파일이 필요함

```
{"version":1,
"partitions":[
    {"topic":"dongguk-topic","partition":0,"replicas":[1,2]}
    {"topic":"dongguk-topic","partition":1,"replicas":[2,3]}    
]}
```

- 파티션 0의 복제수는 2 (replicas의 숫자 1, 2가 2개) 이고 리더는 1, 2 의 순서기준으로 브로커1 이 리더이고 브로커2가 리플리카
- 파티션 1은 복제수가 2이고 브로커2가 리더, 브로커3이 리플리카 
- 리더는 변경되지 않도록 현재 토픽의 리더정보를 확인해서 그대로 설정해야 함

```
./kafka-reassign-partitions.sh \
--zookeeper dongguk-zk001:2181,dongguk-zk002:2181,dongguk-zk003:2181/dongguk-kafka \
--reassignment-json-file rf.json \
--execute

# 출력결과
# Current partition replica assignment
# Successfully started reassignment of partitions
```

```
./kafka-topics.sh \ 
--zookeeper dongguk-zk001:2181,dongguk-zk002:2181,dongguk-zk003:2181/dongguk-kafka \ 
--topic dongguk-topic \
--describe

# 출력결과
# Topic:dongguk-topic     PartitionCount:2        ReplicationFactor:2     Configs:
#           Topic: dongguk-topic    Partition: 0    Leader: 1       Replicas: 1,2 Isr: 1,2
#           Topic: dongguk-topic    Partition: 1    Leader: 2       Replicas: 2,3 Isr: 2,3
```

- ReplicationFactor가 2로 변경됨
- 각 파티션마다 replicas가 숫자가 1개였던 것들이 2개로 늘어남

## 6.1.7. 컨슈머 그룹 목록 확인

#### 오프셋 저장방식에 따른 컨슈머 구분 방법
- 올드 컨슈머 : 오프셋을 주키퍼에 저장
  - -zookeeper와 주키퍼 목록 입력 필요
- 뉴 컨슈머 : 오프셋을 카프카의 토픽에 저장
  - --bootstrap-server와 브로커 목록 입력 필요

```  
./kafka-consumer-groups.sh \
--bootstrap-server dongguk-kafka001:9092,dongguk-kafka002:9092,dongguk-kafka003:9092 \
--list

# 출력결과
# dongguk-consumer
```

- --bootstrap-server 옵션을 사용했기 때문에 주키퍼 기반이 아닌 컨슈머만 보임

## 6.1.8. 컨슈머 상태와 오프셋 확인

```  
./kafka-consumer-groups.sh \
--bootstrap-server dongguk-kafka001:9092,dongguk-kafka002:9092,dongguk-kafka003:9092 \
--group dongguk-consumer
--describe

# 출력결과
# Consumer group 'dongguk-consumer' has no active members.

TOPIC PARTITION CURRENT-OFFSET  LOG-END-OFFSET  LAG CONSUMER-ID HOST  CLIENT-ID
dongguk-topic 1 2 2 0 - - - 
dongguk-topic 0 8 8 0 - - - 
```

- dongguk-consumer는 종료된 상태, 현재 활성화된 멤버가 없다는 정보와 상세 정보 확인가능
- dongguk-topic 파티션 1번은 현재 오프셋은 2, 마지막 오프셋도 2, LAG는 0
- dongguk-topic 파티션 2번은 현재 오프셋은 8, 마지막 오프셋도 8, LAG는 0

**그림 6-1** LAG=0과 LAG=5의 차이

- LAG이 계속 증가하는 경우라면 컨슈머 처리가 늦어지고 있는 상황. 컨슈머나 파티셔을 늘려서 대응
- 특정 파티션에만 LAG이 증가한다면 파티션에 연결된 컨슈머에 문제가 없는지 확인 필요

# 6.2. 주키퍼의 스케일 아웃