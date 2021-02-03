---
layout: post
title: "Create High Availability Kubernetes Cluster"
date: 2021-02-04 21:00 +0900
tags: [kubernetes]
---

### 1. Introduction
* CentOS 7.9에서 High Availability Kubernetes 구동
* External ETCD 사용

### 2. Environment
* Intel X64 Architecture
* CentOS 7.9
* Docker 19.03
* Kubernetes 1.20.2 (kubeadm, kubelet, kubectl)

### 3. Prerequisite
* Install net-tools, nc, tree
{% highlight shell %}
yum install net-tools nc tree
{% endhighlight %}

### 4. ETCD
#### 4.1 Objective
* External ETCD를 사용하지 않을 경우, 4. ETCD 과정은 패스
* ETCD 클러스터는 최소 3대 이상으로 구성
* 모두 root에 설치(일반계정은 sudoer 권한 필요)
* 192.168.0.241(etcd1), 192.168.0.242(etcd2), 192.168.0.243(etcd3)에 설치
* kubeadm을 이용 (1.20.2-0)
  * 추후 업데이트를 통해 etcdadm을 사용하여 설정할 수 있다고 함
  * 즉, Control plane과 etcd 설정 기능을 분리할 예정

#### 4.2 Prerequisite
{% highlight shell %}
$ kubeadm config images pull  # 모든 노드에서 수행
# 여기서부터는 HOST0에서만 수행
$ export HOST0="192.168.0.241"
$ export HOST1="192.168.0.242"
$ export HOST2="192.168.0.243"

# 작업 편의를 위해 설정
$ ssh-copy-id root@${HOST1}
$ ssh-copy-id root@${HOST2}
{% endhighlight %}

#### 4.3 Kubelet Service Configuration 파일 생성
{% highlight shell %}
$ cat << EOF | tee /usr/lib/systemd/system/kubelet.service.d/20-etcd-service-manager.conf
[Service]
ExecStart=
# Replace "systemd" with the cgroup driver of your container runtime. The default value in the kubelet is "cgroupfs".
ExecStart=/usr/bin/kubelet --address=127.0.0.1 --pod-manifest-path=/etc/kubernetes/manifests --cgroup-driver=systemd
Restart=always
EOF
# 공식문서에서는 /etc/systemd/system/kubelet.service.d/ 이지만,
# kubernetes-1.20.2에서는 이 폴더의 위치가 /usr/lib/systemd/system/kubelet.service.d/로 변경됨

$ systemctl daemon-reload
$ systemctl restart kubelet
$ systemctl status kubelet
● kubelet.service - kubelet: The Kubernetes Node Agent
   Loaded: loaded (/usr/lib/systemd/system/kubelet.service; enabled; vendor preset: disabled)
  Drop-In: /usr/lib/systemd/system/kubelet.service.d
           └─10-kubeadm.conf, 20-etcd-service-manager.conf
...

# Drop-In에 20-etcd-service-manager.conf가 로딩되어있는지 확인
{% endhighlight %}

#### 4.4 Kubeadm Configuration 파일 생성
{% highlight shell %}
$ mkdir -p /tmp/${HOST0}/ /tmp/${HOST1}/ /tmp/${HOST2}/

$ ETCDHOSTS=(${HOST0} ${HOST1} ${HOST2})
$ NAMES=("infra0" "infra1" "infra2")

$ for i in "${!ETCDHOSTS[@]}"; do
HOST=${ETCDHOSTS[$i]}
NAME=${NAMES[$i]}
cat << EOF > /tmp/${HOST}/kubeadmcfg.yaml
apiVersion: "kubeadm.k8s.io/v1beta2"
kind: ClusterConfiguration
etcd:
    local:
        serverCertSANs:
        - "${HOST}"
        peerCertSANs:
        - "${HOST}"
        extraArgs:
            initial-cluster: ${NAMES[0]}=https://${ETCDHOSTS[0]}:2380,${NAMES[1]}=https://${ETCDHOSTS[1]}:2380,${NAMES[2]}=https://${ETCDHOSTS[2]}:2380
            initial-cluster-state: new
            name: ${NAME}
            listen-peer-urls: https://${HOST}:2380
            listen-client-urls: https://${HOST}:2379
            advertise-client-urls: https://${HOST}:2379
            initial-advertise-peer-urls: https://${HOST}:2380
EOF
done
{% endhighlight %}

#### 4.5 Certificate 파일 생성
{% highlight shell %}
$ kubeadm init phase certs etcd-ca
$ ls /etc/kubernetes/pki/etcd # ca.crt, ca.key 생성 확인
-rw-r--r--. 1 root root 1058  2월  3 15:49 ca.crt
-rw-------. 1 root root 1679  2월  3 15:49 ca.key

$ kubeadm init phase certs etcd-server --config=/tmp/${HOST2}/kubeadmcfg.yaml
$ kubeadm init phase certs etcd-peer --config=/tmp/${HOST2}/kubeadmcfg.yaml
$ kubeadm init phase certs etcd-healthcheck-client --config=/tmp/${HOST2}/kubeadmcfg.yaml
$ kubeadm init phase certs apiserver-etcd-client --config=/tmp/${HOST2}/kubeadmcfg.yaml
$ cp -R /etc/kubernetes/pki /tmp/${HOST2}/
# 불필요한 certification 파일 삭제(재사용 불가)
$ find /etc/kubernetes/pki -not -name ca.crt -not -name ca.key -type f -delete

$ kubeadm init phase certs etcd-server --config=/tmp/${HOST1}/kubeadmcfg.yaml
$ kubeadm init phase certs etcd-peer --config=/tmp/${HOST1}/kubeadmcfg.yaml
$ kubeadm init phase certs etcd-healthcheck-client --config=/tmp/${HOST1}/kubeadmcfg.yaml
$ kubeadm init phase certs apiserver-etcd-client --config=/tmp/${HOST1}/kubeadmcfg.yaml
$ cp -R /etc/kubernetes/pki /tmp/${HOST1}/
$ find /etc/kubernetes/pki -not -name ca.crt -not -name ca.key -type f -delete

$ kubeadm init phase certs etcd-server --config=/tmp/${HOST0}/kubeadmcfg.yaml
$ kubeadm init phase certs etcd-peer --config=/tmp/${HOST0}/kubeadmcfg.yaml
$ kubeadm init phase certs etcd-healthcheck-client --config=/tmp/${HOST0}/kubeadmcfg.yaml
$ kubeadm init phase certs apiserver-etcd-client --config=/tmp/${HOST0}/kubeadmcfg.yaml

$ find /tmp/${HOST2} -name ca.key -type f -delete
$ find /tmp/${HOST1} -name ca.key -type f -delete
{% endhighlight %}

#### 4.6 Certificate 파일 이동
{% highlight shell %}
$ scp -r /tmp/${HOST1}/* root@${HOST1}:

$ ssh root@${HOST1}
# In HOST1
$ chown -R root:root pki
$ mv pki /etc/kubernetes
$ tree /etc/kubernetes/pki # 파일이 제대로 옮겨졌는지 확인
/etc/kubernetes/pki
├── apiserver-etcd-client.crt
├── apiserver-etcd-client.key
└── etcd
    ├── ca.crt
    ├── healthcheck-client.crt
    ├── healthcheck-client.key
    ├── peer.crt
    ├── peer.key
    ├── server.crt
    └── server.key
$ ls /root/kubeadmcfg.yaml
/root/kubeadmcfg.yaml
$ exit
# exit from HOST1

# HOST2에 대해서도 같은 작업을 수행
# In Host0
$ cp /tmp/${HOST0}/kubeadmcfg.yaml /root/
# 공식문서에서는 tmp를 바로 활용한다
# HOST1, HOST2와 비슷하게 맞춰주기 위해서 편의상 복사
{% endhighlight %}

#### 4.7 Static Pod 생성
{% highlight shell %}
# 모든 호스트(HOST0, HOST1, HOST2) 에서 아래와 같이 실행
$ kubeadm init phase etcd local --config=/root/kubeadmcfg.yaml
{% endhighlight %}

#### 4.8 Cluster health-check
{% highlight shell %}
$ kubeadm config images list
...
k8s.gcr.io/etcd:3.4.13-0
...

$ docker run --rm -it \
	--net host \
	-v /etc/kubernetes:/etc/kubernetes k8s.gcr.io/etcd:3.4.13-0 etcdctl \
	--cert /etc/kubernetes/pki/etcd/peer.crt \
	--key /etc/kubernetes/pki/etcd/peer.key \
	--cacert /etc/kubernetes/pki/etcd/ca.crt \
	--endpoints https://${HOST0}:2379 endpoint health --cluster

https://192.168.0.241:2379 is healthy: successfully committed proposal: took = 35.261986ms
https://192.168.0.243:2379 is healthy: successfully committed proposal: took = 36.311493ms
https://192.168.0.242:2379 is healthy: successfully committed proposal: took = 37.103945ms
{% endhighlight %}

### 5. Control Plane Node
#### 5.1 Prerequsite
* Kubernetes Master 노드의 seamless한 서비스를 위해서 두 컴포넌트가 필요
  * [keepalived](https://keepalived.readthedocs.io/en/latest/)
  * [HA Proxy](http://www.haproxy.org/)

##### 5.1.1 Install & Configure Keepalived
* 가상 아이피 설정
{% highlight shell %}

{% endhighlight %}

### Reference

{% highlight shell %}
{% endhighlight %}

* [Creating Highly Available clusters with kubeadm(Kubernetes Official Doc)](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/high-availability/)
* [Set up a High Availability etcd cluster with kubeadm(Kubernetes Official Doc)](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/setup-ha-etcd-with-kubeadm/)
* [HA-Consideration](https://github.com/kubernetes/kubeadm/blob/master/docs/ha-considerations.md#options-for-software-load-balancing)