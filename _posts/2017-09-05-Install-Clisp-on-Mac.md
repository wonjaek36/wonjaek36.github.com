---
layout: post
title: "Install clisp on Mac OS"
date: 2017-09-05 23:09 +0900
category: [language, common_lisp]
tags: [mac, language, common_lisp]
---

별 것 아니지만, 그래도 기록 남겨놓아아지. <br />

clisp은 가장 일반적인 lisp 타입 중 하나로 정식 명칭은 "ANSI Common Lisp" 이다.<br />
mac에서 install하는 방법은 두 가지가 있는데, 여기서는 brew 명령어를 이용해서 설치한다.<br />
(사실 블로깅할 내용도 아니지만...)<br />

먼저 homebrew를 설치하고 바로 clisp을 사용하면 된다.<br />
{% highlight shell %}
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
brew install clisp
{% endhighlight %}
