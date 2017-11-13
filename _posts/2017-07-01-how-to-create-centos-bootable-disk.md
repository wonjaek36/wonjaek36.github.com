---
layout: post
title: "How to Create CentOS Bootable Disk"
date: 2017-07-01 19:09 +0900
category: utils
---

[Universial USB Installer]를 이용해서 CentOS Bootable Disk를 만들었더니,
darcut time initqueue만 잔뜩 나오다가 결국 설치에 실패...

정확한 이유는 모르겠지만, Universial USB Installer로는 CentOS 설치가 안 될 수 있다고 한다.

어느 StackOverflow 페이지에서,(페이지를 종료하는 바람에 링크는 없다.)
Windows의 경우에는 Win32Disk Imager를
Linux의 경우에는 dd명령어를 이용을 권장했다.

이번 포스팅에서는 Linux dd 명령어를 이용하여 CentOS Disk를 만들어본다.

1) "fdisk -l" 을 이용하여 usb의 장치 이름을 알아낸다. (일반적으로 sdc이다.)
{% highlight shell %}
fdisk -l
{% endhighlight %}
2) usb는 mount되어 있으면 안 되므로, mount된 파티션들을 umount한다.
{% highlight shell %}
umount /dev/<device partition name>
{% endhighlight %}
3) dd 명령어를 이용하여 cd image파일을 usb에 복사한다.
{% highlight shell %}
dd BS=4M if=<path to image file> of=/dev/<device name>
{% endhighlight %}
중요한 것은 <device name>이 <device partition name>이 아니란 것이다.
예를 들어, usb의 <device name>이 sdc라면 usb의 파티션 네임은 sdc1, sdc2, ... 이다.

4) 생성된 Usb로 CentOS 설치

[Universial USB Installer]: https://www.pendrivelinux.com/universal-usb-installer-easy-as-1-2-3/
