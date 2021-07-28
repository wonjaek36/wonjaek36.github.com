---
layout: post
title: "Install librdkafka on CentOS"
date: 2021-07-28 23:30 +0900
tags: [kafka, librdkafka]
---

{% highlight bash %}
yum install centos-release-scl
yum install devtoolset-7

scl enable devtoolset-7
. /opt/rh/devtoolset-7/enable  # For reboot

yum install zlib-devel
yum install openssl-devel
yum install cyrus-sasl cyrus-sasl-devel
# Skip zstd library

git clone https://github.com/edenhill/librdkafka.git
cd librdkafka && git checkout v1.7.0  # Currently latest Version
./configure
make -j 3 && make install

# python -m pip install confluent-kafka==1.7.0  # Check Installation 
{% endhighlight %}