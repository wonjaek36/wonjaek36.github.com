---
layout: post
title: "Python string to int type cast"
date: 2019-01-08 23:15 +0900
category: []
tags: [python]
---

파이썬에서는 숫자로 구성된 문자열을 정수로 변환할 때, 아래 코드와 같이 구현한다.  

{% highlight python %}
try:
    value = int(value)
except ValueError as e:
    print (e)
{% endhighlight %}

int 대신에 float을 사용하여 실수로도 변환을 시도할 수 있다.