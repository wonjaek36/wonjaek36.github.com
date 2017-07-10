---
layout: post
title: "How to Format USB in terminal(Ubuntu)"
date: 2017-07-10 10:13 +0900
categories: utils
---

Ubuntu에서 가끔 disk util로 usb를 포맷하지 못 하는 경우가 있다.
(dd명령어를 이용해 usb를 설정했을 때, 안 되더라고요.)

이런 경우에는 다음과 같이 터미널에서 포맷하면 됩니다.

{% highlight shell %}
sudo fdisk -l #find <device name>
sudo umount /dev/<device name>
sudo mkdosfs -F 32 -I /dev/<device name>
{% endhighlight %}

Reference: [how to format USB Flash Disk using Ubuntu terminal]

[how to format USB Flash Disk using Ubuntu terminal]: https://askubuntu.com/questions/662935/how-to-format-usb-flash-disk-using-ubuntu-terminal
