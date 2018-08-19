---
layout: post
title: "Managing kernels in Jupyter(Python 3.6)"
date: 2018-08-19 20:24 +0900
category: "jupyter"
tags: [python3, jupyter]
---

Environment: Python3.6, Ubuntu Desktop 18.04  
Required: Root privilege(Optional), python Jupyter, python-venv(or virtualenv)    

Jupyter kernel is a kind of environment for jupyter notebook.  
You can import a virtual environment to use specific python version or installed packages.  
I will explain how to add, remove and watch list kernel in this page.

#### 1. Add new kernel in Jupyter
  
{% highlight shell %}
$ source <venv directory>/bin/activate <venv name>
$ pip install ipykernel # You can skip if you already install it
$ python -m ipykernel --user --name <venv name> --display-name <display name>
$ # if you add --user option, it will create kernel in ~\/.local/share/jupyer/kernel
$ # if you don't add it, it will create kernel in ~\/usr/local/share/jupyter (it may require root privilege)
{% endhighlight %} 

#### 2. Watch kernel list in Jupyter

{% highlight shell %}
$ python -m jupyter kernelspec list
Avaiable kernels:
	python3   /usr/local...
	...
{% endhighlight %}

#### 3. Remove a kernel in Jupyter

{% highlight shell %}
$ python -m jupyter kernelspec remove\<venv name\>
$ # You can check venv name to use #2 way.
{% endhighlight shell %}
