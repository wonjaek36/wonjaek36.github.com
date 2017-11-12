---
layout: post
title: "Format USB by dd command"
date: 2017-10-04 12:16 +0900
category: "utils"
---
<p>
dd를 이용해서 CentOS USB를 만들고 나면 USB를 Windows나 Ubuntu disk utility로 partition을 제대로 인식하지 못한다.<br />
정확한 이유는 잘 모르겠지만, dd 커맨드가 파티션을 아예 지워버리는 것 같다.<br />
해서 다시 파티션을 잡기 위해서는 다음과 같이 하면 된다.<br />
</p>

<p>
{% highlight shell %}
umount /dev/<partition-name> #if device is mounted

sudo dd if=/dev/zero of=/dev/<device-name> bs=512 count=32 #test device it works fine-can be skipped
sudo dd if=/dev/zero of=/dev/<device-name> bs=1M

dmesg #check kernel message
{% endhighlight %}
</p>
<p>
usb format은 완료되었고, 다음에 파티션 생성을 하면 된다.
</p>
<p>
{% highlight shell %}
sudo fdisk /dev/sdc
Command (m for help): n
Select (default p): p
Partition number (1-4, default 1): 1
First Sector (2048-7796735, default: 2048): <push enter>
Last Sector, +sectors or +size{K,M,G} (2048-7796735, default 7796735): <push enter>
Command (m for help): t
Hex code(type L to list codes): 6
Command (m for help): w

sudo mkfs.vfat /dev/sdc1
{% endhighlight %}
</p>

Reference: <a href="https://askubuntu.com/questions/223598/how-to-format-a-usb-stick>AskUbuntu</a>

