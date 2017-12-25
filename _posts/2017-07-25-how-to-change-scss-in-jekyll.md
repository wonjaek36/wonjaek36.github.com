---
layout: post
title: "Jekyll SCSS 수정"
date: 2017-07-25 19:04 +0900
category: jekyll
tags: [jekyll]
---

원하는 jekyll theme가 딱히 없어서 커스터마이징을 하려는데, 상당히 어렵다.<br />
이번 포스트에서는 scss에 관해서만 작성해본다.(html 관련은 나중에 작성)<br />

현재 테마의 원본파일의 위치를 다음과 같이 찾을 수 있다.<br />
{% highlight shell %}
$ open $(bundle open <thema-name>)
{% endhighlight %}

일반적으로 macOS에서는<br />
/usr/local/lib/ruby/gems/<gems version>/gems/<theme-name><br />
에서 테마 원본 파일들을 찾을 수 있다.

테마파일 내부에 보면,<br />
_sass 폴더를 발견할 수 있는데, 이 폴더가 스타일을 담당한다.(scss는 css의 wrapping language인 것 같다.)<br />

이 폴더를 현재 jekyll 폴더로 복사
_config.yml 파일에 다음 내용을 추가해준다.
{% highlight shell %}
sass:
  sass_dir: _sass
{% endhighlight %}

끝!<br />
Reference: [Override theme default]

[Override theme default]:https://jekyllrb.com/docs/themes/#overriding-theme-defaults

