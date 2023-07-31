---
layout: post
title: "시놀로지 NFS 설정 (feat. AutoFS)"
date: 2023-07-28 23:18+0900
tags: [synology, linux, ubuntu]
---

Synology에서 친절하게 설명해주니, 자세한 사항은 [여기](https://kb.synology.com/ko-kr/DSM/tutorial/How_to_access_files_on_Synology_NAS_within_the_local_network_NFS)를 참고  
NFS 기능은 다행이 보급형 버전인 J에서도 지원한다. :)

#### Environment
* Ubuntu 20.04 LTS
* Synology 218j (DSM 6.2.4)

<br />
#### Synology Part
1. 제어판 &rarr; 파일 서비스 &rarr; 맨 아래 NFS 파트 NFS 활성화 체크(NFS v4.1 미체크)
2. 제어판 &rarr; 공유 폴더 &rarr; (원하는 폴더) 편집 &rarr; NFS 권한에서 생성을 눌러 새로운 권한 모드를 추가
    * 내부 네트워크에서만 사용하기 위해서, IP에 일반적인 공유기의 C 클래스 192.168.0.0/24 할당
    * 리눅스에 별다른 설정 없이 이용하려면,
        * "권한이 없는 포트로부터 연결 허용 (1024보다 높은 포트 사용 체크)"
        * "사용자에게 마운팅된 하위 폴더 엑세스 허용"

<br />
#### Linux(Ubuntu) Part
{% highlight shell %}
# Install Packages
$ sudo apt update && sudo apt install nfs-common  # dnf install nfs-utils on RetHat Linux

# Connect
$ sudo mount -t nfs 192.168.0.X:/volumeX/blah /target/directory
{% endhighlight %}
Volume 번호는 DSM의 저장소 관리자에 가서 확인하면 된다. (Volume이 하나라면, volume1)

<br />
#### Automount Part
NFS을 Mount하는 것은 네트워크 상황에 따라 다르다. 그래서 NFS를 /etc/fstab에 올려버리면, 부팅 시에 문제를 야기할 수 있다. (매우 오래걸리거나, Lock이 걸릴 수 있다.)
AutoFS를 이용하면, 사용자가 파일시스템에 접근할 때 비로소 Automount을 해준다. 그러므로, 안전하게 NFS를 쓰자 :)

IP를 사용해도 좋지만, 그래도 Domain Name을 사용하기 위해서 /etc/hosts에 Synology IP를 등록한다.
{% highlight shell %}
$ sudo vim /etc/hosts
# Add below text
<synology ip> synology
{% endhighlight %}

그리고, AutoFS를 설치하고, 버전을 체크한다.
{% highlight shell %}
$ sudo apt install autofs
$ automount --version
Linux automount version 5.1.6
...
{% endhighlight %}

Ubuntu 20.04 에서는 버전 5가 설치된다. (버전 4와 5는 비교적 큰 차이가 있는 것 같으니 4를 사용하실 분은 kernel 문서를 참고해주세요.)

AutoFS에서는 Mount 별로 Map 파일을 구성하고, auto.master에 Map 파일을 등록한다. 두 가지 방법(Direct/Indirect)로 가능한데, 나는 하나의 Synology만 연결하면 되니 Direct를 이용 (여러 Directory를 자동으로 연결하고 싶다면, Indirect)

연결하고자하는 Synology의 Volume은 volume1, Directory는 Target이라고 하고 연결해보면
{% highlight shell %}
$ sudo vim /etc/auto.master
# auto.master 파일에 아래 statement 추가
/synology /etc/auto.synology
{% endhighlight %}

그리고, auto.synology 파일을 생성하고 내용을 추가한다.
{% highlight shell %}
$ sudo vim /etc/auto.synology
# Mounted Directory <Tab> Map File Path
Target -fstype=nfs synology:/volume1/Target
{% endhighlight %}

마지막으로, autofs를 재시학하고, mount를 확인한다.
{% highlight shell %}
$ systemctl restart autofs
$ systemctl status autofs
blah blah

$ mount | grep synology
/etc/auto.synology on /synology type autofs (options...)

$ ls /synology
# Not-Mounted before Access

$ cd /synology/Target
$ ls /synology
Target
{% endhighlight %}

(Optional) 직접 Access를 하기 전에 Directory가 보이지 않는 것이 불편하다면, browse_mode를 켜면 된다. (성능이 떨어지지만, 개인 사용자는 괜찮다. Production과 관계있다면, 조심해서 사용!)
{% highlight shell %}
$ sudo vim /etc/autofs.conf
# Find browse_mode (close to line 51)
browse_mode = yes
{% endhighlight %}

<br />
#### 참고
* [Kernel Document](https://docs.kernel.org/filesystems/autofs.html#:~:text=%22autofs%22%20is%20a%20Linux%20kernel,managed%20by%20the%20same%20daemon)
* [Ubuntu AutoFS](https://help.ubuntu.com/community/Autofs)
* [ArchLinux AutoFS](https://wiki.archlinux.org/title/autofs)
