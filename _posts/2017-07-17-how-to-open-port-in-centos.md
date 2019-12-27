---
layout: post
title: "How to open port in CentOS 7"
date: 2017-07-17 16:40 +0900
category: [linux, centos]
tags: [linux, centos, network, firewall]
---

서버들을 운영하다보면, 방화벽을 열 때가 있다. <br />
CentOS는 우분투와 다르므로 정리해본다.(7에서만 적용된다. 6에선 안 됨.) <br />
{% highlight shell %}
firewall-cmd --zone=public --add-port=[port number]/[tcp or udp] --permanent
firewall-cmd --reload
{% endhighlight %}

하면 된다. [port number]에는 포트 번호를 넣어주면 되고,<br />
[tcp or udp]에는 "tcp" 또는 "udp" 둘 중 원하는 프로토콜을 넣으면 된다.

이미 추가한 firewall list을 보고 싶다면
{% highlight shell %}
firewall-cmd --list-all
{% endhighlight %}
로 확인할 수 있다.

