---
layout: post
title: "How to Create CentOS Bootable Disk"
date: 2017-07-01 19:09 +0900
categories: utils
---

[Universial USB Installer]를 이용해서 CentOS Bootable Disk를 만들었더니,
darcut time initqueue만 잔뜩 나오다가 결국 설치에 실패...

정확한 이유는 모르겠지만, Universial USB Installer로 안 되는 경우가 종종 있다고 한다.
방법을 찾아보다가 어느 StackOverflow 페이지에서,

Windows의 경우에는 Win32Disk Imager를
Linux의 경우에는 dd명령어를 이용을 권장했다.

그래서 Linux dd 명령어를 이용하여 CentOS Disk를 만들어보았다.
미래의 나에게 도움이 되도록, 순서대로 그 과정을 기록해본다.

1) usb를 Linux에 꽂는다.
2) "fdisk -l" 을 이용하여 usb의 장치 이름을 알아낸다. (일반적으로 sdc이다.)
{% highlight shell %}
fdisk -l
{% endhighlight %}
3) usb는 mount되어 있으면 안 되므로, mount된 파티션들을 umount한다.
{% highlight shell %}
umount /dev/<device partition name>
{% endhighlight %}
4) dd 명령어를 이용하여 cd image파일을 usb에 복사한다.
{% highlight shell %}
dd BS=4M if=<path to image file> of=/dev/<device name>
{% endhighlight %}
여기서 중요한 것은 <device name>이 <device partition name>이 아니란 것이다.
예를 들어, usb의 <device name>이 sdc라면 usb의 파티션 네임은 sdc1, sdc2, ... 이다.

5) 생성된 Usb로 CentOS 설치!

[Universial USB Installer]: https://www.pendrivelinux.com/universal-usb-installer-easy-as-1-2-3/
