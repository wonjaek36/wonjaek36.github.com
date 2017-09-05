---
layout: post
title: "Events and Probability(Chapter One)"
date: 2017-09-06 00:45 +0900
category: "Probability and Computing"
---

대학원 알고리즘 특론 정리노트<br />
책 재목은<br />
Probability and Computing: Randomization and Probabilistic Techniques in Algorithms and Data Analysis<br />
이다.

<h4> 1.1 Application: Verifying Polynomial Identities </h4>
<p>
다음과 같은 다항식이 주어졌을 때,<br />
(x+1)(x-2)(x+3)(x-4)(x+5)(x-6)=x^6-7x^3+25<br />
이 두식이 맞는지 체크하고 싶다면?<br />
<br />
우리는 쉽게 판단할 수 있지만, (이 예제는 맨 마지막 상수항만 계산해봐도 두 식이 다르다는 것을 알 수 있다.) 매우 큰 다항식도 검증할 수 있는 방법을 알아야한다. <br />
먼저 간단한 방법으로 좌변을 우변으로 계산해주는 프로그램을 구현해서 double-checking을 하면 된다.<br />
그렇지만 이 방법은 다음과 같은 두 가지 단점이 있다.<br />
<br />
<strong>1) 기존의 코드와 다를 것이 없다.</strong><br />
- 기존의 코드를 구현한 사람이 또 개발한다면, 같은 버그가 존재할 가능성이 매우 높다.<br />
<strong>2) 속도가 느리다.</strong><br />
- 좌변의 식을 우변으로 바꾸기 위해서는 O(d^2)의 시간복잡도가 소요된다.<br />
-> 맨 마지막 (x-6) 를 제외하고 다 계산했다고 가정해보자.<br />
계산된 식은 5차항이다.<br />
(x-6)와 5차항을 곱하기 위해서는 10번(5*2)의 계산이 필요하다.<br />
<br />
이것을 다시 d차항의 식으로 바꿔말하면,<br />
맨 마지막 항을 계산할 때에는 d*2 = O(d) 만큼 계산해야한다.<br />
그런데 항이 총 d개이므로 O(d^2)의 시간이 필요하다.<br />
- 우리는 이것보다 검증을 더 빠르게 하고 싶다.<br />
</p>
<p>
랜덤을 이용한 두 번째 방법을 알아보자.<br />
좌변 F(x)와 우변 G(x)의 차수를 d라고 하고,<br />
{1 ... 100*d} 집합에서 하나의 수를 선택한다(uniformly하게...).<br />
그리고 그 수를 r이라 정한다.<br />
<br />
숫자 r을 각각 F(x)와 G(x)에 대입하여<br />
F(r) = G(r)이 같으면 두 식은 같고, F(r) != G(r)이면 두 식은 다르다고 판단한다.<br />
<br />
물론 이 방식은 항상 옳바른 답을 주지는 않는다. (이것은 조금 있다가 설명)<br />
그렇지만, 시간 복잡도가 기존의 방식에 비해 한 차수 낮다. (O(d))<br />
먼저 좌변의 식 F(r)을 계산하는 것은 O(d)이다.<br />
기존의 x 에 r을 대입한 후 곱셈을 해나가면 된다.<br />
우변의 식 G(r)을 계산하는 것은 [Horner's method]를 사용하면 된다.<br />
<br />
요 방법은 다음과 같은 경우의 수가 있다.<br />
<strong>1) F(x) = G(x), F(r) = G(r)인 경우</strong><br />
옳바르게 결과가 나왔다. :)<br />
<strong>2) F(x) != G(x), F(r) != G(r)인 경우</strong><br />
역시 옳바르게 나왔다. 알고리즘이 두 다항식이 다르다고 잘 판단했다.<br />
<strong>3) F(x) != G(x), F(r) = G(r)인 경우</strong><br />
이 부분이 바로 오답을 주는 경우이다.<br />
H(x) = F(x) - G(x)라고 했을 때, 함수 H(x)의 해가 존재할 수 있는데, 그 해가 만약에 r일 경우 이 경우가 된다.<br />
이 경우에는 알고리즘이 두 다항식이 같다고 하지만, 실제로는 다르다.<br />

그렇지만 이 알고리즘은 매우 빠르고, 오류의 확률이 낮은 편이다. 이 부분은 챕터 3에서 이어서 설명한다. <br />
</p>


[Horner's method]: https://en.wikipedia.org/wiki/Horner%27s_method