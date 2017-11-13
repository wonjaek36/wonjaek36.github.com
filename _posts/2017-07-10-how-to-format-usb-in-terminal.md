---
layout: post
title: "How to Format USB in terminal(Ubuntu)"
date: 2017-07-10 10:13 +0900
category: utils
---

Ubuntu에서 가끔 disk util로 usb를 포맷하지 못 하는 경우가 있다.
이런 경우에는 다음과 같이 터미널에서 포맷하면 된다.

{% highlight shell %}
sudo fdisk -l #find <device name>
sudo umount /dev/<device name>
sudo mkdosfs -F 32 -I /dev/<device name>
{% endhighlight %}

Reference: [How to format USB flash disk using ubuntu terminal][usb-format]

[usb-format]:	https://askubuntu.com/questions/662935/how-to-format-usb-flash-disk-using-ubuntu-terminal