---
layout: post
title: "카프카 아키텍쳐 소개"
date: 2017-08-23 10:45 +0900
category: kafka
---

카프카는 링크드인에서 개발한 Message Queue 시스템이다. 실시간 로그 처리에 사용되는 아키텍쳐로 대용량 처리에 적합하다.<br />

카프카는 Producer, Consumers, Cluster(Brokers)로 구성된다.<br />
1) Producer는 데이터를 생성하여 카프카로 전송한다.<br />
2) Consumer는 Cluster로부터 데이터를 전달받는다.<br />
3) Cluster는 Producer가 전송한 데이터를 Consumer가 소비할 때까지 보관한다. <br />
* (정확하지는 않습니다. 밑에 자세히 설명하겠습니다.)
카프카는 Pub/Sub 모델을 이용하여 Producer는 자신이 생성/전달하는 데이터의 Topic를 설정할 수 있게 하였고, Consumer는 자신이 원하는 Topic 데이터만 받을 수 있게 설정할 수 있다.

({{ site.url }}/assets/simple_kafka_architecture.jpg)
(Simple Kafka Architecture)
