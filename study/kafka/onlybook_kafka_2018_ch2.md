# 카프카 용어정리
- 카프카 : 아파치 프로젝트 애플리케이션 이름. 클러스터 구성이 가능하며 카프카 클러스터라고 부름 
- 브로커 : 카프카 애플리케이션이 설치되어 있는 서버 또는 노드
- 토픽 : 프로듀서와 컨슈머들이 카프카로 보낸 자신들의 메시지를 구분하기 위한 이름으로 사용함. 많은 수의 프로듀서, 컨슈머들이 동일한 카프카를 이용하게 된다면 메시지들이 서로 뒤섞여 각자 원하는 메시지를 얻기가 어려워짐. 그래서 토픽이라는 이름으로 구분해서 사용
- 파티션 : 병렬처리가 가능하도록 토픽을 나눌수 있고 많은 양의 메시지를 처리하기 위해 파티션의 수를 늘릴수 있다. 
- 프로듀서 : 메시지를 생산하여 브로커의 토픽 이름으로 보내는 서버 또는 애플리케이션을 말함
- 컨슈머 : 브로커의 토픽 이름으로 저장된 메시지를 가져가는 서버 또는 애플리케이션을 말함


# 주키퍼
- 분산 애플리케이션을 위한 코디네이션 시스템
- 용어
  - 지노드 : 데이터를 저장하기 위한 공간
  - 앙상블 (클러스터) : 호스트 세트, 과반수 기존의 노드 체크
- zoo.cfg
```
tickTime=2000
initLimit=10
syncLimit=5
dataDir=/home1/irteam/data
clientPort=2181
server.1=dev-dongguk-zk001-ncl:2888:3888
server.2=dev-dongguk-zk002-ncl:2888:3888
server.3=dev-dongguk-zk003-ncl:2888:3888
```

- zookeeper-server.service
```
[Unit]
Description=zookeeper-server
After=network.target

[Service]
Type=forking
User=irteam
Group=irteam
SyslogIdentifier=zookeeper-server
WorkingDirectory=/home1/irteam/apps/zookeeper
Restart=always
RestartSec=0s
ExecStart=/home1/irteam/apps/zookeeper/bin/zkServer.sh start
ExecStop=/home1/irteam/apps/zookeeper/bin/zkServer.sh stop

[Install]
WantedBy=multi-user.target
```

- 주키퍼의 세션 타임아웃은 노드에 동작중인 자바 애플리케이션의 풀GC등의 시간을 고려해서 충분한 시간(3초 이상)을 잡아주는게 좋다.  

# 카프카
- 주키퍼의 경우 과반수 방식으로 운영되지만 카프카의 경우 과반수가 아닌 자유롭게 운영이 가능하다. 안정적인 운영을 위해서 리얼서버에서는 주키퍼와 카프카를 별도의 서버에서 운영하는게 좋다. 
- 카프카 설정 파일내 주키퍼 설정은 주키퍼 앙상블내 서버 모두 입력해야 한다. 

## 카프카 환경설정
- 주키퍼 앙상블 세트를 여러개의 애플리케이션에서 공용으로 사용하려면 주키퍼의 최상위 경로를 사용하지 않고 지노드를 구분해서 사용하면 된다. 
  - 주키퍼 정보를 입력할때 호스트 이름과 포트 정보 입력 후 마지막에 지노드 이름을 추가하면 된다.
- 카프카 환경설정 server.properties파일
  - broker.id
  - log.dirs
  - zookeeper.connect
  - 표 2-1. 카프카 주요옵션

## 카프카 실행
- 백그라운드 실행을 위해 & 나 -daemon옵션을 사용하자
- systemd관련 kafka-server.service 파일
``` 
[Unit]
Description=kafka-server
After=network.target

[Service]
Type=simple
User=irteam
Group=irteam
SyslogIdentifier=kafka-server
WorkingDirectory=/home1/irteam/apps/kafka
Restart=always
RestartSec=0s
ExecStart=/home1/irteam/apps/kafka/bin/kafka-server-start.sh /home1/irteam/apps/kafka/config/server.properties
ExecStop=/home1/irteam/apps/kafka/bin/kafka-server-stop.sh

[Install]
WantedBy=multi-user.target
```

# 카프카 상태확인

## TCP 포트확인
- 주키퍼 기본포트 (2181), 카프카 기본포트 (9092)
- netstat -ntlp | grep 2181 (또는 9092)

## 주키퍼 지노드를 이용한 카프카 정보 확인
- 주키퍼 CLI 를 사용 (zkCli.sh 명령어)

## 카프카 로그 확인

# 카프카 시작하기

## 카프카 토픽 생성

```
bin/kafka-topics.sh --zookeeper dongguk-zk001:2181,dongguk-zk002:2181,dongguk-zk003:2181/dongguk-kafka --replication-factor 1 --partitions 1 --topic dongguk-topic --create
```

```
bin/kafka-topics.sh --zookeeper dongguk-zk001:2181,dongguk-zk002:2181,dongguk-zk003:2181/dongguk-kafka --topic dongguk-topic --delete
```

```
bin/kafka-console-producer.sh --broker-list dongguk-kafka001:9092,dongguk-kafka002:9092,dongguk-kafka003:9092 --topic dongguk-topic
```

```
bin/kafka-console-consumer.sh --bootstrap-server dongguk-kafka001:9092,dongguk-kafka002:9092,dongguk-kafka003:9092 --topic dongguk-topic --from-beginning
```

# 참고사이트
- http://yookeun.github.io/kafka/2018/07/01/kafka-install/
