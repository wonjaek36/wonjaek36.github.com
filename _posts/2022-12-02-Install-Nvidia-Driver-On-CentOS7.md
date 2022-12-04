---
layout: post
title: "Install Nvidia-Driver on CentOS 7"
date: 2022-12-02 11:39+0900
tags: [centos, linux]
---

## 1. Introduction
레거시 시스템인 CentOS 7의 Nvidia-Driver를 업데이트하려고, 구글을 검색하면 아득한 결과들이 나온다.  
어렵기도 어렵거니와 이해하더라도, 장치를 파악하고 직접 nvidia driver를 받아서 설치하라고 하기 때문에 귀찮기 그지 없다.  

(꽤 오래 되었지만) 최근에 Nvidia가 패키지 레포로 RHEL 계열도 지원하여, CentOS 7으로도 시도했고, 성공했다!  
나중을 위해 기록해 놓는다. :)

## 2. Instruction
### 2.1 Install "dnf"
반드시 설치할 필요가 있는지는 확실치 않지만, 최근 rhel 시스템은 거의 dnf 를 사용하기 때문에,
서버의 패키지 관리를 용이하게 할 겸 기존 yum 시스템에서 dnf 로 업그레이드한다.`

{% highlight shell %}
$ yum install dnf dnf-plugins-core
{% endhighlight %}

### 2.2 Add Package Repo & Install Kernel Header and Devel 
Nvidia의 Cuda 레포를 추가하고 버전에 맞는 Kernel 헤더와 디벨 설치
CentOS 7이라면 uname -r을 하였을 때, 3.10.XXX가 뜬다.

{% highlight shell %}
$ dnf install epel-release
$ dnf config-manager --add-repo https://developer.download.nvidia.com/compute/cuda/repos/rhel7/x86_64/cuda-rhel7.repo
$ dnf install kernel-devel-$(uname -r) kernel-headers-$(uname -r)
{% endhighlight %}

### 2.3 Install Nvidia-Driver & Settings
Nvidia-Driver와 Settings를 설치!

{% highlight shell %}
$ dnf install nvidia-driver nvidia-settings
# 필요하다면, dnf install cuda-driver도 설치
# 기존에, 드라이버가 설치되어있으면, --eraseallowing(?) 옵션을 줘서 dnf가 자동으로 패키지 문제를 해결할 수 있도록 할 수 있다. 
{% endhighlight %}
