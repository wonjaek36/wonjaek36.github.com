---
layout: post
title: "Kubernetes Worker Join"
date: 2021-02-18 21:00 +0900
tags: [kubernetes]
---

## 1. Introduction
* Kubernetes Control Plane 노드를 init하고 나면, 아래와 같은 문구가 나온다.
{% highlight shell %}
kubeadm join <Kubernetes API Server:PORT> --token <Token 값> --discovery-token-ca-cert-hash sha256:<Hash 값>
{% endhighlight %}
* 이를 이용하여, Kubernetes Worker, Backup Control Plane 노드에서 Join 한다.
* 하지만, 일정 기간이 지난 후에는 동작하지 않는데, 이는 token이 소멸되었기 때문
  * token을 새로 생성하고, 연결한다.
* 이 글에서는 Token을 생성하고, 생성된 Token을 이용하여 Worker 노드를 연결하는 방법을 알아본다.

## 2. Environment
* CentOS 7.9
* Docker 19.03
* Kubernetes 1.20.2 (kubeadm, kubelet, kubectl)

## 3. Join
### 3.1 Objective
* Worker 노드를 Control Plain 노드에 연결

### 3.2 Background
* Worker 노드가 Control Plain에 연결될 때, 서로 신뢰할 수 있는지 확인한다.
  * Bidirectional trust라고 표현
  * Node가 Control Plain을 신뢰하기 위해서 discovery 작업을 수행
  * Control Plain의 Node를 신뢰하기 위해서 TLS bootstrap을 수행

* Discovery는 두 개의 방법으로 수행
  * API Server를 통해 shared token을 사용(이 글에서 사용)
  * File을 이용하는 방법

* Shared Token을 사용하려면, --discovery-token-ca-cert-hash flag을 이용하여 Control Plain의 Certificate Authority(CA)의 Public Key를 전달
  * 이 flag을 사용하지 않으면, --discovery-token-unsafe-skip-ca-verification flag 전달
  * 단, Discovery 과정이 생략되므로, Node는 신뢰할 수있는 Control Plain에 연결되었다고 보장할 수 없음

### 3.3 Token Create & Verification
* Control Plain 노드에서 확인

{% highlight shell %}
$ kubeadm token list
kubeadm token list
TOKEN                     TTL         EXPIRES                     USAGES                   DESCRIPTION                                                EXTRA GROUPS
a02xm5.8acxgx73uov5hjcl   8h          2021-02-19T21:32:23+09:00   authentication,signing   <none>                                                     system:bootstrappers:kubeadm:default-node-token
# Token이 하나 있는 것을 확인, 연습을 위해 하나 더 생성

$ kubeadm token create
828d4t.l6npwyctm7fe4g7c

$ kubeadm token list
TOKEN                     TTL         EXPIRES                     USAGES                   DESCRIPTION                                                EXTRA GROUPS
828d4t.l6npwyctm7fe4g7c   23h         2021-02-20T12:44:26+09:00   authentication,signing   <none>                                                     system:bootstrappers:kubeadm:default-node-token
a02xm5.8acxgx73uov5hjcl   8h          2021-02-19T21:32:23+09:00   authentication,signing   <none>                                                     system:bootstrappers:kubeadm:default-node-token
# 생성된 토큰이 첫 번째 줄에 있는 것을 확인
# Expires를 보면, 각 토큰이 언제 사라지는지 확인할 수 있음 
{% endhighlight %}

### 3.4 Ca-cert 파일 Hash 계산
* Control Plain 노드에서 확인
{% highlight shell %}
$ openssl x509 -pubkey -in /etc/kubernetes/pki/ca.crt | openssl rsa -pubin -outform der 2>/dev/null | openssl dgst -sha256 -hex | sed 's/^.* //'
196ae33f39ba03ddcb4763326c2fb9f3775dc54089b80abe58505e6c834693a6
{% endhighlight %}

### 3.5 Worker 노드 Join
* Worker 노드에서 연결 시도
{% highlight shell %}
$ kubeadm join <Kubernetes API Server:PORT> --token 828d4t.l6npwyctm7fe4g7c --discovery-token-ca-cert-hash sha256:196ae33f39ba03ddcb4763326c2fb9f3775dc54089b80abe58505e6c834693a6
[preflight] Running pre-flight checks
[preflight] Reading configuration from the cluster...
[preflight] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -o yaml'
...
This node has joined the cluster:
* Certificate signing request was sent to apiserver and a response was received.
* The Kubelet was informed of the new secure connection details.

Run 'kubectl get nodes' on the control-plane to see this node join the cluster.
# 연결 성공
{% endhighlight %}

## Reference
* https://kubernetes.io/docs/reference/setup-tools/kubeadm/kubeadm-join/