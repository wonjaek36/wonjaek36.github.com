---
layout: post
title: "Create Pyenv Image with Alpine"
date: 2021-07-18 20:48 +0900
tags: [docker, python]
---

개발을 하다보면, 내부망, 즉 인트라넷에서 업무를 해아할 때가 있다.  
이럴 때 환경을 설정하는 것이 정말 힘든데, 여기에 버전이 다른 두 프로젝트를 진행하면 고달프다..ㅜㅜ  

개발 환경을 최대한 통일하고, deploy도 쉽게 하기 위해서 pyenv를 설치한 Docker를 개인적으로 설정했다.  
(Docker Official Python을 이용해도 된다.)  


Alpine Docker의 버전은 3.14를 사용하였고, pyenv는 master branch를 사용했다.
pyenv의 설치 위치는 "/.pyenv" 이다.  

파이썬 설치 및 사용은 아래 주석을 활용하면 된다.  

{% highlight docker %}
FROM alpine:3.14

RUN apk add curl \ 
    ca-certificates \
    bash \
    git \
    openssl-dev \
    readline-dev\
    bzip2-dev \
    sqlite-dev \
    ncurses-dev \
    linux-headers \
    build-base \
    zlib-dev \
    make

RUN git clone https://github.com/pyenv/pyenv.git /.pyenv
RUN cd /.pyenv && src/configure && make -C src

ENV PYENV_ROOT="/.pyenv"
ENV PATH="$PYENV_ROOT/bin:$PATH"
ENV PATH="$PYENV_ROOT/shims:$PATH"

# ENV PYTHON_VERSION="3.5.10"
# RUN pyenv install $PYTHON_VERSION
# RUN pyenv global $PYTHON_VERSION
# RUN python -m pip install numpy==1.11.1 pandas==0.18.1
{% endhighlight %}







