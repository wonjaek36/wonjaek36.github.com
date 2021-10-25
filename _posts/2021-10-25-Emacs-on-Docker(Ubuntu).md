---
layout: post
title: "Emacs, Zsh on Docker for Python Project"
date: 2021-10-21 23:41+0900
tags: [emacs, docker]
---

우리 회사는 보안이 강하다. 정말 너무 강하다.
금융권이라서 어쩔 수 없다지만, 생각보다 쉽지 않다.

데이터를 수집하는 코드를 분석하기 위해 Pycharm을 사용하고 Debugging을 하려 했지만, Remote interpreter를 사용할 수가 없었다.
해당 데이터는 지정된 한 서버에서만 받을 수 있었기 떄문에, Pycharm Docker interpreter도 사용할 수가 없다.
결국 서버에서 console 개발환경을 구축하기 시작했다. 하지만 Emacs, zsh 버전이 너무 낮아, 결국 Docker(Ubuntu 20.04)에 설치를 했다.

업무보다 환경설정이 역시 더 어렵다 ㅋㅋ..

### 1. Objective

- Install Emacs & Zsh on Docker(Ubuntu 20.04)
- pesudo-tty로 직접 접근해서 설치
- Docker 파일 사용 버전은 맨 아래(일부 작업은 tty로 진행해야함)

### 2. Download & Setting Ubuntu
#### 2.1 Download Ubuntu(20.04) Docker & Run

{% highlight shell %}
$ docker image pull ubuntu:20.04
$ docker run -it --rm --name <image_name> ubuntu:20.04
# -rm, --name은 optional
{% endhighlight %}

#### 2.2 Download Packages

{% highlight shell %}
$ apt update
# 다운로드 속도가 너무 느리다면,
# /etc/apt/sources.list에서 "[archive.ubuntu.com](http://archive.ubuntu.com/)" -> "[ftp.daumkakao.com](http://ftp.daumkakao.com/)"으로 변경
# sed -i 's/archive.ubuntu.com/ftp.daumkakao.com/g' /etc/apt/sources.list로 변경 가능
{% endhighlight %}

### 3. Install Pyenv

여러 Python Project를 할 때는 Pyenv를 설치하는 것이 편하다.  
[Pyenv Github](https://github.com/pyenv/pyenv) 참고

{% highlight shell %}
# Install Prerequsite Package
$ apt install curl bash git build-essential make
$ apt install zlib1g-dev libssl-dev libbz2-dev libncurses-dev libreadline-dev libsqlite3-dev

# Install pyenv
$ git clone https://github.com/pyenv/pyenv.git ~/.pyenv
$ cd ~/.pyenv && src/configure && make -C src

# Pyenv test
$ pyenv versions # install test
$ pyenv install <version>
$ python --version
Python <version>
{% endhighlight shell %}

### 4. Install & Setting Zsh

#### 4.1 Install Zsh

{% highlight shell %}
$ apt install zsh
$ zsh --version
zsh 5.8 (x86_64-ubuntu-linux-gnu)
{% endhighlight %}

#### 4.2 Install oh-my-zsh

Ubuntu에서 설치하는 방법은 Mac과 거의 흡사하다.   
[Install zsh on Mac](https://wonjaek36.github.io/2020/10/09/zsh-install-on-mac.html) 글 참고

설정하다보면, power10k의 Colorful함이 줄어드는데, console의 색상 범위가 줄었기 때문
docker를 실행할 때, "xterm-256color" 환경변수를 설정해야 함
{% highlight shell %}
$ docker run --rm -it -e "TERM=xterm-256color" --name <name> ubuntu:20.04

# 여태까지 만든 Docker를 저장하고 싶다면, Host에서 아래 명령어 사용
$ docker commit <container_name>
{% endhighlight %}

Ubuntu Docker는 기본 쉘이 bash
chsh로 쉘을 변경하거나 bashrc 마지막 줄에 "exec zsh" 삽입

### 5. Install & Setting Emacs
#### 5.1 Download Emacs & Download Melpa
emacs를 다운로드하고, emacs에서 python project 개발을 위해 많이 사용하는 elpy와 treeview를 지원하는 neotree package를 설치(Melpa 통해서 설치 진행)

{% highlight shell %}
$ apt install emacs gnutls-bin -y # for Redhat gnutls-utils
$ cat <<EOF >> .emacs
(require 'package)
(add-to-list 'package-archives '("melpa" . "https://melpa.org/packages/") t)
(package-initialize)
(global-linum-mode)
(setq linum-format "%5d |")
(setq neo-window-fixed-size nil)
(elpy-enable)
(setq elpy-rpc-virtualenv-path 'current)
EOF

# Run emacs & Initialize Package
$ emacs
# In emacs 
M-x package-refresh-contents
M-x package-install neotree
M-x package-install elpy

# elpy 설치 확인은
M-x elpy-config
# 로 확인
{% endhighlight %}

### 6. Dockerfile
zshrc의 theme 설정과 emacs의 package 설정은 직접 interactive terminal에서 직접 진행  
{% highlight docker %}
FROM ubuntu:20.04

RUN sed -i 's/archive.ubuntu.com/ftp.daumkakao.com/g' /etc/apt/sources.list
RUN apt update

RUN apt install curl libssl-dev build-essential make -y
RUN apt install libreadline-dev bash libbz2-dev libncurses-dev -y
RUN apt install libsqlite3-dev git zlib1g-dev -y 

### Install PYENV

RUN git clone https://github.com/pyenv/pyenv.git /root/.pyenv
WORKDIR /root/.pyenv
RUN src/configure
RUN make -C src

ENV PYENV_ROOT=/root/.pyenv
ENV PATH="$PYENV_ROOT/bin:$PATH"
ENV PATH="$PYENV_ROOT/shims:$PATH"

RUN pyenv install 2.7.18
# RUN pyenv install 3.9.6
# RUN pyenv install 3.5.12

# Install Zsh

RUN apt install zsh -y
RUN sh -c "$(curl -fsSL https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
RUN git clone https://github.com/zsh-users/zsh-autosuggestions ~/.oh-my-zsh/custom/plugins/zsh-autosuggestions
RUN git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ~/.oh-my-zsh/custom/plugins/zsh-syntax-highlighting
RUN git clone https://github.com/romkatv/powerlevel10k.git $ZSH_CUSTOM/themes/powerlevel10k
### Set ~/.zshrc in interactive terminal
### For powerlevel10K, Run Docker with TERM=xterm-256color
### docker run --rm -it -e "TERM=xterm-256color" /bin/bash
ARG DEBIAN_FRONTEND=noninteractive
RUN apt install emacs -y
WORKDIR /root
RUN echo '(global-linum-mode)' >> '.emacs'
RUN echo '(setq linum-format "%5d |")' >> '.emacs'
RUN echo '(setq neo-window-fixed-size nil)' >> '.emacs'
RUN echo '(elpy-enable)' >> '.emacs'
RUN echo "(setq elpy-rpc-virtualenv-path 'current)" >> '.emacs'
{% endhighlight %}