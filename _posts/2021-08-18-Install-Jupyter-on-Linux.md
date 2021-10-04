---
layout: post
title: "Install Jupyter on Linux(Ubuntu)"
date: 2021-08-18 23:13 +0900
tags: [jupyter]
---

## Environment
* Ubuntu 20.04
* Python 3.9.6(by pyenv)

## Objective
* Jupyter 설치
* Jupyter Config
  * Local 서버 기준
* Ubuntu systemd 등록
* Jupyter theme 설정

## Jupyter 설치
{% highlight bash %}
$ python -m pip install jupyter
{% endhighlight %}

## Jupyter Config
{% highlight bash %}
$ python -m jupyter notebook --generate-config
# jupyter notebook --generate-config도 가능 
{% endhighlight %}

NotebookApp(JupyterApp) Section에서 아래 두 가지는 설정하는 것이 좋음
* c.NotebookApp.notebook_dir
  * 사용자가 생성할 Jupypter Notebook의 root_dir
  * 개인이 원하는 경로 사용하면 됨
* c.NotebookApp.password
  * 주피터에 접속하기 위해 패스워드 설정
  * notebook 내부에 있는 passwd 함수를 이용해서 원하는 패스워드의 해쉬값 계산
  * 아래 'argon2:----'을 복사

{% highlight python %}
from notebook.auth import passwd
passwd
Enter passwd: ********
Verify password: ********
'argon2:--------'
{% endhighlight %}


원격 서버에 주피터를 사용한다면, 아래 site에서 더 자세한 config를 참고
참고: https://jupyter-notebook.readthedocs.io/en/stable/config_overview.html

## Ubuntu systemd 등록
* sudo 권한 필요
* /etc/systemd/system에 jupyter.service 파일 생성

{% highlight bash %}
$ echo -n "[Unit]
Description=Jupyter to Learn Data Science

[Service]
Type=simple
User=wonjaek36
Group=wonjaek36
PIDFile=/home/wonjaek36/.jupyter/jupyter.pid
ExecStart=/home/wonjaek36/.pyenv/shims/python -m jupyter notebook --config /home/wonjaek36/.jupyter/jupyter_notebook_config.py
WorkingDirectory=/home/wonjaek36/Workspace/jupyter
Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target" | tee /etc/systemd/system/jupyter.service

$ systemctl daemon-reload
$ systemctl start jupyter
$ systemctl status jupyter
{% endhighlight %}


## Jupyter theme 설정
* jupyter theme 설치 
* jt command 이용하여 theme 설치

{% highlight bash %}
$ python -m pip install jupyterthemes
$ python -m pip install --upgrade jupyterthemes

$ jt  # pyenv 을 이용하여 설치하면, path에 등록이 안 되어있을 수도 있음
$ find /home/<home dir> -name "jt"  
# 저의 경우에는 ./.pyenv/versions/3.9.6/bin/jt
# 패스를 등록해도되고, 직접 사용해도 무방
$ ./.pyenv/versions/3.9.6/bin/jt -l  # theme list
$ ./.pyenv/versions/3.9.6/bin/jt -t chesterish
{% endhighlight %}
