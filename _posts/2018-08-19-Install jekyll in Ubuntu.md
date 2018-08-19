---
layout: post
title: "Install Jekyll in Ubuntu (18.04)"
date: 2018-08-19 20:11 +0900
category: "jekyll"
tags: [ubuntu, jekyll]
---

Environment: Ubuntu 18.04 Desktop
Required: Root privilege 
[Official Documentation](https://jekyllrb.com/docs/installation/)

### 1. Install Ruby
  
{% highlight shell %}
$ sudo apt-get install ruby ruby-gem build-essential
{% endhighlight %} 

### 2. Install Jekyll and bundler

{% highlight shell %}
$ gem install jekyll bundler
{% endhighlight %}

### 3. Install gem files in Jekyll blog

Install plugins described in "Gemfile".  

{% highlight shell %}
$ cd <jekyll blog directory>
$ bundle install 
{% endhighlight %}

### 4. Run Jekyll

{% highlight shell %}
bundle exec jekyll serve
{% endhighlight %}