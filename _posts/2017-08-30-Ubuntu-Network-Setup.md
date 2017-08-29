---
layout: post
title: "How to Network Setup on Ubuntu"
date: 2017-08-30 02:20 +0900
category: "utils"
---

이번에 실험을 위해 클러스터를 구성하면서 열심히 우분투를 설치하였다.<br />
근데, 네트워크 설정에서 헛...하고 막히네.<br />
너무 오랜만에 해서 그런가. 금붕어가 울고갈 이 기억력은 어떻게 할 수 없나보다.<br />
<br />
그러므로 이 참에 열심히 웹서핑한 기록을 남기도록 한다.<br />
일단 버전은 16.04에서 진행하였다.<br />

우선 /etc/network/interfaces 파일을 연다. (필요하다면 관리자 권한으로 연다.)<br />
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
dns-nameservers는 구글 것을 그냥 사용했다.<br />
