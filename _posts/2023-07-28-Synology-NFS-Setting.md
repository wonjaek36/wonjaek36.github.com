---
layout: post
title: "시놀로지 NFS 설정"
date: 2023-07-28 23:18+0900
tags: [synology, linux, ubuntu]
---

Synology에서 친절하게 설명해주니, 자세한 사항은 [여기](https://kb.synology.com/ko-kr/DSM/tutorial/How_to_access_files_on_Synology_NAS_within_the_local_network_NFS)를 참고  
NFS 기능은 다행이 보급형 버전인 J에서도 지원한다. :)

#### Environment
* Ubuntu 20.04 LTS
* Synology 218j (DSM 6.2.4)

#### Synology Part
1. 제어판 &rarr; 파일 서비스 &rarr; 맨 아래 NFS 파트 NFS 활성화 체크(NFS v4.1 미체크)
2. 제어판 &rarr; 공유 폴더 &rarr; (원하는 폴더) 편집 &rarr; NFS 권한에서 생성을 눌러 새로운 권한 모드를 추가
    * 내부 네트워크에서만 사용하기 위해서, IP에 일반적인 공유기의 C 클래스 192.168.0.0/24 할당
    * 리눅스에 별다른 설정 없이 이용하려면,
        * "권한이 없는 포트로부터 연결 허용 (1024보다 높은 포트 사용 체크)"
        * "사용자에게 마운팅된 하위 폴더 엑세스 허용"

#### Linux(Ubuntu) Part
{% highlight shell %}
# Install Packages
$ sudo apt update && sudo apt install nfs-common  # dnf install nfs-utils on RetHat Linux

# Connect
$ sudo mount -t nfs 192.168.0.X:/volumeX/blah /target/directory
{% endhighlight %}
Volume 번호는 DSM의 저장소 관리자에 가서 확인하면 된다. (Volume이 하나라면, volume1)


