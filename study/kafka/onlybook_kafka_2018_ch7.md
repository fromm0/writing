# 7.1. 카프카를 활용한 데이터 흐름도

- 아파치 나이파이 (nifi.apache.org)
- 엘라스틱 파일비트 (www.elastic.co/products/beats/fileBeat)
- 엘라스틱 엘라스틱서치 (www.elastic.co/products/elasticsearch)
- 엘라스틱 키바나 (www.elastic.co/products/kibana)

**그림 7-1** 카프카를 활용한 데이터 흐름도 예제1

# 7.2. 파일비트를 이용한 메시지 전송

```
./kafka-topics.sh \
--zookeeper dongguk-zk001:2181,dongguk-zk002:2181,dongguk-zk003:2181/dongguk-kafka \
--replication-factor 2 \
--partitions 3 \
--topic peter-topic \
--create
```

## 7.2.1. 파일비트 설치

- 파일비트는 엘라스틱에서 제공하는 경량 데이터 수집기
- 파일비트 설치 페이지 
  - https://www.elastic.co/guide/en/beats/filebeat/current/setup-repositories.html
- yum 인증키 등록
  - `rpm --import https://packages.elastic.co/GPG-KEY-elasticsearch`
- /etc/yum.repos.d/elastic.repo 파일 생성
```
[elastic-6.x]
name=Elastic repository for 6.x packages
baseurl=https://artifacts.elastic.co/packages/6.x/yum
gpgcheck=1
gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch
enabled=1
autorefresh=1
type=rpm-md
```
- yum명령을 사용해 설치
  - `yum -y install filebeat`

## 7.2.2. 파일비트 설정

- 파일비트 설정 파일 (/etc/filebeat/filebeat.yml)

**예제 7-2** 파일비트 환경설정 filebeat.yml 파일

- 파일비트 실행
  - `systemctl start filebeat.service`
- 파일비트 상태 체크
  - `systemctl status filebeat.service`

## 카프카 토픽의 메시지 유입 확인

```
./kafka-console-consumer.sh \
--bootstrap-server dongguk-kafka001:9092,dongguk-kafka002:9092,dongguk-kafka003:9092 \
--topic peter-log \
--max-messages 10 \
--from-beginning
```

# 7.3. 나이파이를 이용해 메시지 가져오기

- 나이파이는 카프카의 peter-log 토픽으로부터 메시지를 가져오는 컨슈머

**그림 7-3** 나이파이를 이용해 peter-log 메시지 가져오기























