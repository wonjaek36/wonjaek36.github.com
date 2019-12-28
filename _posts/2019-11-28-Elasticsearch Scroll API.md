---
layout: post
title: "Elasticsearch Scroll API"
date: 2019-11-28 19:03 +0900
category: [database, elasticsearch]
tags: [elasticsearch, elk]
---

#### Environment  
* CentOS 7
* Elasticsearch 6.8

Elasticsearch에서 데이터를 불러오기 위해 Search request를 사용하면, 리턴 개수의 한계가 있다. 공식 문서에서는 single "page" of result를 전달한다고 한다. 그런데 최대 이 개수가 50,000개이기 때문에, elasticsearch index 안에 매우 많은 document가 존재하면 데이터를 전부 받을 수가 없다. 이를 해소하기 위한 방법으로  

1. Scroll
2. Bulk

두 가지 방법이 있다. 이번 글에서는 Scroll에 대해서 알아보고, curl로 사용하는 방법을 정리한다.

Scroll은 real-time process를 지원하려고 나온 기능은 아니고, 매우 큰 batch-process 를 처리하려고 나온 기능이다. 그러다 보니, 매우 큰 데이터를 처리할 때 사용하기 적합하다.

사용 방법은 어렵지 않다. 먼저 search와 동일하게 query문을 작성하면서, scroll 파라미터를 추가한다. scroll 뒤에 주는 파라미터는 scroll이 얼마나 오랫동안 유지되어야하는지 시스템에게 알려주는 시간 값이다. 후술하지만, scroll을 열고, 설정한 시간이 지나면 자동으로 scroll의 자원은 회수된다. Scroll의 시간 파라미터 Time Unit은 Appendix 1에서 기술한다.

{% highlight shell %}
curl -XGET "localhost:9200/twitter/_search?scroll=1m&pretty" -H 'Content-Type:application/json' -d '
{
	"size": 100,
	"query": {
		"match": {
			"title": "elasticsearch"
		}
	}
}
{% endhighlight %}
(cf. 대부분의 예제는 elasticserach 공식문서에서 가져옴.)

그러면 elasticsearch에서 response_body에 _scroll_id 값을 반환하여 준다.
{% highlight shell %}
{
	{
		"_scroll_id" : "DnF1ZXJ5VGhlbkZldGNoBQAAAAAAmGrQFnloekNnaXgyUTBxNXg0ZEFaUl9aZHcAAAAAAJicKRZEZnl4S3o0SlNacWw1OXd1UnlmSjJRAAAAAACYatEWeWh6Q2dpeDJRMHE1eDRkQVpSX1pkdwAAAAAAmJwqFkRmeXhLejRKU1pxbDU5d3VSeWZKMlEAAAAAAJnUahZFbnlYcUI3elNWcVR0WlFaSkNYbkhB",
  		"took" : 2,
  		"timed_out" : false,
  		"_shards" : {
    		"total" : 5,
    		"successful" : 5,
    		"skipped" : 0,
    		"failed" : 0
  		},
  		"hits" : {
    		"total" : 23998,
				...
			}
		}
}
{% endhighlight %}

&nbsp;  
그리고 첫 Request로부터 온 response에 있는 _scroll_id 값을 이용해서 다음과 같이 curl을 요청하면, 그 다음 100개를 전달하여 준다. 약간 주의할 점은 elasticsearch 주소 다음에 index이름을 적을 필요가 없다. 바로 endpoint로 _search/scroll을 바로 입력하면 된다. 그리고 계속해서 scroll 아이디를 이용해서 요청하면, 모든 document를 받을 수 있다.

공식문서에는 POST를 사용하라고 되어있지만, GET을 사용해도 상관은 없다.

{% highlight shell %}
curl -XPOST "localhost:9200/_search/scroll" -H 'Content-Type: application/json' -d '{
	
	"scroll": "1m",
  	"scroll_id" : "DnF1ZXJ5VGhlbkZldGNoBQAAAAAAmGrQFnloekNnaXgyUTBxNXg0ZEFaUl9aZHcAAAAAAJicKRZEZnl4S3o0SlNacWw1OXd1UnlmSjJRAAAAAACYatEWeWh6Q2dpeDJRMHE1eDRkQVpSX1pkdwAAAAAAmJwqFkRmeXhLejRKU1pxbDU5d3VSeWZKMlEAAAAAAJnUahZFbnlYcUI3elNWcVR0WlFaSkNYbkhB"
}
{% endhighlight %}

&nbsp;  
그리고, 마지막으로 자원을 회수하기 위해서 scroll을 지워야한다. 시간 설정을 해놓았기 때문에 scroll이 시간이 지나면 지워지겠지만, scroll이 쌓이는 속도가 제거되는 속도보다 빠르면, 언젠가는 자원이 다 소비되기 때문이다. 

scroll의 자원 삭제는 DELETE method를 이용하여 scroll 아이디를 전달하면 된다.

{% highlight shell %}
curl -XDELETE "localhost:9200/_search/scroll" -H 'Content-Type: application/json' -d '{
 	
 	"scroll_id" : "DnF1ZXJ5VGhlbkZldGNoBQAAAAAAmGrQFnloekNnaXgyUTBxNXg0ZEFaUl9aZHcAAAAAAJicKRZEZnl4S3o0SlNacWw1OXd1UnlmSjJRAAAAAACYatEWeWh6Q2dpeDJRMHE1eDRkQVpSX1pkdwAAAAAAmJwqFkRmeXhLejRKU1pxbDU5d3VSeWZKMlEAAAAAAJnUahZFbnlYcUI3elNWcVR0WlFaSkNYbkhB"
}
{% endhighlight %}

이 정도의 기능만 있으면, scroll을 간단하게 사용하는데는 문제가 없다. 조금 더 많은 기능이 필요하다면, elasticsearch 공식문서 [https://www.elastic.co/guide/en/elasticsearch/reference/current/search-request-body.html#request-body-search-scroll](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-request-body.html#request-body-search-scroll)를 참조하면 된다.

이번 Scroll API는 elasticsearch 공식문서가 더 잘 정리되어 있다. 하지만...역시 영어가 모국어이지 않는 이상, 오해를 문서를 오해하기도 하고 프로젝트에서 사용한 경험을 오래 기억하고 싶기 때문에, 간단하게 정리해서 올린다.

Reference
1. [Elasticsearch 공식문서](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-request-body.html#request-body-search-scroll)

Appendix 1.

Time Unit.

| Unit | Meaning |
| :--- | :------ |
| d | Days |
| h | Hours |
| m | Minutes |
| s | Seconds |
| ms | Milliseconds |
| micros | Microseconds |
| nanos | Nanoseconds |