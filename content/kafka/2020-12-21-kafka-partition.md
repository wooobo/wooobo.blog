---
emoji: 🔮
title: 도대체 카프카 토픽 파티션?
date: '2020-12-21 00:00:00'
author: wooobo
tags: kafka
categories: kafka
---


# 카프카 파티션?
카프카는 초당 수백만개의 메시지를 전송하고 처리를 제공하는 분산 스트리밍 플랫폼입니다.  
처리성능을 높이기 위해 토픽 저장을 병렬화 할 수 있습니다.    
그러한 방법은 브로커에 파티션을 나누는 것입니다.  
하지만, 파티션은 늘리는것은 가능하지만 줄이는 것은 안되기 때문에 주의 해야합니다.  
파티션 줄이고 싶다면 토픽을 삭제밖에 방법이 없습니다.   
ex)  
1개의 파티션이 1분에 10개의 메시지를 받을 수 경우를 가정해보겠습니다.  
프로듀서에서 1분에 50개의 메세지를 보낸다면,  
토픽의 파티션을 5개로 늘려 처리 성능을 높여서 병목현상을 막을수 있을것입니다.

## 커맨드로 따라 해보기
```shell
# 주키퍼 실행
$ bin/zookeeper-server-start.sh config/zookeeper.properties

# 카프카 실행 
$ bin/kafka-server-start.sh config/server.properties

# 생성된 토픽 리스트 가져오기
$ bin/kafka-topics.sh --zookeeper localhost:2181 --list
> __consumer_offsets

# 파티션 3개 토픽 생성하기
$ bin/kafka-topics.sh --zookeeper localhost:2181 --topic partition-first-topic --create --partitions 3 --replication-factor 1
> Created topic partition-first-topic.

# 다시 토픽 리스트 확인하기
$ bin/kafka-topics.sh --zookeeper localhost:2181 --list
> __consumer_offsets
> partition-first-topic ## 토픽이 생성된것을 확인하실 수 있습니다.

# 토픽 기본정보 확인하기
$ bin/kafka-topics.sh --zookeeper localhost:2181 --describe --topic partition-first-topic
> Topic: partition-first-topic	PartitionCount: 3	ReplicationFactor: 1	Configs:
    Topic: partition-first-topic	Partition: 0	Leader: 0	Replicas: 0	Isr: 0
    Topic: partition-first-topic	Partition: 1	Leader: 0	Replicas: 0	Isr: 0
    Topic: partition-first-topic	Partition: 2	Leader: 0	Replicas: 0	Isr: 0
=> 파티션 3개가 생성된것을 확인할 수 있습니다.
   ReplicationFactor는 토픽을 복제할 브로커 수입니다. 현재는 브로커 1개로만 테스트 하고 있으므로 "1"입니다.
   
# 이제 토픽을 넣어 보겠습니다.
# 파티션이 3개이니 3개로 분산해서 저장되는것을 예상하고 진행해보겠습니다.
# 토픽 전송
$ bin/kafka-console-producer.sh --topic partition-first-topic --bootstrap-server localhost:9092
>send 01
>send 02
>send 03
>send 04
>send 05

# 5개 메시지를 전송 하였습니다.

# 컨슈머로 토픽을 받아 보겠습니다. ("--property print.partition=true" 추가하여 파티션을 출력합니다) 
$ bin/kafka-console-consumer.sh --topic partition-first-topic --from-beginning --property print.partition=true --bootstrap-server localhost:9092
> Partition:1	send 01
> Partition:1	send 03
> Partition:1	send 05
> Partition:2	send 04
> Partition:0	send 02

# 보시면 아시겠지만 파티션 0,1,2로 분산되어 저장된것을 확인하실 수 있습니다.
# 눈에 띄는것이, 토픽을 가져올때는 순서대로 가져오지 않는다는걸 확인 할 수 있습니다.
```  

# 마무리
1. 토픽에 파티션을 추가하여 토픽을 생성해보았습니다.
2. 토픽 전송 후 파티션을 확인하였을때 분산되어 저장되는것을 확인 할 수 있었습니다.
3. 토픽을 가져올때는 순서대로 가져오지 않는다는걸 확인 하였습니다.