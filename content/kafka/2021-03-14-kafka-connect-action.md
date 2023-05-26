---
emoji: 🔮
title: Kafka connect File Source 알아보기 & 실행
date: '2021-03-14 00:00:00'
author: wooobo
tags: kafka
categories: kafka
---

# Kafka Connect를 활용하여 File Source 해보기

1. kafka, zookeeper가 실행되어있어야 합니다.(카프가가 없으시다면 공식 홈페이지에서 [다운로드](https://www.apache.org/dyn/closer.cgi?path=/kafka/2.7.0/kafka_2.13-2.7.0.tgz))
   (처음 이시라면 [참고](https://kth12.github.io/kafka-quick-start/))
2. kafka/config 폴더를 열어보면 아래의 파일들이 존재합니다.
   connect-file-sink.properties     
   connect-standalone.properties
   위 파일들을 확용하여 File을 감시해보겠습니다.

---

### config/connect-file-source.properties

```shell
name=local-file-source
connector.class=FileStreamSource
tasks.max=1
file=../example/source.txt
topic=connect-test
```

위 설정 파일은, connect 이름 / connect 라이브러리(connect class) / Source Topic 이름  등등을 설정 할수 있습니다.

### config/connect-standalone.properties

```shell
bootstrap.servers=localhost:9092
key.converter=org.apache.kafka.connect.json.JsonConverter
value.converter=org.apache.kafka.connect.json.JsonConverter
key.converter.schemas.enable=true
value.converter.schemas.enable=true

offset.storage.file.filename=/tmp/connect.offsets
offset.flush.interval.ms=10000
```

카프카 서버 주소 설정 / Source key, value 타입 지정 / 감시할 파일 이름 등등을 설정 할 수 있습니다.

## 실행 해보기

기본 설정 파일에 대해 알아보았고 실행 예제를 알아보겠습니다.

실행에 앞서 감시할 파일을 생성하여 줍니다.

저같은 경우 "source.txt" 라는 파일을 생성해주었습니다.

### 실행 커맨드

```bash
$ bin/connect-standalone.sh config/connect-standalone.properties config/connect-file-source.properties

INFO Kafka startTimeMs: 1615649461180 (org.apache.kafka.common.utils.AppInfoParser:121)
INFO [Producer clientId=connector-producer-local-file-source-0] Cluster ID: ovrhH6KaQnOfZ0SseIBV6g (org.apache.kafka.clients.Metadata:279)
INFO WorkerSourceTask{id=local-file-source-0} Source task finished initialization and start (org.apache.kafka.connect.runtime.WorkerSourceTask:233)
INFO Created connector local-file-source (org.apache.kafka.connect.cli.ConnectStandalone:112)
 
# 위 출력 로그처럼 정상적으로 실행이 되었다면 파일을 수정해보겠습니다.

$ cat >> source.txt
	hi
	hello

# 감시 파일(source.txt)에 내용을 입력 하였습니다.

$ bin/kafka-console-consumer.sh --topic connect-test --from-beginning  --bootstrap-server localhost:9092
{"schema":{"type":"string","optional":false},"payload":"hi"}
{"schema":{"type":"string","optional":false},"payload":"hello"}

# 파일에서 입력했던 내용들을 컨슘되는것을 확인할 수 있습니다.
# 터미널 창을 하나 더 열어 내용을 추가적으로 넣어서 실시간 동기화 되는지 확인해보겠습니다.
```

### 실행 화면

![img.png](../../assets/images/kafkaconnectgif.gif)

---

### 마무리

가장 간단하게 카프카 커넥트 예제에 대해서 알아 보았습니다.

카프카 커넥트는 여러 플러그인을 붙여 다양한 데이터 소스를 커넥트 할 수 있습니다.

예로, debezium 오픈소스 플러그인을 활용하면

mysql, mongoDB, PostgreSQL,Oracle,SqlServer 등 데이터 베이스를 감시하고 데이터를 Source 할 수 있습니다.