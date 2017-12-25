---
layout: post
title: "Download packages by yum (CentOS)"
date: 2017-09-28 16:10 +0900
category: "utils"
tags: [centos, linux]
---

CentOS에서 rpm packages만 받고 싶을 때 사용하는 명령어.<br />
Dependencies까지 받고 싶다면, 해당 dependecies packages가 설치되지 않은 PC에서 해야된다.<br />

{% highlight shell %}
sudo yum install \
--downloadonly --downloaddir <folder_path> \
<packages>
{% endhighlight %}

Example)
{% highlight shell %}
sum yum install \
--download only --downloaddir $HOME/packages \
gcc-c++
{% endhighlight %}