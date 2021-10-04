---
layout: post
title: "Install KVM on Linux(feat. Ubuntu)"
date: 2021-09-28 22:47+0900
tags: [Linux]
---

### 1. Environment
* Ubuntu 20.04

### 2. Objective
* KVM 설치

### 3. Overview KVM
KVM은 Kernel-based Virtual Machine의 약자로 Linux kernel 기반의 가상화 기술이다. Linux Kernel 2.6.20 부터 linux mainline에 편입되어 가상화 기술을 리눅스 사용자는 비교적 편하게 사용할 수 있다.  

### 4. Install KVM 
#### 4.1 Check cpu support virtualization
기본적으로 CPU에서 가상화 기술(vmx - Intel VT or svm - AMD-V)을 지원해야 사용할 수 있다.
{% highlight shell %}
$ cat /proc/cpuinfo | grep -Eoc '(vmx|smv)'
24  # 제 PC 사양 - 0이 아닌 값이 나오면 사용 가능
{% endhighlight %}

#### 4.2 Check BIOS Setting enabled virtualization
Virtualization이 가능한 하드웨어라도 BIOS 옵션에 따라서 기능을 끄고 있을 수 있다.  
{% highlight shell %}
$ apt install cpu-checker  # need sudo privileges
$ kvm-ok
INFO: /dev/kvm exists
KVM acceleration can be used
{% endhighlight %}

아래와 같이 결과가 나오지 않으면, BIOS에 접속하여 Virtualization 기능을 Enable한다.
메인보드 제조사마다 하는 방법이 다르기 때문에 아래 두 명령어를 이용해 직접 보드를 확인하고 알아본다.
{% highlight shell %}
$ dmidecode | more  # sudo dmidecode -t 2
$ lspci  
{% endhighlight %}

#### 4.3 Installing KVM
필요 Package 설치
  * qemu-kvm: Open-source 가상화(hypervisor) 시스템, 프로세서를 에뮬레이트함
  * libvirt-daemon-system: libvirt는 Linux virtualization system의 C API Interface(Library). QEMU, KVM, XEN, VirtualBox 등을 지원함. libvirtd-daemon-system는 libvirt가 linux service로 동작하기 위해 필요한 패키지
  * libvirt-clients: libvirtd shell virsh를 포함한 여러 client binary가 포함됨
  * bridge-utils: Linux Ethernet bridge 설정 유틸리티
  * virt-manager: GUI 유저 인터페이스

{% highlight shell %}
$ sudo apt-get install qemu-kvm libvirt-daemon-system libvirt-clients bridge-utils virt-manager
{% endhighlight %}

#### 4.4 계정 추가
{% highlight shell %}
$ sudo adduser `echo $(whoami)` libvirtd
$ sudo adduser `echo $(whoami)` kvm
{% endhighlight %}

#### 4.5 설치 확인
{% highlight shell %}
$ virsh list --all
{% endhighlight %}


### Reference
* KVM Wikipedia: https://en.wikipedia.org/wiki/Kernel-based_Virtual_Machine
* KVM official: https://www.linux-kvm.org/page/Main_Page
* Ubuntu Community: https://linuxize.com/post/how-to-install-kvm-on-ubuntu-20-04/