---
layout: post
title: "Use ssh-agent in CI/CD"
date: 2022-03-28 00:18 +0900
category: [linux]
tags: [linux, ssh]
---

## 1. Overview

상황에 따라, CI/CD를 위해 docker 또는 runner가 git을 clone 받거나, pip로 설치를 해야하는 경우가 있다.  
이 때, docker의 root와 runner는 사용자 계정이 아니어서, 사용자 계정처럼 ssh-key를 등록할 수 없다. (ㅜㅜ...)

이 때 이용하는 것이 **ssh-agent**!
별도의 ssh-agent 프로세스를 실행하고, Repo에 등록된 ssh-key를 활용하여 git repo를 받을 수 있다.

ssh-agent의 man page의 Description을 보면,
>ssh-agent is a program to **hold private keys used for public key authentication.** Through use of environment variables the agent can be located and automatically used for authentication when logging in to other machines using ssh(1).

ssh-agent는 public key 인증 과정에 사용되는 private-key를 보유(hold)할 수 있다. 이를 이용하여 git repository와 인증과정을 수행할 수 있다.

## 2. Environment

* Linux(Ubuntu, CentOS 관계 X!)
* Gitlab

## 3. Works

ssh-keygen 이용하여 git repo를 다운 받을 키를 생성하고, ssh-agent에 등록하여 git repo를 받아온다.  
아래 shell 스크립트를 참고!  

{% highlight shell %}
$ ssh-keygen -t rsa -f ~/.ssh/id_rsa.<repo>  # 이름은 자유롭게
$ eval $(ssh-agent)
$ echo $SSH_AGENT_PID  # pid 확인용
$ ssh-add ~/.ssh/id_rsa.<repo>
# git clone 또는 pip 설치 수행
$ git clone ... or pip install git+ssh://...

# SSH_AGENT 프로세스를 반드시 종료해준다.
# 종료하지 않고, 자동으로 CI/CD를 할 경우 서버의 process가 limits에 걸릴 수 있음
$ kill -9 $SSH_AGENT_PID
{% endhighlight %}

P.S. Gitlab에서는 repo의 Settings -> Repository Settings -> Deploy Keys에 가서 public key를 등록하면 된다.
