---
layout: post
title: "How to use Synology as a WebDAV server(+access through Windows)"
date: 2019-05-06 12:13 +0900
category: [platform, synology]
tags: [synology]
---

### Environment
Synology: DS1819+
Windows: 10
RaiDrive: 1.6.2.416

### Problem & Solving
연구실 인프라를 담당하게 되면서 시놀로지도 관리하게되었다. ㅜㅜ  
문제는 처음에 하면 별 것 아니지만, 나중에 또 하려다보면 매우 귀찮은 일...  

어쨌든 목적은 WebDAV 기능을 이용하여 연구실 사람들이 쉽게 시놀로지 드라이브를 사용할 수 있으면 된다. WebDAV에 대한 내용은 다음 링크에서...  
[WebDav Resources](http://www.webdav.org/)  
[WebDAV Wikipedia](https://en.wikipedia.org/wiki/WebDAV)  


먼저 Synology 패키지 관리에서 WebDav Server를 설치한다.  

![WebDAV server](/assets/Synology_WebDAV/20190506_120823.png){: .center-image}  

설치된 Synology WebDAV를 열고, 다음과 같이 포트를 설정해준다.  
(필요에 따라 https, http만 열어도 된다.)  

![WebDAV setting](/assets/Synology_WebDAV/20190506_120933.png){: .center-image}  

이러면 WebDAV를 시놀로지에서 사용할 수 있다.


#### Windows용 WebDAV 클라이언트  

리눅스 or 맥과 달리 Windows는 WebDAV를 사용하기 위해서 클라이언트를 설치하는 것이 좋다.  

여기서 [레이(RaiDrive)](https://www.raidrive.com/)를 사용해보았다.

![RaiDrive](/assets/Synology_WebDAV/20190506_120958.png){: .center-image} 

이 프로그램을 다운로드 받고, 다음 밑 그림과 같이  
* NAS - WebDAV
* Device - Synology  
를 설정하고, address와 path, account 정보를 입력하고 사용하면 된다.  

![RaiDrive setting](/assets/Synology_WebDAV/20190506_121127.png){: .center-image} 

설정하고 나면 다음과 같이 "내 컴퓨터"에서 네트워크 드라이브가 설정된 것을 확인할 수 있다. (왜 용량은 제대로 나오지 않는지는 잘 모르겠다...)  

![RaiDrive complete](/assets/Synology_WebDAV/20190507_021317.png){: .center-image} 