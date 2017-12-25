---
layout: post
title: "clisp chapter2"
date: 2017-09-09 11:50 +0900
category: clisp
tags: [clisp]
---

<h4>1. 전역 변수 정의</h4>
<p>
"defparameter" 명령어<br />
[1]> (defparameter *small* 1)<br />
*SMALL*<br />
[2]> (defparameter *big* 100)<br />
*BIG*<br />
<br />
"defvar" 명령어<br />
defvar는 defparameter와 달리 재정의하는 경우에 기존의 값을 덮어쓰지 않는다.<br />
[5]> (defparameter *foo* 6)<br />
*FOO*<br />
[6]> *foo*<br />
6<br />
[7]> (defvar *foo* 7)<br />
*FOO*<br />
[8]> *foo*<br />
6<br />
</p>
<h4>2. 전역 함수 정의</h4>
<p>
"defun" 명령어<br />
(defun (function_name (arguments)<br />
... )<br />
<br />
[9]> (defun guess-my-number ()<br />
(ash (+ *small* *big*) -1))<br />
GUESS-MY-NUMBER<br />
<br />
Common Lisp REFL은 모든 명령어는 값을 반환.<br />
defun은 정의된 함수명을 반환 -> GUESS-MY-NUMBER가 출력되는 이유<br />
<br />
cf) ash는 shift함수. -1을 하면 오른쪽으로 한 칸 shift해준다.<br />
<br />
(guess-my-number)를 실행하면 50이 나오는데, <br />
이것은 lisp의 함수에서는 return을 명시적으로 써주지 않아도 자동으로 맨 마지막 값을 반환하기 때문이다.<br />
또한 화면에 50이 출력되는 것은 함수가 숫자를 출력한 것이 아니라, REPL의 기능 때문에 출력된 것 이다.<br />
<br />
Bigger와 Smaller 정의<br />
<br />
Break 1 [11]> (defun smaller ()<br />
(setf *big* (1- (guess-my-number)))<br />
(guess-my-number))<br />
SMALLER<br />
Break 1 [11]> (defun bigger()<br />
(setf *small* (1+ (guess-my-number)))<br />
(guess-my-number))<br />
BIGGER<br/>
<br />
cf) 여기서 1-, 1+은 lisp의 함수이다.<br />
<br />
Break 2 [12]> (defun start-over()<br />
(defparameter *small* 1)<br />
(defparameter *big* 100)<br />
(guess-my-number))<br />
START-OVER<br />
Break 2 [12]> (start-over)<br />
50<br />
<br />
</p>
<h4>지역 변수 정의하기</h4>
<p>
(let (variable declarations)<br />
...body...)<br />
<br />
Break 2 [12]> (let ((a 5)<br />
(b 6))<br />
(+ a b))<br />
11<br />
</p>
<h4>지역 함수 정의하기</h4>
<p>
(flet ((function_name (arugments)<br />
...function_body...))<br />
...body...)<br />
<br />
Break 2 [12]> (flet ((f (n) <br />
(+ n 10)))<br />
(f 5))<br />
15<br />
<br />
Break 2 [12]> (flet ((f (n)<br />
(+ n 10))<br />
(g (n)<br />
(+ n 3)))<br />
(g (f 5)))<br />
18<br />
<br />
지역함수 정의 내에서 다른 지역함수를 사용하려면 labels를 사용한다.<br />
Break 2 [12]> (labels ((a (n)<br />
(+ n 5))<br />
(b (n)<br />
(+ (a n) 6)))<br />
(b 10))<br />
21<br />
<br />
flet을 사용하는 경우 함수 b는 함수 a를 알지 못 한다.<br />
재귀적인 특성이기 ë문인데, 이는 추후 챕터에서 다시 다룬다.<br />
</p>
