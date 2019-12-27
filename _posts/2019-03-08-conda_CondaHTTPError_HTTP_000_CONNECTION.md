---
layout: post
title: "TS-CondaHTTPError: HTTP 000 CONNECTION FAILED for url"
date: 2019-03-08 14:42 +0900
category: []
tags: [conda, windows]
---

#### Environment
* conda: version 4.5.12
* windows 10

#### Situation
(학교 과제를 위해) miniconda를 설치하고, conda install numpy를 했을 때!

{% highlight shell %}
Solving environment: failed 

CondaHTTPError: HTTP 000 CONNECTION FAILED for url <https://repo.anaconda.com/pkgs/free/noarch/repodata.json.bz2>
Elapsed: - 

An HTTP error occurred when trying to retrieve this URL.
HTTP errors are often intermittent, and a simple retry will get you on your way.

If your current network has https://www.anaconda.com blocked, please file
a support request with your network engineering team.

SSLError(MaxRetryError('HTTPSConnectionPool(host=\'repo.anaconda.com\', port=443):   
Max retries exceeded with url: /pkgs/free/noarch/repodata.json.bz2
(Caused by SSLError("Can\'t connect to HTTPS URL because the SSL module is not available."))'))
{% endhighlight %}

와 같은 에러를 만났다...  
구글링으로 알아보니 윈도우에 openssl 라이브러리가 없어서 문제가 발생하는 것이라고 한다.([Git issue](https://github.com/conda/conda/issues/6007))

#### TroubleShooting

해결방법은 openssl를 window에 설치하면 된다.  
[Win Openssl Download](http://slproweb.com/products/Win32OpenSSL.html)에서 설치하고 conda를 시도해보면,  
정상적으로 작동하는 것을 확인할 수 있다. (저는 1.1.1b 를 설치하고 진행)

#### Reference
* Git issue: [https://github.com/conda/conda/issues/6007](https://github.com/conda/conda/issues/6007)  
* Win Openssl Download: [http://slproweb.com/products/Win32OpenSSL.html](http://slproweb.com/products/Win32OpenSSL.html)  