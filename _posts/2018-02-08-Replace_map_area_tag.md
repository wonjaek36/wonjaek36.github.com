---
layout: post
title: "반응형 웹 디자인에서 html <map> 태그처럼 하기"
date: 2018-01-28 23:56 +0900
category: [library, jQuery]
tags: [library, jQuery, web]
---

html에서 이미지의 특정영역에 어떤 이벤트가 발생하도록 하려면 [map 태그](https://www.w3schools.com/tags/tag_area.asp)를 사용한다.

{% highlight html %}
<img src="test.git" width="300" height="400" alt="test" usemap="#mapTest">
<map name="mapTest">
	<area shape="rect" coords="0,0,50,40" href="www.helloworld.com" alt="world">
	<area ... >
</map>
{% endhighlight %}

map 태그는 coords에 입력된 좌표에 정확하게 영역을 만들어준다. 위의 코드는 (0,0) (50,40) 영역에 마우스를 클릭하면 www.helloworld.com으로 이동시켜준다.
그렇지만, map 태그는 반응형 웹디자인에서는 문제가 발생한다. 이미지의 사이즈가 width와 height에 입력된 것과 다르게 되면, 작동을 하지 않는다.

반응형 웹디자인에서 이미지 사이즈가 줄어들어도 map 태그처럼 작동하고, 이미지 크기가 줄어든만큼 우리가 정한 특정 영역의 사이즈가 줄어들게 하려면, div 태그를 이용해 노가다를 진행해야한다.

구현하는 방식을 순서대로 설명하면,

1. 전체를 포함하는 div 태그를 만들고, img 태그가 div 태그를 꽉 채우게 한다.
  * 이 때, div 태그의 position 속성은 static이 아니어야한다. (relative 권장)
2. 원하는 위치와 왼쪽 위 좌표와 오른쪽 아래 좌표를 구한다.
3. 전체그림을 기준으로 너비, 높이, (왼쪽 위 좌표를 기준으로) 위치의 비율을 구한다.
  * css에서 width, height, left, top을 이용해 네모의 사이즈와 위치를 결정한다.


쉽게 설명하기 위해서, 이미지의 사이즈가  1000x500이고, \[100,100\] 위치에 50x50 네모에
링크를 걸어야 하는 상황을 가정한다.

html, css, jQuery 코드는 다음과 같이 작성한다.

### HTML
{% highlight html %}
<div id="content">
     <img id="image" />
     <div id="area" /></div>
</div>
{% endhighlight %}

### CSS
{% highlight css %}
#content {
  position: relative;
}
#image {
  width: 100%;
  height: 100%;
}
#area {
  position: absolute;
  top: 10%;
  left: 20%;
  width: 5%;
  height: 5%;
}
{% endhighlight %}

### Javascript(jQuery)
{% highlight javascript %}
$("#area").onclick(function() {
  window.open("www.helloworld.com");
});
{% endhighlight %}

이러한 방식으로 구현하면, 이미지가 창의 크기에 따라 작아지더라도(혹은
커지더라도) area의 크기도 그에 맞게 작아지거나 커지게 된다. 또한 area의 링크도
정상적으로 작동한다.

P.S. 위의 방식은 [dq impact report 페이지](https://www.dqinstitute.org/2018dq_impact_report/)를 개발할 때, 적용한 방식이다. (The Challenge Section 세계지도)
