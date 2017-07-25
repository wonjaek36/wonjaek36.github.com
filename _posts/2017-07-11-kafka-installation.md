---
layout: post
title: "Installation Kafka"
date: 2017-07-11 14:48 +0900
category: kafka
---

Step 1. Download kafka
{% highlight shell %}
curl http://mirror.navercorp.com/apache/kafka/<version>/kafka_<version>.tgz > kafka_<version>.tgz
tar -xzf kafka_<version>.tgz
cd kafka_<version>
{% endhighlight %}

Step 2. Run zookeeper
이미 설치해둔 주키퍼가 있으면 넘어가도 상관없다.<br >
없다면 kafka에 내장된 주키퍼를 사용한다.

{% highlight shell %}
./bin/zookeeper-server-start.sh ./config/zookeeper.properties
{% endhighlight %}

여기서는 주키퍼는 Single Node로만 사용한다.

Step 3. Create Topic
토픽 생성을 생성한다.
{% highlight shell %}
./bin/kafka-topic.sh --create zookeeper localhost:2181 --replication-factor 1 partitions 1 --topic <topic name>
{% endhighlight %}
replication-factor는 1로 한다.(처음에는 single node kafka로)<br />
partition은 병렬처리를 위한 것이니까, 일단은 1로 설정한다.

* 존재하지 않는 토픽이 들어오면, 자동으로 토픽을 생성한다. (replication, partitions default 옵션은 아직 잘 모른다...)
* zookeeper의 ip와 port는 default를 사용함

Step 4. Check topic list
{% highlight shell %}
./bin/kafka-topic.sh --list --zookeeper localhost:2181
{% endhighlight %}

Step 5. Describe topic
{% highlight shell %}
./bin/kafka-topic.sh --describe zookeeper localhost:2181 --topic  <topic name>
Topic: <topic name> PartitionCount:1 ReplicationFactor: 1 Configs:
   Topic: <topic name> Partition: 0  Leader: 0(broker id) Replica: 0 Isr: 0
{% endhighlight %}
로 결과가 나온다. 

각각의 파라미터에 대해 간략하게 설명하면,<br />
PartitionCount: 파티션의 개수<br />
ReplicationFactor: Replica의 개수, 지금은 Single Node이므로 1밖에 되지 않는다.<br />
Partition: partition 번호<br />
Leader: 현재 이 Topic Leader(broker id로 표현된다.)<br />
Replica: 이 topic 데이터를 갖는 노드들<br />
Isr: leader와 싱크를 맞춘 노드들<br />

Step 6. Producer, Consumer
{% highlight shell %}
./bin/kafka-console-producer.sh --broker-list localhost:9092 --topic <topic name>
./bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic <topic name> --from-beginning
{% endhighlight %}

** Step 7. Multi-broker cluster
kafka server.properties file 중 4가지 파라미터를 바꿔준다.<br />
broker.id: broker의 unique한 아이디(겹치면 안 된다.)<br />
log.dir: log directory path <br />
zookeeper.connect: <zookeeper_ip:zookeeper_port><br />
listeners: (PLAINTEXT://ip:port or SSL, CLIENT, etc...)<br />
로 설정하고 각 kafka를 구동하면 된다. 

Reference: [apache kafka] 

[apache kafka]: https://kafka.apache.org
