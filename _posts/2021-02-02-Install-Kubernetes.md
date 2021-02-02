---
layout: post
title: "Install Kubernetes on CentOS"
date: 2021-02-02 20:00 +0900
tags: [kubernetes]
---

### 1. Introduction
CentOS 7에서 Kubernetes 1.20.2-0 버전 설치 문서

### 2. Environment
* CentOS 7
* Intel x64

### 3. Prerequsite
#### 3.1 Check MAC Address, product_uuid
* Kubernetes는 두 값을 이용해서 Node를 구별
* Barebone 장비를 사용한다면, 문제 없음
* Virtual Machine을 사용하면, 노드 별로 두 값이 구분되는지 확인 필요

#### 3.2 iptable이 bridged traffic(경유 트래픽) 볼 수 있도록 설정
{% highlight shell %}
$ lsmod | grep br_netfilter # 아무 결과가 없다면, 설정 안 됨

$ modprobe br_netfilter
$ lsmod | grep br_netfilter
br_netfilter     22256 0
bridge          151336 2 br_netfilter,ebtable_broute

$ sysctl --system | grep tables  # 설정이 안 되어있음
net.bridge.bridge-nf-call-ip6tables = 0
net.bridge.bridge-nf-call.iptables = 0

$ cat <<EOF | tee /etc/modules-load.d/k8s.conf
br_netfilter
EOF

$ cat <<EOF | tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF

$ sysctl --system | grep tables
...
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
{% endhighlight %}

#### 3.3 Port check
* Control-Plain Node

	| &nbsp; Protocol &nbsp; | &nbsp; Direction &nbsp; | &nbsp; Port Range &nbsp; |
	| :---: | :---: | ------: |
	|TCP|Inbound|6443|
	|TCP|Inbound|2379-2380|
	|TCP|Inbound|10250|
	|TCP|Inbound|10251|
	|TCP|Inbound|10252|

&nbsp;  
* Worker Node

    | &nbsp; Protocol &nbsp; | &nbsp; Direction &nbsp; | &nbsp; Port Range &nbsp; |
    | :---: | :---: | ---: |
    |TCP|Inbound|10250|
    |TCP|Inbound|30000-32767|

&nbsp;  
* 연구실에서는 앞단에 방화벽 장비를 놓았다고 가정함. 따라서 방화벽 서비스를 disable
{% highlight shell %}
$ systemctl stop firewalld
$ systemctl disable firewalld
$ systemctl status firewalld
● firewalld.service - firewalld - dynamic firewall daemon
Loaded: loaded (/usr/lib/systemd/system/firewalld.service; disabled; vendor preset: enabled)
Active: inactive (dead)
Docs: man:firewalld(1)
{% endhighlight %}

#### 3.4 Disable Swap
* Kubernetes(Kubelet)을 사용하기 위해서는 리눅스의 swap 영역을 disable 해야함 
{% highlight shell %}
$ swapoff -a 
$ echo 0 > /proc/sys/vm/swappiness
$ sed -e '/swap/ s/^#*/#/' -i /etc/fstab 
{% endhighlight %}

#### 3.5 Install Runtime
* Kubernetes는 CRI(Container Runtime Interface)를 사용
* Runtime을 지정하지 않으면, 아래 3가지를 찾아서 사용
  * Docker	/var/run/docker.sock
  * containerd /run/containerd/containerd.sock
  * CRI-O	/var/run/crio/crio.sock

* Docker랑 containerd가 동시에 발견될 경우, Docker 사용
  * Docker 18.09부터 containerd 사용
* 그 이외에 두 개 이상 발견되면 에러 발생
* 이 문서에서는 Docker를 사용
  * [Docker Install](https://docs.docker.com/engine/install/centos/)
{% highlight shell %}
$ yum remove docker \
			docker-client \
			docker-client-latest \
			docker-common \
			docker-latest \
			docker-latest-logrotate \
			docker-logrotate \
			docker-engine

$ yum install -y yum-utils
$ yum-config-manager \
	--add-repo \
	https://download.docker.com/linux/centos/docker-ce.repo
$ yum list docker-ce --showduplicates | sort -r # Docker version 확인
$ yum install docker-ce-19.03.9 docker-ce-cli-19.03.9 containerd.io # 2021-02-02 기준으로 kubernetes는 19.03.9까지 지원
$ systemctl enable docker
$ systemctl start docker
{% endhighlight %}

### 4. Install Kubernetes
* kubeadm, kubectl, kubelet을 설치
  * kubeadm: cluster를 bootstrap하는 명령어
  * kubelet: 모든 클러스터에서 동작하는 컴포넌트, pods, container를 시작할때 필요
  * kubectl: 사용자가 클러스터에 필요한 요청할때 사용하는 컴포넌트

{% highlight shell %}
$ cat <<EOF | tee /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-\$basearch
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
exclude=kubelet kubeadm kubectl
EOF

$ setenforce 0
$ sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config
$ yum list kubelet --showduplicates --disableexcludes=kubernetes | sort -r # kubelet Version 확인
$ yum install -y kubelet-1.20.2-0 kubeadm-1.20.2-0 kubectl-1.20.2-0 --disableexcludes=kubernetes
$ systemctl enable --now kubelet
$ systemctl daemon-reload
$ systemctl restart kubelet
{% endhighlight %}

### 5. Configure cgroup driver as systemd
* systemd는 리눅스에서 사용하는 resource constrainer
  * kubernetes는 cgroupfs를 사용
  * 두 개를 따로따로 써도 되지만... 하나로 쓰는 게 좋다.

* 먼저 Docker가 systemd를 사용하도록 설정
  * 후에, kubelet이 사용할 수 있도록 설정

{% highlight shell %}
$ cat <<EOF | sudo tee /etc/docker/daemon.json
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2",
  "storage-opts": [
    "overlay2.override_kernel_check=true"
  ]
}
EOF

$ vim /usr/lib/systemd/system/kubelet.service.d/10-kubeadm.conf
# Service 밑에 Environment="KubeLET_CGROUP_ARGS=--..." 추가

[Service]
Environment="KUBELET_CGROUP_ARGS=--cgroup-driver=systemd --runtime-cgroups=/systemd/system.slice --kubelet-cgroups=/systemd/system.slice"
...

# 또는 sed 명령어를 이용하여 추가
$ sed -i '/^\[Service\]/a\Environment=\"KUBELET_CGROUP_ARGS=--cgroup-driver=systemd --runtime-cgroups=/systemd/system.slice --kubelet-cgroups=/systemd/system.slice\"' /usr/lib/systemd/system/kubelet.service.d/10-kubeadm.conf
{% endhighlight %}