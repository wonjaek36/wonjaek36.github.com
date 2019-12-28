---
layout: post
title: "How to Install Elasticsearch(Cluster)"
date: 2018-01-09 17:23 +0900
category: [database, elasticsearch]
tags: [database, elasticsearch, elk]
---

### Installation Environment
* Ubuntu 16.04 Server
* ElasticSearch 6.1.1

### Download Elasticsearch File
{% highlight shell %}
wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-6.1.1.tar.gz
{% endhighlight %}

### Unzip tar.gz File
{% highlight shell %}
tar -zxvf elasticsearch-6.1.1.tar.gz
{% endhighlight %}

### (Optional) Add soft link 
{% highlight shell %}
ln -s elasticsearch-6.1.1 /path/to/elasticsearch
{% endhighlight %}

### (Optional) Add Environment Variable in .bash_profile 
{% highlight shell %}
echo "export ES_HOME=\"/path/to/elasticsearch\"" >> ~/.bash_profile
echo "PATH=\"$PATH:$ES_HOME/bin\"" >> ~/.bash_profile
{% endhighlight %}

### Edit Config File
{% highlight shell %}
vim $ES_HOME/config/elasticsearch.yml
Editing...
{% endhighlight %}

#### Config 파일에서 필수 설정 값들
* cluster.name: cluster 이름 / 같은 Cluster 이름을 갖는 Node들이 서로 연결된다.
* node.name: node 이름 / 같은 Cluster 내에 노드들은 고유한 이름을 가져야한다.
* path.data: data 경로
* path.logs: log 경로
* network.host: 자신의 네트워크 주소
* discovery.zen.ping.unicast.hosts: cluster끼리 연결할 때, 여기 연결된 노드에게 메시지를 보내고 연결된다.(당연히 같은 Cluster의 노드들이여야한다.)
* discovery.zen.minimum_master_nodes: split brain을 막기위한 노드 수, master 노드 / 2 + 1로 설정하는 것이 좋다. (이보다 큰 것은 괜찮지만, 적으면 split brain이 발생할 수 있다. - 두 그룹의 마스터가 consensus를 유지하지 못하는 현상)