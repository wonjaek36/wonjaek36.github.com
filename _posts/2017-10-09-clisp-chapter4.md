---
layout: post
title: "clisp chapter4"
date: 2017-10-09 22:56 +0900
category: clisp
tags: [clisp]
---

<h4>1. True/False </h4>
<p>
다음과 같이 4가지 경우를 Lisp에서는 False로 본다.<br />
'(), (), 'nil, nil<br />

이 이외의 것은 모두 True이다.<br >
item이 있는 list는 참이다.<br >

{% highlight Clojure %}
> (if '()
    'i-am-true
    'i-am-false)
I-AM-FALSE
</p>
{% endhighlight %}

{% highlight Clojure %}
> (if '(1)
    'i-am-true
    'i-am-false)
I-AM-TRUE
{% endhighlight %}

{% highlight Clojure %}
> (eq '() nil)
T
> (eq '() ())
T
> (eq '() 'nil)
T
{% endhighlight %}
첫번 째가 같은 이유는 nil은 홑따음표를 빼도 상관없는 상수로 취급한다.<br>
두번 째는 빈 리스트를 해석할 때는 자연스럽게 저렇게 처리된다.<br>
세번 째는 lisp에서 nil과 빈괄호는 동일한 것으로 처리하기 때문이다.<br>
</p>

<h4> 2. if(조건문) </h4>
<p>
{% highlight Clojure %}
> (if (= (+ 1 2) 3)
    'yup
    'nope)
YUP
> (if (= (+ 1 2) 4)
    'yup
    'nope)
NOPE
{% endhighlight %}

if문을 이용해 list가 비어있는지 확인할 수 있다.<br>
{% highlight Clojure %}
> (if '(1)
    'the-list-has-stuff-in-it
    'the-list-is-empty)
THE-LIST-HAS-STUFF-IN-IT
> (if '()
    'the-list-has-stuff-in-it
    'the-list-is-empty)
THE-LIST-IS-EMPTY
{% endhighlight %}

아래의 if문에서 1을 0으로 나누려는 구문이 있어, 오류가 날 법하지만,
조건이 맞지 않아 실행되지 않으므로 프로그램이 에러가 나지 않는다.
-> if문에서는 두 개의 표현식 중 하나만 실행된다.
{% highlight Clojure %}
> (if (oddp 5)
    'odd-number
    (/ 1 0))
ODD-NUMBER
{% endhighlight %}

if문에서 여러 개의 문장을 사용하고 싶다면 명령어 progn을 사용하면 된다.<br>
{% highlight Clojure %}
> (defvar *number-was-odd* nil)
*NUMBER-WAS-ODD*

> (if (oddp 5)
    (progn (setf *number-was-odd* t)
    	'odd-number)
	'even-number)
ODD-NUMBER
{% endhighlight %}

그러나 위와 같이 매번 progn을 사용하면 불편하므로, progn을 내포하고 있는 명령어 when, unless, cond, case를 지원한다.<br>

<h5>2.1 when</h5>
{% highlight Clojure %}
> (when (oddp 5)
      (setf *number-is-odd* t)
      'odd-number)
ODD-NUMBER
> *number-is-odd*
T
{% endhighlight %}
when은 조건이 참일 경우 내부 표현식을 모두 실행한다.<br>

<h5>2.2 unless</h5>
{% highlight Clojure %}
> (unless (oddp 4)
	(setf *number-is-odd* nil)
	'even-number)
EVEN-NUMBER
> *number-is-odd*
T
{% endhighlight %}
unless는 조건이 거짓을 경우 내부 표현식을 모두 실행한다.<br>

<h5>2.3 cond</h5>
{% highlight Clojure %}
> (defvar *arch-enemy* nil)
*ARCH-ENEMY*
> (defun pudding-eater (person)
       (cond ((eq person 'henry) (setf *arch-enemy* 'lisp-alien)
       	     '(curse you lisp alien ? you ate my pudding))
	     ((eq person 'johnny) (setf *arch-enemy* 'johnny)
	     '(i hope you choked on my pudding johnny))
	     (t '(why you eat my pudding strange ?))))
PUDDING-EATER

> (pudding-eater 'johnny)
(I HOPE YOU CHOKED ON MY PUDDING JOHNNY)
> *arch-enemy*
JOHNNY
>(pudding-eater 'test)
(WHY YOU EAT MY PUDDING STRANGER ?)
{% endhighlight %}

cond는 괄호를 이용해 구간을 나눈다. 괄호 안의 첫 표현식은 구간을 실행하기위한 조건이다.<br>
위의 예제에서는 henry, johnny, 그리고 나머지 경우(t)로 구분하였다.<br>

cond는 위에서부터 차례대로 조건을 비교한다. 마지막 구문에 참 조건(t)가 있어, 두 가지 조건을 다 만족하지 못 하였을 경우 세 번째 조건이 무조건 실행되도록 하였다.<br>

<h5>2.4 case</h4>
case는 cond보다 조금 더 간결하다.<br>

{% highlight Clojure %}
> (defun pudding-eater (person)
  (case person
  	((henry) (setf *arch-enemy* 'stupid-lisp-alien)
		 '(curse you lisp alien ? you ate my pudding))
 	 ((johnny) (setf *arch-enemy* 'useless-old-johnny)
	 	 '(I hope you choked on my pudding johnny))
	(otherwise '(why you eat my pudding stranger ?))))
PUDDING-EATER
> (pudding-eater 'johnny)
(I HOPE YOU CHOKED ON MY PUDDING JOHNNY)
> (pudding-eater 'henry)
(CURSE YOU LISP ALIEN ? YOU ATE MY PUDDING)
{% endhighlight %}

주의) case는 비교할 때 내부적으로 eq를 쓴다. 심볼 값으로 분기할 경우에만 사용한다.<br>
</p>

<h4>3. and/or</h4>
<p>
and와 or는 간단한 수학 연산자이고 불린 값을 만들어낸다.<br>
기본적으로 다음과 같이 사용할 수 있다.<br>

{% highlight Clojure %}
> (and (oddp 5) (oddp 7) (oddp 9))
T
> (or (oddp 4) (oddp 7) (oddp 8))
T
{% endhighlight %}

or나 and를 이용해서 조건문처럼 사용할 수도 있다.<br>
{% highlight Clojure %}
(boolean short circuit evaluation을 통해서 다음과 같이 적용된다.)<br>
> (or (oddp 4) (setf *is-it-even* t))
T
> *is-it-even*
T
{% endhighlight %}

--Member 함수--
Member함수는 아이템이 리스트의 멤버인지 확인한다.<br>
{% highlight Clojure %}
> (if (member 1 '(3 4 1 5))
     'one-is-in-the-list
     'one-is-not-in-the-list)
ONE-IS-IN-THE-LIST
{% endhighlight %}

Lisp에서는 nil이 아니면 모두 참으로 인식하기 때문에,<br>
{% highlight Clojure %}
> (member 1 '(3 4 1 5))
(1 5)
{% endhighlight %}

member함수가 (1 5)를 리턴하여도, true로 판단한다.<br>
</p>

<h4>4. 비교함수</h4>
<p>
비교함수는 다음과 같이 다양하게 있다.<br>
eq, eql, equal, =, string-equal, equalp<br>

저자는 심볼을 비교할 때에는 eq, 다른 것을 비교할 때에는 equal을 권유한다.<br>
비교함수들 중 eq을 가장 간단하고 빠르다. 따라서 심볼을 사용할 때에는 eq을 사용하는 것을 권장한다.<br>

{% highlight Clojure %}
> (defparameter *fruit* 'apple)
FRUIT
> (cond ((eq *fruit* 'apple) 'its-an-apple)
  	((eq *fruit* 'orange) 'its-an-orange))
ITS-AN-APPLE
{% endhighlight %}

두 심볼만을 비교할 것이 이라면 equal을 사용한다.<br>
equal은 두 심볼이 같은 데이터를 가지고 있는지 판단한다.<br>
{% highlight Clojure %}
> (equal 'apple 'apple)
T
> (equal (list 1 2 3) (list 1 2 3))
T
> (equal '(1 2 3) (cons 1 (cons 2 (cons 3 ()))))
T
{% endhighlight %}

eql은 eq에서 숫자와 문자를 비교할 수 있다.<br>
{% highlight Clojure %}
> (eql 'foo 'foo)
T
> (eql 3.4 3.4)
T
{% endhighlight %}

equalp는 equal과 거의 같지만, 더 복잡한 기능을 제공한다.<br>
문자열 비교에서는 대소문자 구분없이 비교하고, 숫자에서는 정수와 소수를 비교할 수도 있다.<br>
{% highlight Clojure %}
> (equalp "Bob Smith" "bob smith")
T
> (equalp 3.0 3)
T
{% endhighlight %}
</p>