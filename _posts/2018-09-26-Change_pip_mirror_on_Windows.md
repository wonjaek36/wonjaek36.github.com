---
layout: post
title: "Change pip mirror on Windows"
date: 2018-09-26 20:57 +0900
tags: [python, windows]
---

Environment: Windows 10, Python 3.6.6  
Text Editor: Emacs  
  
파이썬 package를 받을 때, mirror가 외국서버로 설정되어있으면 매우 속도가 느린 경우가 있다.  
이럴 때 카카오 mirror로 변경하면, package를 매우 빠르게 받을 수 있다.  
  

mirror를 변경하기 위해서, C:\\ProgramData\\pip\\pip.ini 파일을 생성하고 텍스트 에디터로 파일 실행  

{% highlight powershell %}
emacs -nw C:\ProgramData\pip\pip.ini
{% endhighlight %}

그리고 다음과 같이 작성하여, 카카오 mirror로 변경한다.
{% highlight shell %}
[global]
index-url=http://ftp.daumkakao.com/pypi/simple
trusted-host=ftp.daumkakao.com
{% endhighlight %}

Reference: [Stackoverflow-16403229](https://stackoverflow.com/questions/16403229/how-do-you-configure-pypi-under-windows)