---
layout: post
title: "Swap disk in Synology"
date: 2019-06-19 09:44 +0900
category: []
tags: [synology]
---


### Environment
Synology: DS1819+

### Objective
WD Red 하드 6TB의 수량을 원하는만큼 확보하지 못 해서, 급한대로 있는 하드들을 DS1819+ 에 다 넣었는데,  
이제는 6TB 하드의 충분한 수량이 확보되어서 기존의 일부 하드를 제거하고 6TB로 맞추려고 합니다 :)

DS1819+ 는 [Hot Swapping](https://en.wikipedia.org/wiki/Hot_swapping) 기능을 지원하기 때문에 Synology 장비의 전원을 끄지 않아도 변경할 수는 있습니다.

밑의 그림에서 2TB(1.82TB) 하드를 6TB로 바꿀려고 합니다.  

![Initial State](/assets/swap_disk_in_synology/1.png){: .center-image width="75%"}

Hot swapping을 지원하기 때문에, 끄지 않고 교체를 하면, 

![Changing Hard](/assets/swap_disk_in_synology/2.png){: .center-image width="75%"}  
에러메세지와 함께 교체되었다고 뜨고...  

위의 "작업"(Action) 버튼에서 "수리" 탭을 누르고 진행을 하면, 원하는 디스크를 선택하고 교체를 진행할 수 있습니다.  
![Repairing](/assets/swap_disk_in_synology/2.png){: .center-image width="75%"}  

끝

### Conclusion

워낙 Synology가 잘 만들어놓은 기능이기 때문에 글로 남길만한 가치가 있을까 싶었는데,  
매번 별 것 아닌 기능을 찾는 것보다는 기록을 해놓는게 좋을 것 같습니다. :)

### Reference
[Hot Swapping](https://en.wikipedia.org/wiki/Hot_swapping)
[Synology storage_pool_expand_replace_disk](https://www.synology.com/ko-kr/knowledgebase/DSM/help/DSM/StorageManager/storage_pool_expand_replace_disk)
