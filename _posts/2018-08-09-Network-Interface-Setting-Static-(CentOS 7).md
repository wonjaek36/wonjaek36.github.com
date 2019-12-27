---
layout: post
title: "Network Interface Setting - Static (CentOS 7)"
date: 2018-08-09 14:29 +0900
category: [linux, centos]
tags: [linux, centos, network]
---

Environment: CentOS 7(1804) Minimal  
Required: Root privilege, text editor  

### 1. ip addr로 network interface 확인 
  
{% highlight shell %}
$ ip addr
1: lo : <LOOPBACK,UP,LOWER_IP> mtu 65536 qdisc noqueue state ...
	link/loopback 00:00:00:00:00:00 ...
	inet 127.0.0.1/8 scope host lo
	...
2: enp0s25: <BROADCAST,MULTICAST,UP,...>...
	link/ether ...
	inet ...
{% endhighlight %} 

여기서는 내 network interface가 enp0s25임을 확인할 수 있다.  
2번에서 ifcfg-\<network interface card 이름\>으로 사용하면 된다.
lo는 loopback.   

### 2. Load network interface script  

{% highlight shell %}
vim /etc/sysconfig/network-script/ifcfg-enp0s25
{% endhighlight %}

### 3. Setting

{% highlight shell %}
...
BOOTPROTO=static
...
IPADDR=192.168.0.100
NETMASK=255.255.255.0
GATEWAY=192.168.0.1
DNS1=8.8.8.8
{% endhighlight %}