---
layout: post
title: "Builder Pattern"
date: 2018-02-23 16:55 +0900
category: pattern
tags: [pattern, java]
---

빌더(Builder) 패턴은 생성자의 인자가 많을 때, 그리고 다수의 인자들이 필수가 아닌 선택적(optional)할 때 사용하면 편리하다.
선택적 인자들의 기본 값을 빌더가 갖고 선택적 인자를 설정하고 싶을 때만 해당 함수를 호출하면 된다.

또한, 빌더 패턴은 객체가 생성 도중에 사용될 것을 걱정하여 객체를 freezing하는 방법을 사용하지 않아도 된다.
이를 위하여, 빌더 패턴에서는 개발자가 빌더 객체를 통하지 않고 객체를 생성할 수 없도록 하기 위하여, 생성자를 private으로 설정한다.

다음은 빌더 패턴의 예제코드이다. 
{% highlight java %}
public class Broker {
	
	private String ip; // necessary parameter
	private Integer port; // necessary paramter
	private String name; // optional parameter
	private List<String> neighbor; //optional parameter

	public static class Builder {
		private final String ip;
		private final Integer port;
		private String name = ""; //initialize default value
		private List<String> neighbor = NULL; //initialize default value;

		public Builder(String ip, Integer port) {
			this.ip = ip;
			this.port = port;
		}
		public Builder setName(String name) {
			this.name = name;
			return this;
		}
		public Builder setNeighbor(List<String> neighbor) {
			this.neighbor = neighbor;
			return this;
		}

		public Broker build() {
			return new Broker(this);
		}
	}

	private Broker(Builder builder) {
		ip = builder.ip;
		port = builder.port;
		name = builder.name;
		neighbor = builder.neighbor;
	}


	public statid void main(String[] args) {
		Broker aBroker = new Broker.Builder("192.168.0.10", 8082).setName("brokerA")
		.(Lists.asList("192.168.0.11:8082","192.168.0.12:8082")).build();
	}
}
{% endhighlight %}




