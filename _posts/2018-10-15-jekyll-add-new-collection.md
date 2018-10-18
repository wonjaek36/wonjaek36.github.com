---
layout: post
title: "Add new collection"
date: 2018-10-15 12:15 +0900
tags: [jekyll]
---

Environment: Windows 10, ruby 2.4, gem 2.7.6  

새로운 collection을 추가하고자 할 때, \_config.yml 파일 내 collection에 추가  
(이 문서에서는 tag를 추가)  

{% highlight shell %}
...
collection:
	tags: 
		output: true
		permalink: /:collection/:name
...
{% endhighlight %}
  
그리고 collection과 관련된 문서는 \_<collection name>에 저장  
(\_tags 폴더 내에 tag 문서들을 저장)  

P.S. layout에서 개별 포스터의 title이나 date에 접근할 때, collection의 이름이 아닌 page로 접근한다.  
(즉, tag 페이지에서도 tag.title이 아닌, page.title로 문서의 제목에 접근할 수 있다.)