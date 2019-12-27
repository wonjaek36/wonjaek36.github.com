---
layout: post
title: "Keyboard layout change on Windows"
date: 2019-03-08 15:03 +0900
category: []
tags: [windows, keyboard]
---

#### Environment
* Windows 10

#### Problem & Sovling

표준 키 배열이 아니라 정말 특이한 배열의 키보드(해피해킹, vortex 키보드 등...)을 사용하다보면,  
한/영키를 사용하기 어려울 때가 있다.  

그래서 리눅스와 같이 윈도우에서도 shift + space를 사용하기 위해 레지스트리를 변경하면 된다.  
레지스트리 편집기를 열고(Window key + R -> regedit) [HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\i8042prt\Parameters]로 찾아간다.  

여기서 파라미터 
* LayerDriver Kor을 kbd101c.dll
* OverrideKeyboardSubtype을 5로 변경하면 된다.

그리고 다시 원래 일반적인 키보드 배열로 복구하고 싶다면,
* LayerDriver Kor을 kbd101a.dll
* OverrideKeyboardSubtype을 3으로 변경한다.

#### Reference
윈도우 커뮤니티 답변: [https://answers.microsoft.com/ko-kr/windows/forum/windows_10-ime/%EC%9C%88%EB%8F%84%EC%9A%B010-%ED%99%88/28e6d6dd-7b01-4e41-8d27-a33626cefa4c](https://answers.microsoft.com/ko-kr/windows/forum/windows_10-ime/%EC%9C%88%EB%8F%84%EC%9A%B010-%ED%99%88/28e6d6dd-7b01-4e41-8d27-a33626cefa4c)