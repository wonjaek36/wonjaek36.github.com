---
layout: post
title: "TS-Attempt to read or write outside of disk"
date: 2019-02-27 12:05 +0900
tags: [linux, ubuntu]
---

### Environment 
* Ubuntu 18.04

Ubuntu 18.04를 연구실의 오래된 PC에 설치했을 때, 발견했던 문제  
PC는 2010년도 HP PC이고 새로 장착한 하드디스크의 용량은 6TB였다.  
  
Ubuntu를 설치했을 때, 파티션은 매우 단순하게 swap영역 8기가와 나머지 부분은 전부 / 에 줬다.  
(grub 영역은 자동으로 할당)  
  
설치 후 부팅 결과 바로 grub rescue 화면이 나타났고, 문제를 파악하기 위해 / 파티션의 /boot 폴더를 확인하려고 하였는데...
  
{% highlight shell %}
$ ls (hd0, gtp3)/boot 
Attempt to read or write outside of disk hd0 
{% endhighlight %}

에러메시지를 받으면서, /boot 영역이 확인이 불가능했다.  
구글링을 통해 알아보니 Ubuntu 설치할 때 /boot 영역을 따로 파티션을 만들어주고 1G를 할당해주면 해결된다고 한다.  

* 정확한 에러메시지는 다를 수 있음
* 확실하지는 않지만, BIOS에서 하드디스크의 용량을 1.6TB까지 밖에 확인을 못 하였는데, 그것과 관련된 이슈가 아닌가 추측합니다.

참고사이트: [참고1](https://askubuntu.com/questions/397485/what-to-do-when-i-get-an-attempt-to-read-or-write-outside-of-disk-hd0)