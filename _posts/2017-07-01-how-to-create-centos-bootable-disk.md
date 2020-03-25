---
layout: post
title: "How to Create CentOS Bootable Disk"
date: 2017-07-01 19:09 +0900
lastmod: 2020-03-25 20:26 +0900
categories: [linux, centos]
tags: [linux, utils]
---

[Universial USB Installer]를 이용해서 CentOS Bootable Disk를 만들었더니,
[darcut time initqueue](https://ahelpme.com/linux/centos7/centos-7-dracut-initqueue-timeout-and-could-not-boot-warning-dev-disk-by-id-md-uuid-does-not-exist/)만 잔뜩 나오다가 결국 설치에 실패...

정확한 이유는 모르겠지만, Universial USB Installer로는 CentOS 설치가 안 될 수 있다고 합니다.

한 StackOverflow 페이지에서, (실수로 인해 링크가 없음..)
CentOS Bootable USB를 만들 때, Windows의 경우에는 Win32Disk Imager, Linux의 경우에는 dd 명령어를 활용하면 된다고 합니다.

Windows는 새로운 유틸리티를 설치해야되기 때문에, 이번 포스팅에서는 Linux dd 명령어를 이용하여 CentOS Disk를 만들어 보았습니다.

{% highlight shell %}
	fdisk -l # usb 장치의 이름을 알아냄
	umount /dev/<device partition name> # usb가 mount 되어있으면 진행이 안 되므로, umount
	dd BS=4M if=<path to image file> of=/dev/<device name> # copy cd image -> usb 
{% endhighlight %}

중요한 것은 <device name>이 <device partition name>이 아니란 것이다.
예를 들어, usb의 <device name>이 sdc라면 usb의 파티션 네임은 sdc1, sdc2, ... 이다.

이 후에, usb를 이용하여 CentOS 설치

[Universial USB Installer]: https://www.pendrivelinux.com/universal-usb-installer-easy-as-1-2-3/
