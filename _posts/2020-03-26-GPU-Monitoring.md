---
layout: post
title: "Python 기반 GPU 모니터링 앱"
date: 2020-03-26 16:32 +0900
lastmod: 2020-03-26 16:32 +0900
tags: [utils, gpu]
---

Deep Learning 알고리즘을 돌리기 위해서, 일반적으로 NVIDIA 그래픽 카드를 많이 활용합니다. 그리고, 학습 중에 그래픽 카드의 utilization을 관찰 하기 위해서는 보통 nvidia-smi를 많이 사용합니다.

nvidia-smi가 그래픽카드와 관련해서 충분한 정보를 제공하지만, 상황에 따라 더 간단하게 또는 더 자세한 모니터링을 하고 싶을 때가 있습니다. python 기반의 두 모니터링 툴, [gpustat](https://github.com/wookayin/gpustat)와 [glances](https://nicolargo.github.io/glances/)은 nvidia-smi을 대체할 수 있습니다.

### 1. gpustat

gpustat은 nvidia-smi의 더 간편한 버전입니다.  

그래픽 카드 당 한줄로 기본적으로, 그래픽 카드 이름, 온도, utilization, Memory, process 정보를 제공합니다. 몇 가지 옵션을 주면, fan과 codec, power 정보도 받을 수 있습니다. (-a, —show-all) 옵션을 이용한다면, nvidia-smi에서 얻을 수 있는 모든 정보를 받을 수 있습니다.

이 툴의 장점은 눈에 쉽게 들어오도록 색상을 넣어준 것, 그리고 최근 테스트 버전으로 json과 web도 지원하기 시작한 게 아닌가 싶습니다.

<img src="{{ site.url }}/assets/gpustat.png" class="center-image" />
<center>https://github.com/wookayin/gpustat</center>

&nbsp;
### 2. glances

glances는 그래픽 카드 정보를 포함한 모든 시스템의 상태를 보여줍니다. 

CPU, MEM, GPU는 물론, Network와 파일 시스템, 센서정보까지도 제공합니다. Docker 정보도 제공하는 것 같은데, 이는 어떻게 받는지 아직까지는 잘 모르겠네요. 상당히 많은 정보를 제공하기 때문에, 매우 자세한 모니터링이 필요하다면 이 프로그램을 사용하는게 좋습니다.

또한, 이미 안정적으로 웹 버전이 제공하고 있고, influxDB와 연동하여 실시간으로 값이 변하는 것을 저장 및 관찰할 수 있습니다.

<img src="{{ site.url }}/assets/glances.png" class="center-image" />
<center>https://nicolargo.github.io/glances/</center>