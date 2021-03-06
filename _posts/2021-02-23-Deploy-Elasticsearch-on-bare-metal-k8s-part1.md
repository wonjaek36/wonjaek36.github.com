---
layout: post
title: "Deploy Elasticsearch on bare-metal k8s(Part 1)"
date: 2021-02-23 01:00 +0900
tags: [kubernetes]
---

## 1. Introduction
* Elasticsearch 6.8을 Kubernetes에 설치
* Kubernetes는 Bare-Metal에 직접 설치한 버전
  * LoadBalancer가 없어 MetalLB 사용
* 이번 글에서는 k8s에 Elasticsearch Cluster를 구성하기 위한 Load-Balancer 설치 및 Service를 구동함

## 2. Environment
* CentOS 7.9
* Kubernetes 1.18
* Elasticsearch 6.8
* MetalLB 0.9.5
* Calico 3.8.9

## 3. Install ipvs & Reload kube-system
* Metal Load-Balancer를 사용하기 위해서는 Kubernetes가 ipvs를 사용해야 한다.
* 먼저 ipvs 설치
{% highlight shell %}
$ yum install -y ipset ipvsadm

$ cat > /etc/sysconfig/modules/ipvs.modules <<EOF
> #!/bin/bash
modprobe -- ip_vs
modprobe -- ip_vs_rr
modprobe -- ip_vs_wrr
modprobe -- ip_vs_sh
modprobe -- nf_conntrack_ipv4
EOF

$ chmod 755 /etc/sysconfig/modules/ipvs.modules
$ bash /etc/sysconfig/modules/ipvs.modules && lsmod | grep -e ip_vs -e nf_conntrack_ipv4
{% endhighlight %}  

* Reload Kube-system
  * Main Control Plane Node에서 작업
{% highlight bash %}
$ kubectl edit configmap kube-proxy -n kube-system
# In text editor
apiVersion: v1
data:
  config.conf: |-
    apiVersion: kubeproxy.config.k8s.io/v1alpha1
    ...
    ipvs:
      ...
      strictARP: true # false -> true로 변경
      ...
    mode: "ipvs" # "" -> ipvs로 변경
  ...

$ kubectl get po -n kube-system | grep kube-proxy
kube-proxy-2qr7v                          1/1     Running   0          4h13m
kube-proxy-2z4zg                          1/1     Running   0          3h55m
kube-proxy-bk8v8                          1/1     Running   0          3h54m
...

$ kubectl delete po kube-proxy-2qr7v -n kube-system
$ kubectl delete po kube-proxy-2z4zg -n kube-system
# 모든 kube-proxy-xxxxx pod을 제거
# 처음에 확인했을 때 구동된 것만 제거하면 된다.
# kube-proxy를 제거하면 다시 실행되는데, 다시 실행된 것을 또 제거할 필요는 없음

# ipvs 사용 확인
$ kubectl logs [new kube-proxy pod] -n kube-system | grep "Using ipvs Proxier"
I0222 12:06:24.968695       1 server_others.go:258] Using ipvs Proxier. # 결과가 나온다면 성공

# Cluster IP로 사용되는 것을 확인할 수 있음
# CNI 설정에 따라 ip는 다를 수 있음
$ ipvsadm -Ln
IP Virtual Server version 1.2.1 (size=4096)
Prot LocalAddress:Port Scheduler Flags
  -> RemoteAddress:Port           Forward Weight ActiveConn InActConn
TCP  127.0.0.1:31727 rr
  -> 172.16.3.70:9200             Masq    1      0          0         
  -> 172.16.33.134:9200           Masq    1      0          0         
  -> 172.16.135.6:9200            Masq    1      0          0         
TCP  172.16.241.128:31727 rr
  -> 172.16.3.70:9200             Masq    1      0          0         
  -> 172.16.33.134:9200           Masq    1      0          0         
  -> 172.16.135.6:9200            Masq    1      0          0   
...
{% endhighlight %}

## 4. Install MetalLB
* Elasticsearch는 일반적으로 여러 개의 노드로 클러스터를 구성함
  * 따라서 Service의 Publishing Type으로 Load Balancer를 쓰는게 적당함
* 하지만, 처음에는 Bare-metal k8s에는 Load Balancer 없다.
  * 공식문서에 따르면, "클라우드 공급자의 로드 밸런서를 사용하여 서비스를 외부에 노출시킨다." 인데... 우린 k8s를 직접 설치했기 때문이다.
  * 인그레스(Ingress)를 사용할 수도 있다고 한다. (이 부분은 실패함...)
* Bare-Metal에서 Load Balancer Type을 사용할 수 있게 해주는 방법은 Metal LB을 설치하는 것
  * Layer 2와 BGP, 두 가지 방식이 있음
  * 연구실의 Network는 소규모이고, 아이피 관리가 잘 되지 않아, Layer 2를 사용

* [Install Metal LB](https://metallb.universe.tf/installation/)
* [Configuration Metal LB](https://metallb.universe.tf/configuration/#layer-2-configuration)
{% highlight shell %}
$ kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.9.5/manifests/namespace.yaml
$ kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.9.5/manifests/metallb.yaml
# On first install only
$ kubectl create secret generic -n metallb-system memberlist --from-literal=secretkey="$(openssl rand -base64 128)"

$ cat config.yaml <<EOF
apiVersion: v1
kind: ConfigMap
metadata:
  namespace: metallb-system
  name: config
data:
  config: |
    address-pools:
    - name: default
      protocol: layer2
      addresses:
      - 192.168.0.70-192.168.0.99
EOF
$ kubectl apply -f config.yaml
$ kubectl get po -n metallb-system
NAME                          READY   STATUS    RESTARTS   AGE
controller-65db86ddc6-dtcrq   1/1     Running   0          4h22m
speaker-72cjj                 1/1     Running   0          4h22m
speaker-cr6sb                 1/1     Running   0          4h22m
...

# speaker는 노드의 수 만큼, controller는 하나
# 만약 Controller가 pending 상태라면, control plane 노드만 연결했을 수도 있다.
# Worker 노드가 추가되면, 워커 노드에 controller가 deploy 된다.
{% endhighlight %}

## 5. Create Service
* Elasticsearch는 두 개의 포트를 사용(9200, 9300)
  * 9200은 Rest API, 간단하게 말하면 유저와의 통신
  * 9300은 Node Communicate을 위해 사용
* 따라서, 서비스도 두 가지가 필요

* 먼저 Cluster 간 통신을 위한 포트
  * Pod들 간의 통신이기 때문에, 굳이 외부로 아이피를 공개할 필요가 업승ㅁ
  * ClusterIP type으로 생성
  * metadata.name은 기억 필수, 추후 elasticsearch 환경변수에서 사용함
    * 6.8 버전에서는 discovery.zen.ping.unicast.hosts에 사용

{% highlight yaml %}
apiVersion: v1
kind: Service
metadata:
  name: elastic-cluster-svc
spec:
  clusterIP: None
  ports:
  - port: 9300
    protocol: TCP
    targetPort: 9300
  selector:
    app: elasticsearch
  sessionAffinity: None
  type: ClusterIP
{% endhighlight %}

* 두번 째는, Rest API Port
  * 이는 유저와 통신을 담당해야하는 부분이라서 외부 IP가 필요
    * MetalLB가 할당해줌
  * Elasticsearch Cluster를 3개로 구성할 것이기 때문에, Load Balancer Type으로 사용
{% highlight yaml %}
apiVersion: v1
kind: Service
metadata:
  name: elastic-loadbalancer-svc
spec:
  type: LoadBalancer
  selector:
    app: elasticsearch
  ports:
    - port: 9200
      targetPort: 9200
status:
  loadBalancer:
    ingress:
      - ip: 192.168.0.70 # 유의미한지 아직은 잘 모르겠음...
{% endhighlight %}

* 실행한 서비스 확인
{% highlight shell %}
$ kubectl get svc
NAME                       TYPE           CLUSTER-IP    EXTERNAL-IP    PORT(S)          AGE
elastic-cluster-svc        ClusterIP      None          <none>         9300/TCP         32m
elastic-loadbalancer-svc   LoadBalancer   10.99.13.33   192.168.0.70   9200:32530/TCP   32m
kubernetes                 ClusterIP      10.96.0.1     <none>         443/TCP          15h

$ kubectl describe svc elastic-cluster-svc
Name:              elastic-cluster-svc
Namespace:         default
Labels:            <none>
Annotations:       Selector:  app=elasticsearch
Type:              ClusterIP
IP:                None
Port:              <unset>  9300/TCP
TargetPort:        9300/TCP
Endpoints:         172.16.135.1:9300,172.16.3.65:9300,172.16.33.129:9300
Session Affinity:  None
Events:            <none>

$ kubectl describe svc elastic-loadbalancer-svc
Name:              elastic-cluster-svc
Namespace:         default
Labels:            <none>
Annotations:       Selector:  app=elasticsearch
Type:              ClusterIP
IP:                None
Port:              <unset>  9300/TCP
TargetPort:        9300/TCP
Endpoints:         172.16.135.1:9300,172.16.3.65:9300,172.16.33.129:9300
Session Affinity:  None
Events:            <none>

$ kubectl describe svc elastic-loadbalancer-svc
Name:                     elastic-loadbalancer-svc
Namespace:                default
Labels:                   <none>
Annotations:              Selector:  app=elasticsearch
Type:                     LoadBalancer
IP:                       10.99.13.33
LoadBalancer Ingress:     192.168.0.70
Port:                     <unset>  9200/TCP
TargetPort:               9200/TCP
NodePort:                 <unset>  32530/TCP
Endpoints:                172.16.135.1:9200,172.16.3.65:9200,172.16.33.129:9200
Session Affinity:         None
External Traffic Policy:  Cluster
Events:
  Type    Reason        Age                From                Message
  ----    ------        ----               ----                -------
  Normal  IPAllocated   33m                metallb-controller  Assigned IP "192.168.0.70"
{% endhighlight %}


## 6. Reference
* [Kubernetes Service](https://kubernetes.io/ko/docs/concepts/services-networking/service/#loadbalancer)
* [MetalLB Installation Guide](https://metallb.universe.tf/installation/)
* [IPVS-Based In-Cluster Load Balancing Deep Dive](https://kubernetes.io/blog/2018/07/09/ipvs-based-in-cluster-load-balancing-deep-dive/)
* [Stack overflow-enable ipvs mode in kube-proxy on a ready kubernetes local cluster](https://stackoverflow.com/questions/56493651/enable-ipvs-mode-in-kube-proxy-on-a-ready-kubernetes-local-cluster)
* [Devopstables](https://devopstales.github.io/)