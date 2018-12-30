# 3.1. 카프카 디자인의 특성

## 3.1.1. 분산 시스템

- 같은 역할을 하는 여러대의 서버로 이뤄진 서버 그룹을 분산 시스템
- 분산시스템의 장점
  - 단일 시스템보다 더 높은 성능을 얻을 수 있다. 
  - 분산 시스템 중 하나의 서버 또는 노드등이 장애가 발생하면 다른 서버 또는 노드가 대신 처리한다.
  - 시스템 확장이 용이하다.

## 3.1.2. 페이지 캐시

- 빠른 엑세스를 하기 위해 OS의 페이지 캐시를 이용하도록 디자인 되었음
- 페이지 캐시는 서로 공유하기 때문에 하나의 시스템에 카프카를 다른 애플리케이션과 함께 실행하는 것은 권장하지 않음

## 3.1.3. 배치 전송 처리

- 작은 IO가 여러번 일어나는 것보다는 배치로 한번에 IO가 발생하는 것이 성능에 좋다. 

# 3.2. 카프카 데이터 모델

## 3.2.1. 토픽의 이해

- 토픽은 데이터를 구분하기 위한 단위
- 토픽 이름은 249자 미만, 영문/숫자/./_/- 를 조합해서 명명가능

## 3.2.2. 파티션의 이해

- 파티션은 토픽을 분할한 개념으로 토픽이 한번에 메시지를 받을수 있는 수를 증가시킴
- 파티션을 늘릴 경우 단점
  - 파일 핸들러의 낭비 
    - 각 파티션은 브로커의 디렉토리와 매핑
    - 저장되는 데이터마다 2개의 파일(인덱스와 실제데이터)이 존재
    - 모든 디렉토리의 파일들에 대해 파일핸들을 열게 되어 파티션 수 만큼 파일 핸들수가 증가되어 리소스가 낭비됨
  - 장애 복구 시간 증가
    - 카프카는 높은 가용성을 위해 리플리케이션을 지원
    - 리더와 팔로우 노드의 전환
    - 컨트롤러의 페일오버는 자동으로 동작하지만 새 컨트롤러가 초기화하는 동안 주키퍼에서 모든 파티션의 데이터를 읽어야 해서 파티션이 많을 수록 페일오버에 오랜시간이 걸릴수 있음
- 적절한 파티션 수
  - 프로듀서와 컨슈머 각각의 처리량을 고려해서 정해야 함
  - 카프카에서 파티션을 늘리는 건 언제나 가능하지만 삭제는 토픽을 삭제하는 방법밖에 없음
  - 카프카는 브로커랑 최대 2000개 정도의 파티션을 권장

## 3.2.3. 오프셋과 메시지 순서

오프셋은 각 파티션마다 메시지가 저장되는 위치
오프셋은 파티션내 유일한 값이고 순차적으로 증가 (64비트 정수)
컨슈머는 오프셋 순서대로 데이터를 가져갈수 있음

# 3.3. 카프카의 고가용성과 리플리케이션

## 3.3.1. 리플리케이션 펙터와 리더, 팔로워의 역할

카프카는 토픽 자체를 리플리케이션하지는 않고 토픽의 파티션을 리플리케이션한다. 

리플리케이션 팩터의 설정값은 server.properties에서 다음의 값이다. 

```
default.replication.factor = 2
```

토픽의 리플리케이션
- 원본 (리더) : 모든 읽기/쓰기 작업 처리
- 복제본 (팔로워) : 리더에 대한 리플리케이션만 처리

```
./kafka-topics.sh --zookeeper dev-dongguk-zk001-ncl:2181,dev-dongguk-zk002-ncl:2181,dev-dongguk-zk003-ncl:2181/dongguk-kafka --replication-factor 2 --partitions 1 --topic dongguk-topic --create
```

```
./kafka-topics.sh --zookeeper dev-dongguk-zk001-ncl:2181,dev-dongguk-zk002-ncl:2181,dev-dongguk-zk003-ncl:2181/dongguk-kafka --topic dongguk-topic --describe
```

```
Topic:dongguk-topic     PartitionCount:1        ReplicationFactor:2     Configs:
        Topic: dongguk-topic    Partition: 0    Leader: 3       Replicas: 3,1   Isr: 3,1
# leader의 3 (2번 파티션)이 리더, Replicas의 3, 1중에서 3이 리더이니 1이 팔로워
```

|토픽 사이즈|리플리케이션 팩터|카프카 클러스터내 필요 저장소 크기|
|100GB|1|100GB|
|100GB|2|200GB|
|100GB|3|300GB|


리플리케이션 사용시 저장소 크기 뿐 아니라 지속적인 상태 체크를 통한 리소스 사용을 줄이려면 모든 토픽에 팩터를 3으로 하기 보다는 토픽 성격에 따라 2, 3을 선별적으로 사용하는게 좋음

## 3.3.2. 리더와 팔로워의 관리

- ISR (In Sync Replica)
  - 팔로워가 리더로부터 제대로 리플리케이션을 할수 없어서 데이터 정합성이 맞지 않을 경우를 방지하기 위해  사용
  - 리플리케이션되고 있는 리플리케이션 그룹
  - ISR 그룹내 구성원만이 리더가 될수 있음

# 3.4. 모든 브로커가 다운된다면

모든 브로커가 다운된 경우 선택가능한 방법 (server.properties파일의 unclean.leader.election.enable설정값으로 제어)

- 마지막 리더가 살아나기를 기다린다. (false)
  - 장애복구 시간이 길어질수 있다. 
  - 메시지 손실은 없다. 
- ISR에서 추방되었지만 먼저 살아나면 자동으로 리더가 된다. (true)
  - 장애복구 시간이 짧을수 있다. 
  - 메시지 손실이 발생할수 있다. 

# 3.5. 카프카에서 사용하는 주키퍼 지노드 역할

```
[irteam@dev-dongguk-zk001-ncl bin]$ ./zkCli.sh
... 중략
[zk: localhost:2181(CONNECTED) 0] ls /
[zookeeper, dongguk-kafka]
[zk: localhost:2181(CONNECTED) 1] ls /dongguk-kafka
[cluster, controller_epoch, controller, brokers, admin, isr_change_notification, consumers, log_dir_event_notification, latest_producer_id_block, config]
```

- /dongguk-kafka/controller
  - 클러스터내 브로커 중 하나를 컨트롤러로 선정, 브로커 레벨의 실패를 감지해서 모든 파티션의 리더 변경을 책임짐
- /dongguk-kafka/brokers
  - 브로커에 관련된 정보
- /dongguk-kafka/consumers
  - 컨슈머 관련 정보
- /dongguk-kafka/config
  - 토픽의 상세 설정 정보
