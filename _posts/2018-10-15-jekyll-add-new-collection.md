---
layout: post
title: "Add new collection"
date: 2018-10-15 12:15 +0900
tags: [jekyll]
---

Environment: Windows 10, ruby 2.4, gem 2.7.6  

새로운 collection을 추가하고자 할 때, \_config.yml 파일에 다음과 같이 추가  
이 문서에서는 tag를 추가  

{% highlight shell %}
...
collection:
	tags: 
		permalink: /tags/:path
...
{% endhighlight %}

그리고, 다른 html에서 site.<new collection> 으로 접근할 수 있다.  

