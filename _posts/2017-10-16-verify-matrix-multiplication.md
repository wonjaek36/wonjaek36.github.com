---
layout: post
title: "Verify Matrix Multiplication"
date: 2017-10-16 19:09 +0900
category: "Probability and Computing"
---

<h4>1.3 Application: Verify Matrix Multiplication</h4>
<p>
원소가 {0, 1}로 구성된 Matrix A, B가 있고, C=AB라고 하자.<br />
Matrix 계산결과를 맞는지 틀린지 확인하기 위해서 Matrix를 다시 계산하는 방식은 O(N^3) 시간복잡도를 갖는다.<br />
(Matrix의 개선된 알고리즘을 사용하면, O(N^2.37) ) <br />
<br />
그러나 vector r=(r1, r2, r3, ... , rn) (rk = {0, 1})인 vector r을 이용하여,<br />
C(r^T)=AB(r^T) (^T는 Transpose를 의미) <br />
식으로 검사한다면, O(N^2)에 구할 수 있다. <br />
<br />
그렇지만, 이 알고리즘도 항상 맞는 것은 아니다.<br />
(AB-C) != 0이지만 (AB-C)r = 0이 될 수 있기 때문이다.<br />
<br />
r벡터를 랜덤하게 선택한다고 할 때(원소 r1, r2, r3 ..., rn를 랜덤하게 선택과 동일),<br />
이 랜덤 알고리즘이 잘 못된 답을 줄 확률은 1/2보다 작다.<br />
<br />
Proof)
D = AB-C != 0이고, (AB-C)r = 0이라고 하자.<br />
그렇다면, Dr = 0이라고 할 수 있다.<br />
D != 0이 아니므로 0이 아닌 원소를 반드시 하나 이상 가지고 있다. <br />
편의를 위해 원소 d11이 0이 아니라고 가정하자. <br />
<br />
Dr = 0이므로,<br />
<img src="{{ site.url }}/assets/PAC_1.3.1.png" class="center-image" />
<br />
식을 r1에 대해서 풀어 쓰면, 다음과 같다.<br />
<img src="{{ site.url }}/assets/PAC_1.3.2.png" class="center-image" />
<br />
이 때, 문제를 간소화하기 위해서 r1이전에 r2~rn까지 모든 원소들이 uniformly random하게 결정되어있다고 가정하자.<br />
그러면 r1은 {0, 1} 두 수 중에서 하나의 숫자로 결정되기 때문에, 1/2 확률로 잘 못된 값이 나올 수 있다.<br />
이러한 방식으로 문제를 해결하는 것을 "principle of deferred decisions"라고 한다.<br />
<br />
식으로 표현하자면 <br />
<img src="{{ site.url }}/assets/PAC_1.3.3.png" class="center-image" />
<br />
Law of Probability<br />
<img src="{{ site.url }}/assets/PAC_1.3.LOP.png" class="center-image" />
</p>
