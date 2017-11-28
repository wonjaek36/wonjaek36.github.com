---
layout: post
title: "Introduction to Apache Mesos"
date: 2017-11-25 19:31 +0900
category: mesos
---

<h3>1. Overview</h3>
Mesos는 Berkely 대학에서 개발한 DC/OS이다. DC/OS는 Data-Center Operating System으로 Data-Center 내 다수의 컴퓨터를 마치 하나의 컴퓨터처럼 사용할 수 있도록, 자원(cpu, memory, disk 등)을 추상화한다.<br />
Data Center에서 동작하는 application들은 마치 운영체제가 제공하듯이, Mesos가 제공하는 자원을 이용해 동작한다. <br />
<br />
Mesos는 다음과 같은 특징을 갖고 있다.
<ul type="circle">
    <li>데이터 센터 내 PC 당 불특정한 service 또는 application을 여러 개 돌릴 수 있다. (애플리케이션의 자원소비 및 각 PC의 자원 상태에 따라서 조정된다.)</li>
    <li>PC 문제로 service가 Fail될 경우, 다른 PC에서 해당 service를 구동한다.(Fail-over)</li>
    <li>여러 scheduler를 사용하는 application을 구동할 수 있다.(hadoop, spark, MPI, marathon)</li>
</ul>
<br />
<h3>2. Architecture</h3>

<img src="{{ site.url }}/assets/mesos_architecture.png" class="center-image" />
<br />
Mesos Cluster는 Master, Agent 그리고 Zookeeper로 구성된다.<br />
<br />
Master는 프레임워크들이 주는 job들을 관리(스케줄링)하고, 요청한 자원들을 할당한다. <br />
Agent는 가용 가능한 자원을 Master에 알리고, Master가 요청하는 작업들을 수행한다. <br />
Zookeeper는 마스터의 Leader Election을 위해 사용한다.<br />
<br />
다음 그림을 통해 프레임워크, Master, Agent들이 어떻게 일을 진행하는지 알 수 있다.<br />
<img src="{{ site.url }}/assets/mesos_working.png" class="center-image" />

<ol type="1">
	<li> Agent(Slave)가 Master와 연결되면서 가용가능한 자원의 정보를 전달한다.</li>
	<li> Master가 Framework에게 현재 사용가능한 자원의 정보를 전달한다. </li>
	<li> Framework는 Master에게 Task에 필요한 자원을 요청한다. </li>
	<li> Master는 Framework가 전달해준 정보를 Agent에게 재 전달한다.</li>
</ol>

Reference: [Mesos Paper][mesospaper]

[mesospaper]: 	https://people.eecs.berkeley.edu/~alig/papers/mesos.pdf