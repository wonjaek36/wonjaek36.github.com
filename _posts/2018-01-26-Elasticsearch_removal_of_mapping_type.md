---
layout: post
title: "Elasticsearch removal of mapping type"
date: 2018-01-26 11:30 +0900
category: [database, elasticsearch]
tags: [database, elasticsearch]
---

**Elasticsearch가 7.0 버전부터는 공식적으로 \_type을 지원하지 않기로 결정하였다.** 6.0 버전에서는 deprecated 처리되었으며, \_type을 사용할 경우 warn 메시지를 내보낸다.

Elasticsearch 버전 5.0에서 데이터는 document, type, index. 계층으로 관리한다. 단순하게 말하면, index는 기존 RDBMS의 database와 비슷한 개념으로, 여러 타입들에 대하여 mapping 정보를 가지고 있다. 또한, 데이터가 몇 개의 shard에 나눠지는지 또는 replication은 어떻게 설정되는지 등, 데이터를 어떻게 관리할지에 대한 정보를 담고 있다. type은 RDBMS의 table과 비슷한 개념으로 이해하면 쉽다. index 내에 여러 종류의 데이터가 있을 때, 데이터를 구분하는데 사용된다. 예를 들어서, 날씨와 미세먼지 데이터를 같은 index에 저장하면 이 둘은 \_type으로 구분할 수 있다.

하지만, elasticsearch의 type은 RDBMS의 table과 다른 점이 있다. *같은 index 내에서 \_type이 다르더라도 같은 이름의 field를 갖는다면, 그 정보는 내부적으로 하나의 field로 관리된다.*([Lucene](https://lucene.apache.org/core/)의 특징). 만약에 날씨와 미세먼지가 같이 Date 필드를 갖고 있으면, 두 Date 필드는 하나로 mapping 된다는 것이다. (table은 각자 따로 저장된다.) 날씨 타입에서 Date 데이터를 바꾸거나 타입을 바꾸면, 문제가 될 수 있다.
공통 field가 별로 없는 데이터를 하나의 index에 넣는 것은 **효율적으로 document를 압축하지 못 하기 떄문에, 권장하지도 않는다.** ([Lucene](https://lucene.apache.org/core/) 특징)

위의 문제 때문에, elasticsearch는 type을 제거하기로 하였고, 대안으로 두 가지 방식을 제시한다.
1. type별로 index 생성
+ type별로 index를 생성하면, 기존의 방식보다 [Lucene]가 데이터를 압축이 잘 되는 장점이 있다.
2. custom type
- 기존에 사용하던 \_type을 사용하지 않고, 각 document별로 docType과 같이 type을 구분할 수 있는 field를 넣어준다. 


Reference

* [Elasticsearch Document](https://www.elastic.co/guide/en/elasticsearch/reference/master/removal-of-types.html)