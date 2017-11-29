---
layout: post
title: "How to run mesos with HA (feat. Spark)"
date: 2017-11-25 19:31 +0900
category: mesos
tags: [mesos]
---
<p>
	<h4>1. Environment</h4>
	<ul type="circle">
		<li>CentOS 7.1</li>
		<li>Mesos 1.3.0</li>
		<li>Spark 2.2.0</li>
		<li>Zookeeper 3.4.9</li>
	</ul>
</p>
<p>
	<h4>2. Master</h4>
	일반적인 형식은 다음과 같다.
{% highlight shell %}
cd $MESOS_HOME/build
bin/mesos-master.sh 
	--ip=[ip address] 
	--port=[port] 
	--zk=[zookeeper1:port],[zookeeper2:port],zookeeper3:[port],/[zookeeper-folder-name] 
	--quorum=N 
	--cluster=[cluster-name] 
	--hostname=[ip address] 
	--advertise_ip=[ip address] 
	--advertise_port=[port] 
	--work_dir=[working-directory-path]
{% endhighlight %}

	각 parameter에 대해 간단하게 설명하면,
	<ul type="circle">
		<li>ip: 서버의 ip 주소</li>
		<li>port: mesos master에 할당할 port 주소, 웹 UI도 해당 포트로 접근한다. (default: 5050)</li>
		<li>zk: zookeeper 클러스터의 정보, 각각의 zookeeper의 주소와 포트를 작성하고 제일 마지막에 root znode를 작성한다.</li>
		<li>quorum: cluster의 consensus를 유지하기 위한 숫자. High Availability를 사용할 경우 필수 옵션! quorum의 수가 N이라면, master의 수는 반드시 N*2-1를 넘어야한다.</li>
		<li>cluster: 클러스터의 이름</li>
		<li>hostname: 호스트네임, FQDN 적어도 되고 ip 주소를 적어도 된다.</li>
		<li>advertise_ip: 외부망에서 접근하는 경우, 사용할 ip - 설정하지 않아도, 외부망에서 접근하는데는 문제가 없지만, 지속적인 error pop-up창이 뜬다.</li>
		<li>advertise_port: 외부망에서 접근하는 경우, 사용하는 port - 설정하지 않아도, 외부망에서 접근하는데는 문제가 없지만, 지속적인 error pop-up창이 뜬다.</li>
		<li>work_dir: working directory</li>
	</ul>

	Example)
{% highlight shell %}
cd $MESOS_HOME/build
bin/mesos-master.sh 
	--ip=192.168.0.41 
	--port=5050 
	--zk=192.168.0.11:2181,192.168.0.12:2181,192.168.0.13:2181,/mesos 
	--quorum=2 
	--cluster=mesos-test 
	--hostname=192.168.0.41
{% endhighlight %}

	참고) advertise_ip, advertise_port는 spark의 mesos-dispatcher와 사용이 불가능하다.
</p>
<p>
	<h4>3. Agent</h4>
	일반적인 형식은 다음과 같다.
{% highlight shell %}
cd $MESOS_HOME/build
bin/mesos-agent.sh 
	--ip=[ip address] 
	--port=[port] 
	--master=zk://[zookeeper1:port],[zookeeper2:port],[zookeeper3:port],/[zookeeper-folder-name] 
	--work_dir=[working-directory-path]
{% endhighlight %}

	각 parameter에 대해 간단하게 설명하면,
	<ul type="circle">
		<li>ip: 서버 ip 주소</li>
		<li>port: mesos agent에 할당할 port 주소 / default: 5051</li>
		<li>master: zookeeper의 주소를 적는다. 왜냐하면, mesos master leader는 zookeeper에 의해서 선발되기 때문이다.</li>
		<li>work_dir: working directory</li>
	</ul>

	Example)
{% highlight shell %}
cd $MESOS_HOME/build
bin/mesos-agent.sh 
	--ip=192.168.0.45 
	--port=5051 
	--master=zk://192.168.0.11:2181,192.168.0.12:2181,192.168.0.13:2181,/mesos 
	--work_dir=/mesos
{% endhighlight %}
</p>
<p>
	<h4>4. Spark Dispatcher</h4>
	Spark Application을 Mesos Cluster에 구동할 때, 필요한 프로세스이다. <br />
	Applicaton을 submit할 때, deploy-mode를 client mode로 하면 사용할 필요가 없지만, cluster모드로 돌릴 경우에는 필요하다.<br />
	자세한 내용은 Reference의 Spark Document 를 참고한다. <br />
	$SPARK_HOME/sbin/start-mesos-dispatcher.sh 을 이용해 구동할 수 있다.<br />

	형식은 다음과 같다.
{% highlight shell %}
$SPARK_HOME/sbin/start-mesos-dispatcher.sh 
	--host [ip address] 
	--port [port] 
	--zk [zookeeper1:port],[zookeeper2:port],[zookeeper3:port],/[zookeeper-folder-name] 
	--master mesos://[mesos-master1:port],[mesos-master2:port],[mesos-master3:port]
{% endhighlight %}

	각 parameter에 대해 간단하게 설명하면,
	<ul type="circle">
		<li>host: 서버 ip 주소</li>
		<li>port: dispatcher에 할당할 port 주소 / default: 7077</li>
		<li>zk: zookeeper의 주소를 적는다.</li>
		<li>master: mesos master의 주소를 적는다.</li>
	</ul>

	Example)
{% highlight shell %}
$SPARK_HOME/sbin/start-mesos-dispatcher.sh 
	--host 192.168.0.31 
	--port 7077 
	--zk 192.168.0.11:2181,192.168.0.12:2181,192.168.0.13:2181,/mesos 
	--master mesos://192.168.0.41:5050,192.168.0.42:5050,192.168.0.43:5050
{% endhighlight %}
</p>

Reference: [Mesos document][mesosdocument], [Spark document][sparkdocument]

[sparkdocument]:	https://spark.apache.org/docs/latest/running-on-mesos.html
[mesosdocument]:	http://mesos.apache.org/documentation/latest/

