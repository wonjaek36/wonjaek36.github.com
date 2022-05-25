---
layout: post
title: "Private registry 서버 설치 in docker"
date: 2022-04-27 23:05 +0900
category: [docker]
tags: [docker]
---

## 1. Overview
Docker image를 저장하고 분산하기 위해 저장소를 따로 구축하여 운용해야할 필요가 있다. 이럴 때 [Docker registry](https://hub.docker.com/_/registry)을 이용하면 된다.

* 참고) 이번 글은 [공식 문서](https://docs.docker.com/registry/)와 거의 차이가 없습니다.

## 2. Environment
* Ubuntu 20.04
* Docker version: 20.10.14

## 3. Docker registry Pull & Run
{% highlight shell %}
$ docker pull registry:2.8.1  # 작성날 기준 최신버전
$ docker images
$ docker run -d -p 5000:5000 --name registry registry:2.8.1
# docker container의 이름과 host 포트는 변경 가능
# -p <host port>:<container port>
{% endhighlight %}

## 4. Docker image push & pull
{% highlight shell %}
# Test를 위해서 hub로부터 ubuntu:bionic 다운로드
$ docker image pull ubuntu:bionic
# <server host>:<host port>/image 이름/tag로 작성
$ docker tag ubuntu:bionic localhost:5000/ubuntu:bionic
$ docker image push localhost:5000/ubuntu:bionic
# 다운로드 테스트
$ docker rmi localhost:5000/ubuntu:bionic
$ docker image pull localhost:5000/ubuntu:bionic
{% endhighlight %}

## 5. Docker add insecure-registries
Docker는 registry 서버가 127.0.0.1 또는 localhost에만 http로 접근한다.  
그 외에는 https로 접근하기 때문에, 서버의 아이피 또는 hostname으로 접근해야되는 경우 insecure-registries를 추가해야한다.  

추가하지 않으면 아래 에러를 볼 수 있다.
{% highlight shell %}
$ The push refers to repository [<ip>:<port>/<image>]
$ Get "https://<ip>:<port>/v2/": http: server gave HTTP response to HTTPS client
{% endhighlight %}

{% highlight shell %}
# run editor
$ vim /etc/docker/daemon.json
# /etc/docker/daemon.json
{
    ...
    "insecure-registries": ["<ip>:<port>"] // json 최상단 
    ...
}
$ systemctl restart docker
{% endhighlight %}


