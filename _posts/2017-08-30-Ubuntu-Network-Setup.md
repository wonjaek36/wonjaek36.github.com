---
layout: post
title: "How to Network Setup on Ubuntu"
date: 2017-08-30 02:20 +0900
category: "utils"
tags: [ubuntu, linux]
---

우분투 네트워크 설정 방법<br />
우분투 버전은 16.04<br />

우선 /etc/network/interfaces 파일을 연다. (관리자 권한으로 연다.)<br />
{% highlight shell %}
cd /etc/network
vim ./interfaces
{% endhighlight %}

그리고 다음과 같이 작성한다.<br />

{% highlight shell %}
auto enp2s0
iface enp2s0 inet static
	address 192.168.0.10
	netmask 255.255.255.0
	network 192.168.0.0
	gateway 192.168.0.1
dns-nameservers 8.8.8.8
{% endhighlight %}

여기서 enp2s0는 network eth 카드이름이다.<br />
{% highlight shell %}
ifconfig -a
{% endhighlight %}
명령어로 찾을 수 있다.<br />

그 다음 address, netmask, network, gateway 등은 맞게 구성하면  된다.<br />
