---
layout: post
title: "Install vim without root for python developer"
date: 2021-12-26 17:17 +0900
category: [linux, vim]
tags: [linux, vim]
---

## 1. Overview
2달 전만 하더라도 emacs를 사용했지만, 운영 서버에서 emacs docker를 띄워서 개발하기는 좋지 않았다...  
결국 다른 분들이 사용하는 Vim을 사용하기로 마음을 먹고 설치를 개인 계정에 설치하고, python 개발에 필요한 라이브러리들을 설치했다  

## 2. Objective
  * Root 권한 없이 Vim(8.2) 설치
  * 여기서는 Vim Plugin을 이용하는 방법에 대해 설명은 하지 않는다  
    * Vim와 jedi(jedi-vim), flake8 등을 사용할 것으로 가정하고 필요한 라이브러리를 설치한다.
  * 라이브러리 설치폴더는 "$HOME/app"에 설치한다고 가정하고, "$HOME/app/bin"은 이미 PATH에 설정되어있다고 본다
  * 아래 과정은 Ubuntu 20.04에서 진행했으며, CentOS 7(세부 버전 확인 못 함)에서도 된다.

## 3. Download

### 3.1 Python Download & Install
{% highlight shell %}
$ git clone https://www.github.com/python/cpython cpython
$ cd cpython  # change working directory
$ git checkout 3.6
$ ./configure --prefix="$HOME/app"
$ make -j <cpu-core number>
$ make install
{% endhighlight %}

* Jedi-vim을 사용하기 위해서 최소 3.6 버전 이어야 한다.

### 3.2 OpenSSL Download & Install
{% highlight shell %}
$ wget https://www.openssl.org/source/openssl-1.1.1m.tar.gz
$ tar -xvf openssl-1.1.1m.tar.gz
$ cd openssl-1.1.1m.tar.gz
$ ./config --prefix="$HOME/app"  # openssl에서 권장하는 설정으로 맞춰짐
$ make -j <cpu-core number>
$ make install
{% endhighlight %}

* 2021.17.26 일 기준 openssl 1.1.1m 버전이 최신(1.x.x 중에서)
* 사내 운영서버에 openssl이 설치되어있다면, 불필요함
  * jedi-vim의 goto(<Leader>g) 기능 등, 코드 분석 기능을 사용하려면 필요

### 3.3 Vim Download & Install
{% highlight shell %}
$ git clone https://github.com/vim/vim.git
$ git checkout <version>  # 원하는 버전 사용, 2021.12.26 기준 8.2 최신버전 사용

# SET CFLAGS & LDFLAGS
$ export CFLAGS="-I$HOME/app/include/python3.6m"
$ export CFLAGS="-I$HOME/app/include/openssl"
$ export LDFLAGS="-rdynamics"

$ ./configure --prefix="$HOME/app" \
--with-features huge \
--enable-multibyte \
--enable-python3interp=yes \
--with-python-config-dir="$HOME/app/lib/python3.6/config-3.6m-x86_64-linux-gnu" \
--enable-cscope

$ make -j <cpu-core number>
$ make install

# Re-compile
## configure 옵션, 또는 CFLAGS, LDFALGS 옵션에 실수가 있어 다시 설정할 때에는
$ rm src/auto/config.cache  
## 를 하고 하면 됨
{% endhighlight %} 

### 4. Reference
* https://stackoverflow.com/questions/5872079/compiling-vim-with-specific-version-of-python
* https://vi.stackexchange.com/questions/26399/installing-vim8-2-with-python3-8-python-h-not-found
* https://github.com/vim/vim/issues/2984
* https://stackoverflow.com/questions/41798118/errorrootcode-for-hash-md5-was-not-found