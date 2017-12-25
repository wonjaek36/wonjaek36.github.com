---
layout: post
title: "clisp chapter3"
date: 2017-09-15 00:54 +0900
category: clisp
tags: [clisp]
---

<h4>LISP에서 코드 구성</h4>
<p>
Lisp에서는 코드를 List로 구현한다.<br />
그 리스트 안에는 심볼, 숫자, 그리고 문자열을 넣을 수 있다.<br />
</p>
<ol type="1">
<li>심볼
<p>
심볼은 문자, 숫자, +-/*=...등 기호로 이루어져있다.<br />
그리고 심볼에서는 대소문자 구분을 하지 않는다.<br/>
따라서 다음과 같은 fooo, FoOo는 구분하지 않는다.<br />
[1]> (eq 'fooo 'FoOo)<br />
T<br />
</p>
</li>
<li>숫자
	<p>
리스트는 실수형과 정수형을 모두 지원한다.<br />
단, 두 수를 구분한다. 즉 1과 1.0을 다른 수로 인식한다.<br />
<br />
다음과 같은 수식에서는 정수형 숫자를 실수형으로 바꾸고 결과를 실수형으로 리턴한다.<br />
[2]> (+ 1 1.0)<br />
2.0<br />
<br />
리스트에서 정수형은 매우 큰 수를 계산할 수 있다.(일반적인 C++이나 JAVA보다)<br />
다음 식은 expt는 53의 53 제곱이다.<br />
[3]> (expt 53 53)<br />
24356848165022712132477606520104725518533453128685640844505130879576720609150223301256150373<br />
<br />
리스트에서 다음과 같이 두 정수를 나눌 때 다른 언어와 달리 유리수의 형태로 줄 수 있다.<br />
[4]> (/ 4 6)<br />
2/3<br />
<br />
다음과 같이 나누는 수가 실수라면 무한소수로 나타낸다.<br />
[5]> (/ 4.0 6)<br />
0.6666667<br />
<br />
	</p>
</li>
<li>문자열
	<p>
문자열은 큰 따음표를 이용하여 표현한다.<br />
[6]> (princ "Tutti Frutti")<br />
Tutti Frutti<br />
cf) princ 함수는 문자열을 출력하는 함수이다.(REFL가 출력하는 부분은 생략)<br />
	</p>
</li>
</ol>

<h4>데이터 모드</h4>
<p>
코드가 아닌 데이터를 만들 때 사용한다.<br />
데이터화 하고 싶은 요소에 '(quotation mark)를 붙여서 사용한다.<br />
[7]> '(expt 2 3)<br />
(EXPT 2 3)<br />
</p>

<h4>리스트</h4>
<p>
리스프에서는 데이터와 코드를 전부 리스트로 관리한다.<br />
리스트는 콘셀 구조로 이루어져있다.<br />
콘셀은 2개의 박스로 구성된 포인터 구조이다.<br />
각각의 박스는 다른 콘셀을 가르키거나 데이터를 가리킬 수 있다.<br />
</p>

<h4>리스트 함수</h4>
<ol type="1">
<li> cons
     <p>
두 데이터를 연결하고 싶다면, cons함수를 사용한다.<br />
cons를 사용하면 메모리를 할당하여 콘셀을 만들고, 각 콘셀 칸에 객체의 reference를 만든다.<br />
[8]> (cons 'chicken 'cat)<br />
(CHICKEN . CAT)<br />
<br />
칸을 빈칸으로 만들고 싶으면 nil을 삽입하면 된다.<br />
nil은 리스트를 끝내는데 사용하는 특별한 심볼이다.<br />
[9]> (cons 'chicken 'nil)<br />
(CHICKEN)<br />
<br />
다음과 같이 여러 object를 연결할 수도 있다.<br />
(cons 'pork (cons 'beef (cons 'chicken ())))<br />
[12]> (cons 'pork (cons 'beef (cons 'chicken ())))<br />
(PORK BEEF CHICKEN)<br />
     </p>
</li>
<li> car
     <p>
콘셀에서 첫 번째 항목을 구할 때 사용한다.<br />
(car '(pork beef chicken))<br />
[13]> (car '(pork beef chicken))<br />
PORK<br />
     </p>
</li>
<li> cdr
     <p>
콘셀에서 두 번째 항목을 구할 때 사용한다.<br />
(cdr '(pork beef chicken))<br />
[14]> (cdr '(pork beef chicken))<br />
(BEEF CHICKEN)<br />
<br />
car, cdr를 이용해서 다음과 같은 함수들도 만들어서 사용할 수 있다.<br />
[15]> (cadr '(pork beef chicken))<br />
BEEF<br />
(cadr은 cdr, car를 순서대로 적용한 함수)<br />
car, cdr를 최대 4개까지 조합한 함수를 사용할 수 있다. (cadadr 이런 식으로)<br />
     </p>
</li>
<li> list
	<p>
cons를 중첩하지 않고 편하게 리스트를 만들 때 사용하는 함수<br />
[18]> (list 'pork 'beef 'chicken)<br />
(PORK BEEF CHICKEN)<br />
	</p>
</li>
</ol>
