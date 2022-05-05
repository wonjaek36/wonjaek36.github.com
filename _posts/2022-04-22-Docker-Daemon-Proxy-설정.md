---
layout: post
title: "Docker Daemon Proxy 설정"
date: 2022-04-18 18:07 +0900
category: [docker]
tags: [docker]
---

## 1. Overview
Docker registry 서버가 proxy 서버를 통해 public network로 나가기 위해서는 도커 daemon proxy 설정이 필수이고, 아래 순서대로 설정을 해주어야 한다.
* 참고) 이번 글은 [공식 문서](https://docs.docker.com/config/daemon/systemd/#httphttps-proxy)와 차이가 없습니다. 

## 2. http-proxy.conf 설정
{% highlight shell %}
# sudo privilege 필요
$ mkdir -p /etc/systemd/system/docker.service.d
$ touch /etc/systemd/system/docker.service.d/http-proxy.conf
$ tee /etc/systemd/system/docker.service.d/http-proxy.conf << EOF
[Service]
Environment="HTTP_PROXY=http://proxy.example.com:80"
Environment="HTTPS_PROXY=https://proxy.example.com:443"
Environment="NO_PROXY=example:25678, localhost, 127.0.0.1, docker-registry.example.com"
EOF
# Proxy 서버가 http만 지원한다면, HTTPS_PROXY에 http 주소를 넣어도 됩니다.(서버 설정에 영향)
# NO_PROXY에는 private registry 주소를 주어, 내부망에서 사용하는 Repo registry를 사용할 수 있도록 설정
# 기본 포트가 아닌 특정 포트를 사용할 경우에는 NO_PROXY에 포트번호도 필요
{% endhighlight %}

## 3. 재시작 및 설정 확인
{% highlight shell %}
$ systemctl daemon-reload  # 변경된 파일 reload
$ systemctl restart docker  # docker 서비스 재시작
$ systemctl show --property=Environment docker
Environment=HTTP_PROXY=... HTTPS_PROXY=... NO_PROXY=...
{% endhighlight %}
