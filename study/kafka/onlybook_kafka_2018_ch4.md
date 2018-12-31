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















