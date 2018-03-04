---
layout: post
title: "Telescoping Constructor Pattern"
date: 2018-03-04 16:55 +0900
category: pattern
tags: [pattern, java]
---

Telescoping constructor 패턴은 필수인자를 받는 생성자를 정의하고, 선택적 인자를 받는 생성자들을 따로 추가하는 방식이다.
최종적으로 모든 선택적 인자들을 다 받는 생성자를 정의하면, 모든 생성자 정의가 끝나게 된다.

#### 장점
* 구현이 비교적 쉽다.
* Thread Safe 하다.

#### 단점
* 선택적 인자가 많아지면, 코드가 매우 길어지고 지저분해진다.
* 어떤 생성자를 써야하는지 class 사용자는 헷갈리기 쉽다.

Telescoping Constructor 예제 코드
{% highlight java %}
public class Broker {
	private String ip; // necessary
	private Integer port; // necessary
	private String name; // optional 
	private List<String> neighbors; // optional 

	public Broker(String ip, Integer port) {
		this(ip, port, ""); // blank name
	}
	public Broker(String ip, Integer port, String name) {
		this(ip, port, name, null);
	}
	public Broker(String ip, Integer port, String name, List<String> neighbors) {
		this.ip = ip;
		this.port = port;
		this.name = name;
		this.neighbors = neighbors;
	}
}
{% endhighlight %}