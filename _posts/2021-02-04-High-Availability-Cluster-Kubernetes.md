---
layout: post
title: "Create High Availability Kubernetes Cluster"
date: 2021-02-04 15:00 +0900
tags: [kubernetes]
---

## 1. Introduction
* CentOS 7.9에서 High Availability Kubernetes 구동
* External ETCD 사용

## 2. Environment
* Intel X64 Architecture
* CentOS 7.9
* Docker 19.03
* Kubernetes 1.20.2 (kubeadm, kubelet, kubectl)


## 3. Prerequisite
* Install net-tools, nc, tree
{% highlight shell %}
yum install net-tools nc tree
{% endhighlight %}

## 4. ETCD
### 4.1 Objective
* External ETCD를 사용하지 않을 경우, 4. ETCD 과정은 패스
* ETCD 클러스터는 최소 3대 이상으로 구성
* 모두 root에 설치(일반계정은 sudoer 권한 필요)
* 192.168.0.241(etcd1), 192.168.0.242(etcd2), 192.168.0.243(etcd3)에 설치
* kubeadm을 이용 (1.20.2-0)
  * 추후 업데이트를 통해 etcdadm을 사용하여 설정할 수 있다고 함
  * 즉, Control plane과 etcd 설정 기능을 분리할 예정

### 4.2 Prerequisite
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

### 4.3 Kubelet Service Configuration 파일 생성
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

### 4.4 Kubeadm Configuration 파일 생성
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

### 4.5 Certificate 파일 생성
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

### 4.6 Certificate 파일 이동
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

### 4.7 Static Pod 생성
{% highlight shell %}
# 모든 호스트(HOST0, HOST1, HOST2) 에서 아래와 같이 실행
$ kubeadm init phase etcd local --config=/root/kubeadmcfg.yaml
{% endhighlight %}

### 4.8 Cluster health-check
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

## 5. Control Plane Node

### 5.1 Objective
* Control Plane Node는 최소 3대 이상으로 구성
  * External ETCD와 관계없이 3대 이상
* 모두 root에 설치(일반계정은 sudoer 권한 필요)
* 192.168.0.251(control1), 192.168.0.252(control2), 192.168.0.253(control3)에 설치
* kubeadm을 이용 (1.20.2-0)
  * CNI는 Calico 3.9 사용

### 5.2 Prerequsite
* Kubernetes Master 노드의 seamless한 서비스를 위해서 두 컴포넌트가 필요
  * [keepalived](https://keepalived.readthedocs.io/en/latest/)
  * [HA Proxy](http://www.haproxy.org/)

#### 5.2.1 Pre-Configure & Install Keepalived
* 가상 아이피 설정
  * 251, 252, 253을 대표하는 가상아이피는 250으로 설정

{% highlight shell %}
$ ip addr show
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: p2p1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether c8:1f:66:40:6e:38 brd ff:ff:ff:ff:ff:ff
    inet 192.168.0.251/24 brd 192.168.0.255 scope global secondary noprefixroute p2p1
       valid_lft forever preferred_lft forever

$ cd /etc/sysconfig/network-scripts
$ cp ifcfg-p2p1 ifcfg-p2p1:1

$ vi /etc/sysconfig/network-scripts/ifcfg-p2p1:1
...
NAME="p2p1:1"
DEVICE="p2p1:1"
IPADDR="192.168.0.250"
...

$ vi /etc/sysconfig/network-scripts/ifcfg-p2p1 # ifcfg-p2p1:1과 비교
...
NAME="p2p1"
DEVICE="p2p1"
IPADDR="192.168.0.251"
...

$ systemctl restart network
$ ip addr show
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: p2p1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether c8:1f:66:40:6e:38 brd ff:ff:ff:ff:ff:ff
    inet 192.168.0.251/24 brd 192.168.0.255 scope global secondary noprefixroute p2p1
       valid_lft forever preferred_lft forever
    inet 192.168.0.250/24 brd 192.168.0.255 scope global secondary noprefixroute p2p1:1
       valid_lft forever preferred_lft forever

$ yum install keepalived # Install keep alived
{% endhighlight %}

#### 5.2.2 Configure Keepalived
* Master와 Backup을 구분
  * 192.168.0.251을 Master
  * 192.168.0.252, 253을 Backup으로 설정

* vrrp_instance VI_1 중요값
  * state: MASTER node는 "MASTER"로, BACKUP node는 "BACKUP"으로 설정
  * interface: Virutal IP를 가지고 있는 interface, 지금 환경에서는 p2p1
  * virtual_router_id: keepalived host간에 모두 같은 값을 가지고 있다. 권장 값은 51
  * priority: BACKUP보다 MASTER가 큰 값을 가지면 됨. MASTER는 101, BACKUP은 100으로 설정
  * authentication.auth_pass: keepalived cluster 간에 같은 값을 가지면 된다. 123456789로 설정
  * virtual_ipaddress: 클러스터가 공유하게 될 virtual ip address - 192.168.0.250

&nbsp; 
* MASTER node, 192.168.0.251 설정 파일
{% highlight shell %}
$ export APISERVER_DEST_PORT="6443"
$ export APISERVER_VIP="192.168.0.250"

$ cat <<EOF | tee /etc/keepalived/keepalived.conf
! Configuration File for keepalived

global_defs {
    router_id LVS_DEVEL
}

vrrp_script check_apiserver {
    script "/etc/keepalived/check_apiserver.sh"
    interval 3
    weight -2
    fall 10
    rise 2
}

vrrp_instance VI_1 {
    state MASTER
    interface p2p1
    virtual_router_id 51
    priority 101
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 123456789
    }
    virtual_ipaddress {
        192.168.0.250
    }

    track_script {
        check_apiserver
    }
}
EOF
{% endhighlight %}

* BACKUP node, 192.168.0.252, 253 설정파일
{% highlight shell %}
export APISERVER_DEST_PORT="6443"
export APISERVER_VIP="192.168.0.250"
cat <<EOF | tee /etc/keepalived/keepalived.conf
! Configuration File for keepalived

global_defs {
    router_id LVS_DEVEL
}

vrrp_script check_apiserver {
    script "/etc/keepalived/check_apiserver.sh"
    interval 3
    weight -2
    fall 10
    rise 2
}

vrrp_instance VI_1 {
    state BACKUP
    interface p2p1
    virtual_router_id 51
    priority 100
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 123456789
    }
    virtual_ipaddress {
        192.168.0.250
    }

    track_script {
        check_apiserver
    }
}
EOF
{% endhighlight %}

* 모든 노드에 check_apiserver.sh 추가
{% highlight shell %}
$ cat <<EOF | tee /etc/keepalived/check_apiserver.sh
#!/bin/sh

errorExit() {
    echo "*** $*" 1>&2
    exit 1
}

curl --silent --max-time 2 --insecure https://localhost:${APISERVER_DEST_PORT}/ -o /dev/null || errorExit "Error GET https://localhost:${APISERVER_DEST_PORT}/"
if ip addr | grep -q ${APISERVER_VIP}; then
    curl --silent --max-time 2 --insecure https://${APISERVER_VIP}:${APISERVER_DEST_PORT}/ -o /dev/null || errorExit "Error GET https://${APISERVER_VIP}:${APISERVER_DEST_PORT}/"
fi
EOF

$ chmod 744 /etc/keepalived/check_apiserver.sh
$ systemctl start keepalived
$ systemctl enabled keepalived
{% endhighlight %}

#### 5.2.3 Install & Configure HAProxy
{% highlight shell %}
# HAProxy > 2.1.5 설치
# CentOS 7.9 yum repo는 1.5 지원
# Official Site에서 코드 직접 다운로드

$ yum install gcc pcre-devel tar make wget -y
$ wget http://www.haproxy.org/download/2.3/src/haproxy-2.3.4.tar.gz
$ tar xvf ~/haproxy-2.3.4.tar.gz -C ~/
$ cd ~/haproxy-2.3.4

$ make TARGET=linux-glibc
$ make install

# Configure HAProxy
$ mkdir -p /etc/haproxy
$ mkdir -p /var/lib/haproxy
$ touch /var/lib/haproxy/stats

$ ln -s /usr/local/sbin/haproxy /usr/sbin/haproxy
$ cp ~/haproxy-2.3.4/examples/haproxy.init /etc/init.d/haproxy
$ chmod 755 /etc/init.d/haproxy
$ systemctl daemon-reload

$ chkconfig haproxy on
$ useradd -r haproxy
$ haproxy -v
HA-Proxy version 2.3.4-10189c9 2021/01/13 - https://haproxy.org/
Status: stable branch - will stop receiving fixes around Q1 2022.
Known bugs: http://www.haproxy.org/bugs/bugs-2.3.4.html
Running on: Linux 3.10.0-1160.el7.x86_64 #1 SMP Mon Oct 19 16:18:59 UTC 2020 x86_64
{% endhighlight %}

#### 5.2.4 Configuration for Kubernetes
* 모든 호스트에 적용
* 외부에서 Kubernetes API 접근시에 8443으로 접근
  * Proxy 바깥에서는 6443

{% highlight shell %}
export APISERVER_DEST_PORT="8443"
export HOST1_ID="control1"
export HOST2_ID="control2"
export HOST3_ID="control3"
export HOST1_ADDRESS="192.168.0.251"
export HOST2_ADDRESS="192.168.0.252"
export HOST3_ADDRESS="192.168.0.253"
export APISERVER_SRC_PORT="6443"

cat <<EOF | tee /etc/haproxy/haproxy.cfg
#---------------------------------------------------------------------
# Global settings
#---------------------------------------------------------------------
global
    log /dev/log local0
    log /dev/log local1 notice
    daemon

#---------------------------------------------------------------------
# common defaults that all the 'listen' and 'backend' sections will
# use if not designated in their block
#---------------------------------------------------------------------
defaults
    mode                    http
    log                     global
    option                  httplog
    option                  dontlognull
    option http-server-close
    option forwardfor       except 127.0.0.0/8
    option                  redispatch
    retries                 1
    timeout http-request    10s
    timeout queue           20s
    timeout connect         5s
    timeout client          20s
    timeout server          20s
    timeout http-keep-alive 10s
    timeout check           10s

#---------------------------------------------------------------------
# apiserver frontend which proxys to the masters
#---------------------------------------------------------------------
frontend apiserver
    bind *:${APISERVER_DEST_PORT}
    mode tcp
    option tcplog
    default_backend apiserver

#---------------------------------------------------------------------
# round robin balancing for apiserver
#---------------------------------------------------------------------
backend apiserver
    option httpchk GET /healthz
    http-check expect status 200
    mode tcp
    option ssl-hello-chk
    balance     roundrobin
        server ${HOST1_ID} ${HOST1_ADDRESS}:${APISERVER_SRC_PORT} check
        server ${HOST2_ID} ${HOST2_ADDRESS}:${APISERVER_SRC_PORT} check
        server ${HOST3_ID} ${HOST3_ADDRESS}:${APISERVER_SRC_PORT} check

EOF
{% endhighlight %}

#### 5.2.5 Init Control Plane Node
* etcd1(192.168.0.241)에서 First Control Node로 pki 전달

{% highlight shell %}
$ export CONTROL_PLANE="root@192.168.0.251"
$ scp /etc/kubernetes/pki/etcd/ca.crt "${CONTROL_PLANE}":/etc/kubernetes/pki/etcd/
$ scp /etc/kubernetes/pki/apiserver-etcd-client.crt "${CONTROL_PLANE}":/etc/kubernetes/pki/
$ scp /etc/kubernetes/pki/apiserver-etcd-client.key "${CONTROL_PLANE}":/etc/kubernetes/pki/
{% endhighlight %}

* kubeadm-config.yaml 작성(First Control Node)
{% highlight shell %}
$ cat << EOF | tee kubeadm-config.yaml
apiVersion: kubeadm.k8s.io/v1beta2
kind: ClusterConfiguration
kubernetesVersion: stable
controlPlaneEndpoint: "192.168.0.250:8443"
etcd:
    external:
        endpoints:
        - https://192.168.0.241:2379
        - https://192.168.0.242:2379
        - https://192.168.0.243:2379
        caFile: /etc/kubernetes/pki/etcd/ca.crt
        certFile: /etc/kubernetes/pki/apiserver-etcd-client.crt
        keyFile: /etc/kubernetes/pki/apiserver-etcd-client.key
{% endhighlight %}

* First Control Node Init!
{% highlight shell %}
$ kubeadm init --config kubeadm-config.yaml --upload-certs
...
kubeadm join 192.168.0.250:8443 --token 6gbulo.gcvxyg49i8ii46ne \
    --discovery-token-ca-cert-hash sha256:196ae33f39ba03ddcb4763326c2fb9f3775dc54089b80abe58505e6c834693a6 \
    --control-plane --certificate-key 4481e8f68eae065866b9e514708e87bce72d73c6129f5e6478caa4d6add457e0
   

Please note that the certificate-key gives access to cluster sensitive data, keep it secret!
As a safeguard, uploaded-certs will be deleted in two hours; If necessary, you can use
"kubeadm init phase upload-certs --upload-certs" to reload certs afterward.

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 192.168.0.250:8443 --token 6gbulo.gcvxyg49i8ii46ne \
    --discovery-token-ca-cert-hash sha256:196ae33f39ba03ddcb4763326c2fb9f3775dc54089b80abe58505e6c834693a6
...
{% endhighlight %}

* 다른 Control Node(252, 253)에서 첫 번째 kubeadm join .. 명령어를 이용하여 연결
* Calico 연결 
{% highlight shell %}
$ wget https://docs.projectcalico.org/v3.8/manifests/calico.yaml
$ vi calico.yaml  # 이 부분은 개인의 설정에 따라 변경
# - name: CALICO_IPV4POOL_CIDR
#   value: "192.168.0.0/16"
# 192.168.0.0/24를 172.16.0.0/24로 변경

$ kubectl apply -f calico.yaml
$ kubectl get nodes # 연결 확인
NAME       STATUS   ROLES                  AGE   VERSION
control1   Ready    control-plane,master   19h   v1.20.2
control2   Ready    control-plane,master   19h   v1.20.2
control3   Ready    control-plane,master   19h   v1.20.2
{% endhighlight %}


## 6. Reference

* [Creating Highly Available clusters with kubeadm(Kubernetes Official Doc)](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/high-availability/)
* [Set up a High Availability etcd cluster with kubeadm(Kubernetes Official Doc)](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/setup-ha-etcd-with-kubeadm/)
* [HA-Consideration](https://github.com/kubernetes/kubeadm/blob/master/docs/ha-considerations.md#options-for-software-load-balancing)