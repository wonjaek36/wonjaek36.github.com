---
layout: post
title: "Send Email by Ubuntu(Simple)"
date: 2017-08-31 18:46 +0900
category: "utils"
---

리눅스에서 시간이 오래 걸리는 작업을 계속 진행해야만 했다. 작업이 끝나고 알람이 오도록 설정하고 싶어서 리눅스, 특히 우분투에서 메일을 보내는 방법을 알아보았다.<br />

대표적인 방법은 ssmtp를 사용하는 것이지만, 설정할 것이 꽤 많고 대표 메일까지 써야되는 것 같아서 제외했다.<br />
(공동으로 사용하는 PC이기 때문에, 내 개인 이메일을 입력할 시에 문제가 발생할 수 있다. )<br />

{% highlight shell %}
sudo apt-get install postfix mailutils
echo "mail test" | mail -s "test mail title" "example@mail.co.kr"
{% endhighlight %}
로 간단하게 보낼 수 있다.<br />

그러나, 이 방법은 ISP에 의해서 막힐 확률이 매우 높고, 메일 서비스(gmail, naver 등)에는 spam으로 처리될 가능성이 거의 100%이다.<br />
따라서, 본인에게 알람서비스처럼 사용할 때에민 유용하다.<br />
(각 메일에서 설정을 통해 스팸으로 처리되지 않도록 할 수 있다.)<br />

이런 문제들을 피할려면 ssmtp를 사용하는 것이 제일 좋다.






