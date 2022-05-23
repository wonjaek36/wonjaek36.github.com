---
layout: post
title: "GPG로 파일 암호화하기"
date: 2022-05-19 17:00+0900
tags: [linux, gpg]
---

## 1. Overview
Cloud, git 등에 파일을 올릴 때, 파일을 암호화할 필요가 있을 수 있다. 이 때 Linux gpg tool을 이용하면 암호화할 수 있다.  
이번 글에서는 간단하게, CLI 환경에서 대칭키로 암호화하는 것을 기록한다.

## 2. Environment
* Ubuntu 20.04
* gpg 2.2.19
  * libgcrypt 1.8.5
  * gpg 가 없을 경우 다운로드
    * Package Manager 
      * sudo apt install gpg
    * Source code
      * https://git.gnupg.org/cgi-bin/gitweb.cgi?p=gnupg.git

## 3. Encryption
{% highlight shell %}
$ gpg -c --pinentry-mode loopback --passphrase <password> -o <destination> <file>
{% endhighlight %}
* -c : encryption
* --pinentry-mode loopback : CLI mode 사용 (정확한 정보는 man page 참고)
* --passphrase <password> : password 입력
* -o <destination> : 암호화된 파일의 위치 설정  
  해당 값을 주지 않을 경우, stdout 출력
* <file> : 암호화할 파일 Path
* --armor : encryption이 ascii (hex) 로 생성됨
  * 권장하는 옵션 아님

## 4. Decryption
{% highlight shell %}
$ gpg -d --pinentry-mode loopback --passpharse <password> -o <destination> <file>
{% endhighlight %}
* -d : decryption
* --pinentry-mode loopback : CLI mode 사용
* --passphrase <password> : 암호화할 때 사용된 패스워드 입력
* -o <destination> : 복호화된 데이터 저장할 파일 경로, 설정하지 않으면 stdout
