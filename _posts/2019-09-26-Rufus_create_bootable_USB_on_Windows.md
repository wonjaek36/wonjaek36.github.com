---
layout: post
title: "Rufus: create bootable usb on Windows"
date: 2019-09-26 01:53 +0900
tags: []
---

Link: [https://rufus.ie](https://rufus.ie/)

리눅스 설치를 하려고 bootable USB를 만들어야했고, 나는 주로 Universial USB Installer([https://universal-usb-installer.kr.uptodown.com/windows](https://universal-usb-installer.kr.uptodown.com/windows))을 주로 사용해 왔다.  

근데, Universial USB 프로그램은 CentOS을 설치하지 못 했다......  
아래 에러 메세지가 지속적으로 나오면서 부팅이 되지 않았는데, 그 이유는 정말 모르겠다.  

dracut timeout...

이 부분에 대해서 속 시원하게 설명해주는 글은 아직 발견하지 못 했지만,
해결하는 방법은 Linux에서 dd 명령어를 이용하여 설치 USB를 만들면 된다고 한다.

powershell 또는 cmd가 dd와 비슷한 명령어를 지원하는지 몰라서 포기했었는데,
(Rufus)[[https://rufus.ie](https://rufus.ie/)]를 이용하면 dd 커맨드와 같은 방법을 지원하여 CentOS USB를 만들 수 있다고 한다.  


Partition과 Target System은 상황에 맞게 설정하고, 마지막 Start에서 ISO 옵션 또는 dd 옵션을 설정하여, Ubuntu USB와 CentOS USB를 동시에 필요에 맞춰 만들 수 있다.  

![Rufus dd command](/assets/rufus01.png){: .center-image}