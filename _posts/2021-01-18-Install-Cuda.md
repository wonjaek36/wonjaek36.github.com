---
layout: post
title: "Install Cuda(Cudnn) on CentOS"
date: 2021-01-18 18:00 +0900
tags: [centos, cuda]
---

[CUDA Official Install Guide](https://docs.nvidia.com/cuda/cuda-installation-guide-linux/index.html)와 [CuDNN Install Guide](https://docs.nvidia.com/deeplearning/cudnn/install-guide/index.html)를 보고 설치. 자세하게 보려면 가이드를 보는게 좋음

### 1. Environment 
  * CentOS 7.9
  * Nvidia RTX 3090

### 2. 컴퓨터 사양 및 gcc 확인


* Nvidia 장비 확인
{% highlight shell %}
$ lspci | grep -i nvidia 
# lspci 명령어가 없을 경우 pci utils 설치
# $ yum install pciutils
{% endhighlight %}


* OS 확인 및 cpu bit 확인
{% highlight shell %}
$ uname -m && cat /etc/*release # OS 확인 

x86_64
CentOS Linux release 7.9.2009 (Core)
NAME="CentOS Linux"
VERSION="7 (Core)"
ID="centos"
ID_LIKE="rhel fedora"
VERSION_ID="7"
PRETTY_NAME="CentOS Linux 7 (Core)"
ANSI_COLOR="0;31"
CPE_NAME="cpe:/o:centos:centos:7"
HOME_URL="[https://www.centos.org/](https://www.centos.org/)"
BUG_REPORT_URL="[https://bugs.centos.org/](https://bugs.centos.org/)"

CENTOS_MANTISBT_PROJECT="CentOS-7"
CENTOS_MANTISBT_PROJECT_VERSION="7"
REDHAT_SUPPORT_PRODUCT="centos"
REDHAT_SUPPORT_PRODUCT_VERSION="7"

CentOS Linux release 7.9.2009 (Core)
CentOS Linux release 7.9.2009 (Core)
{% endhighlight %}


* gcc version
{% highlight shell %}
$ gcc --version
gcc (GCC) 4.8.5 20150623 (Red Hat 4.8.5-44)
...
# gcc가 없다면 설치
yum install gcc -y
{% endhighlight %}


### 3. Cuda & Dependency 설치


* 버전에 맞는 kernel header와 development tool 설치
{% highlight shell %}
$ uname -r
3.10.0-1160.el7.x86_64

$ yum install kernel-devel-$(uname -r) kernel-headers-$(uname -r)
{% endhighlight %}


* Cuda 설치
{% highlight shell %}
$ yum-config-manager --add-repo https://developer.download.nvidia.com/compute/cuda/repos/rhel7/x86_64/cuda-rhel7.repo
# yum-config-manager 없다면,
$ yum install yum-utils

$ yum clean all
$ yum install nvidia-driver-latest-dkms cuda
# dkms, opencl-filesystem, ocl-icd가 없다고 에러 난다면,
$ yum install epel-release

$ yum update
$ yum install -y cuda-drivers

$ nvidia-smi # CUDA Driver의 버전이 얼마인지 확인
{% endhighlight %}


### 4. CuDNN 설치
* CuDNN 다운로드
  * 일단 Guide 문서(https://docs.nvidia.com/deeplearning/cudnn/install-guide/index.html)로 가서 2.2 Downloading cuDNN For Linux를 봄
  * NVIDIA Developer Program에 가입
  * NVIDIA cuDNN homepage로 접속에서 다운로드
  * 나는 RPM으로 설치하는 것 받음

{% highlight shell %}
$ rpm -ivh libcudnn8-*.x86_64.rpm
$ rpm -ivh libcudnn8-devel-*.x86_64.rpm
$ rpm -ivh libcudnn8-samples-*.x86_64.rpm
{% endhighlight %}

* cuDNN 설치 확인
  * cuDNN까지 설치하면, cudnn sample들로 설치 확인 가능

{% highlight shell %}
$ cp -r /usr/src/cudnn_samples_v8/ $HOME
$ cd $HOME/cudnn_samples_v8/mnistCUDNN
$ make clean && make -j 4 # -j 옵션은 자유
$ ./mnistCUDNN

# 설치가 정상적으로 되었다면, Test passed 결과가 나옴
# 설치가 정상적으로 되지 않았다면, 아래의 에러 발생
./mnistCUDNN: error while loading shared libraries: libcublasLt.so.11: cannot open shared object file: No such file or directory

# 1. 버전을 확인해본다.
$ yum list installed *cudnn*
$ nvidia-smi 
# 두 명령어를 이용하여 내가 버전을 옳바르게 설치했는지 확인

# 2. cuDNN의 모듈을 로딩
$ ldconfig -v
{% endhighlight %}