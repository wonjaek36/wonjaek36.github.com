---
layout: post
title: "How to show post by category
date: 2017-08-24 23:09 +0900
category: "jekyll"
---

jekyll에서 category별로 글을 띄우고 싶다면 다음과 같이하면 된다.
(여기서는 "{%", "%}" 부분은 "-{", "}-"로 치환한다. - 주석처리하는 방법을 몰라서...)

{% highlight shell %}
-{ for category in site.category }-
	-{ for posts in category }-
		-{ for post in posts }-
		-{ endfor }-
	-{ endfor }-
-{ endfor }-
{% endhighlight %}

for문의 순서대로
1) site.category는 site의 모든 카테고리를 가져온다.
2) category의 posts는 해당 카테고리에 있는 포스트들을 보여준다.
3) posts의 post는 posts안의 각 post 정보를 가지고 있다.

