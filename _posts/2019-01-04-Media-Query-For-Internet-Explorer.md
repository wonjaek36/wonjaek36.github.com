---
layout: post
title: "Media Query for Internet Explorer"
date: 2019-01-04 23:15 +0900
tags: css, "Internet Explorer"
---

Internet Explorer에만 필요한 css를 설정해줘야할 때가 있다.
한국에서는 이런 경우가 많지 않지만, 해외에서는 IE를 쓰는 사람들이 꽤 된다고 하니...

어쨌든, css 옵션을 Edge를 제외하고 Internet Explorer 버전 10 이상에만 적용되게 하기 위해서는 미디어 쿼리를 다음과 같이 작성한다.

{% highlight css %}
@media all and (-ms-high-contrast: none), (-ms-high-contrast: active) {
	/* These css options apply on IE10+ */
	...
}
{% endhighlight %}