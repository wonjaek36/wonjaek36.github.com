---
layout: post
title: "Send Email by Ubuntu(Simple)"
date: 2017-08-31 18:46 +0900
category: "utils"
---

Cluster를 구성하고 실험을 진행하는데, 이게 언제 끝날지 알 수 없었다.(ㅜㅜ)<br />
그렇다고 매번 확인할 수는 없으니 이번에 ubuntu에서 메일을 보내는 방법을 알아보았다.<br />

대표적인 방법은 ssmtp를 사용하는 것이지만, 상당히 설정할 것이 많고 대표 메일까지 써야되는 것 같아서 제외했다.<br />
(어차피 실험이 끝났다는 사실만 알려주는 메일을 보내면 되니까 여기서는 제외)<br />

{% highlight shell %}
sudo apt-get install postfix mailutils
echo "mail test" | mail -s "test mail title" "example@mail.co.kr"
{% endhighlight %}

로 간단하게 보낼 수 있다.<br />

P.S.<br />
이 방법은 ISP에 의해서 막힐 확률이 매우 높다.<br />
메일 서비스(gmail, naver 등)에는 spam으로 처리될 가능성이 매우 높다.<br />
(gmail의 경우 필터를 정해서 스팸으로 처리되지 않도록 할 수 있다.)<br />

이런 문제들을 피할려면 ssmtp를 사용하는 것이 제일 편하긴 하다.






