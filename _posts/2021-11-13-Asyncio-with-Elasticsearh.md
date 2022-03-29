---
layout: post
title: "Asyncio with Elasticsearch"
date: 2021-11-13 23:48+0900
tags: [python, elasticsearch, asyncio]
---

## 1. Overview

Asyncio는 Python async/await를 이용한 동시성(Concurrency) Library 이다.  
  
DB(Elasticsearch)에서 많은 데이터를 불러와 변환하고, 저장하는 작업(ETL)를 빠르게 하고 싶다면, 사용하는 것을 고려할만 하다.
다만 벙렬처리가 아니기 때문에 데이터를 변환하는 것을 빠르게 하지 않고, IO 작업을 Coroutine을 이용해 빠르게 (보이게) 해준다.
Python Asyncio의 공식 문서에서도 IO-Bound에 적합하다고 한다.

Python을 6개월만 사용해본 초짜라서 디테일한 설명을 할 능력이 되지 않고(ㅜㅜ), 추후에 다시 이용하기 위해 Template 정도만 정리하여 올린다. Asyncio를 더 깊게 이해하고 싶다면, 좋은 글들을 아래 링크로 걸어 놓으니 관심 있으신 분들은 참고 :) 

* Concurrency와 Parallelism이 헷갈린다면, [Concurrency vs Parallelism - A brief view](https://medium.com/@itIsMadhavan/concurrency-vs-parallelism-a-brief-review-
b337c8dac350) 글을 추천
* Asyncio는 Coroutine([Wiki](https://en.wikipedia.org/wiki/Coroutine)) (함수)를 Event Loop([Wiki](https://en.wikipedia.org/wiki/Event_loop))을 이용하여 실행
  * Wikipedia Corountine에 따르면, non-preemptive multitasking한 subroutine으로 실행하는 동안 멈췄다가 재시작이 여러번 가능한 프로그램 컴포넌트라고 할 수 있다.
  * Event Loop는 Message Bus로 Coroutine을 실행, 정지, 재시작, 그리고 종료를 관리하는 것으로 보인다. **(확실하지 않음..)**
* Python의 yield을 이용한 Generator가 대표적인 Coroutine인데, 이에 대해서는 [How to heck does async/await work in Python 3.5](https://snarky.ca/how-the-heck-does-async-await-work-in-python-3.5)을 참고하면 좋다. - 버전이 낮아서 이는 감안하고 읽어야 함
* Detail한 Asyncio Guide가 필요하면, Mark McDonnell의 [Guide to Concurrency in Python with Asyncio](https://www.integralist.co.uk/posts/python-asyncio/)을 참고하면 좋다. (Python 3.8) 
* Elasticsearch Asyncio 공식 Document Link: [https://elasticsearch-py.readthedocs.io/en/7.x/async.html](https://elasticsearch-py.readthedocs.io/en/7.x/async.html)

## 2. Objective
* High-level Asnyioc with Elasticsearch

## 3. Environment
* docker.elastic.co/elasticsearch/elasticsearch:7.13.2
* python 3.9.6
* elasticsearch[async] == 7.14.0

## 4. High-level Asyncio
Library/Framework 개발자가 아닌, Application 개발자는 Asyncio의 High-level 기능을 사용하는 것을 추천  
주로 Asynio.run(<Awaitable function>, *args)와 tasks를 사용

{% highlight python %}
import asyncio


async def async_task_func():
    response = await IO Request
    return response


async def async_gather_func(requests):
    tasks = []
    for reqeust in requests:
        task = asyncio.create_task(async_task_func(request))
        task.append(task)
    await asyncio.gather(*tasks)

def main():
    result = asyncio.run(async_gather_func)
{% endhighlight %}

event loop를 직접 이용하면, 함수 하나를 걷어낼 수는 있다.
{% highlight python %}
import asyncio

# async_task_func supposed to be same
loop = asyncio.get_event_loop()
tasks = []
for reqeust in requests:
    task = loop.create_task(async_task_func(request))
    task.append(task)
result = loop.run_until_complete(asyncio.gather(*task))
{% endhighlight %}

Task보다 상위개념인 Future 클래스를 사용하는 것보다는 Task를 사용하는 것을 권장!

## 5. Asyncio with Elasticsearch

Asyncio와 주로 사용하는 Search, Insert(Update), Delete 함수를 정의  
insert(update)와 delete는 elasticsearch에서 제공하는 helpers 모듈을 활용함
{% highlight python %}
from elasticsearch import helpers 

async def search(conn, index, query):
    """Asyncio를 이용한 Elasticsearch Search 함수
    Arguments
    conn (AsyncElasticsearch) - Asyncio Elasticsearch Connection
    index (str) - Index 네임
    query (dict) - query
    """
    responses = await conn.search(
        index=index,
        body=query)

    return responses

async def insert(conn, index, docs):
    """Asyncio를 이용한 Elasticsearch Insert 함수
    필요에 의해 upsert를 사용했지만,필요에 따라 doc_as_upsert를 변경하시면 됩니다.
    cconn (AsyncElasticsearch) - Asyncio Elasticsearch Connection
    index (str) - index 이름
    docs (list) - id 값을 포함한, document의 list
    """
    def reform(doc):
        _id = doc.pop('_id') # 상황에 따라서 제외
        reformed = {
        	'_op_type': 'update',
        	'_index': index,
        	'_id': _id,
        	'doc': doc,
        	'doc_as_upsert': True
        }
        return reformed

    reformed_doc = [reform(doc) for doc in docs]
    try:
    	await helpers.async_bulk(conn, reformed_docs)
    except Exception as err:
        self.logger.error(err)

async def delete(self, index, ids):
    """ Asyncio를 이용한 Elasticsearch Delete 함수

    conn (AsyncElasticsearch) - Asyncio Elasticsearch Connection
    index (str) -- index 이름
    ids(list) -- 삭제할 document의 id
    """
    def reform(_id):
        reformed = {
            '_op_type': 'delete',
            '_index': index,
            '_id': '_id'
        }
        return reformed

    reformed_id = [reform(_id) for _id in ids]
    try:
        await helpers.async_bulk(conn, reformed_id)
    except Exception as err:
        self.logger.error(err)
{% endhighlight %}

정의한 함수를 이용하여, Elasticsearch에 Asyncio를 이용하여 동시에 요청하고 싶다면,  
asyncio.create_task와 asyncio.gather 함수를 이용
{% highlight python %}
import asyncio
from elasticsearch import AsyncElasticsearch
# Create Connection
conn = AsyncElasticsearch(
	[127.0.0.1],
	port=9200)

insert_task = asyncio.create_task(insert(conn=conn, index='index', docs=docs))
search_task = asyncio.create_task(search(conn=conn, index='index', query=query))

tasks = [insert_task, search_task]
result = loop.run_until_complete(asyncio.gather(*tasks))
{% endhighlight %}


#### History
* 2022.03.29 imports 추가