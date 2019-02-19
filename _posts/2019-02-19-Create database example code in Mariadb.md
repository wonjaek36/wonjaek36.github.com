---
layout: post
title: "Redux Example Code with React"
date: 2019-02-19 18:17 +0900
tags: [mariadb]
---

### Environment
Operating System: Ubuntu 18.04
Mariadb version: 10.1.38-MariaDB-0ubuntu0.18.04.1

### CREATE TABLE Example
{% highlight mysql %}
CREATE TABLE users (
	id VARCHAR(16) NOT NULL,
	passwd VARCHAR(128) NOT NULL,
	name VARCHAR(128)
)
{% endhighlight %}

[Official Document](https://mariadb.com/kb/en/library/create-table/)