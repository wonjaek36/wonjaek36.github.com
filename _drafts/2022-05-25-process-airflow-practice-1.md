---
layout: post
title: "Airflow 삽질기(1) - QuickStart"
date: 2022-05-25 17:14+0900
tags: [airflow]
---

## 1. Overview
데이터 작업은 일정한 시간을 주기를 가지고 Batch 작업을 할 때가 종종 있다.  
여태 Crontab이나 Jenkins을 사용해왔지만, 작업이 복잡해지고, 시간이 오래 걸리면 Crontab과 Jenkins으로 대응하기는 너무 어렵다. (어흑...)  
특히, Script로 구성된 Batch는 실패한 지점 재시작(Retry)을 할 수 없는게 큰 단점이다.  
  
반면, Airflow는 Batch 작업을 간단한 작업(Task) 여러개로 조각내어 실행하고, 실패한 지점에서 Retry를 수행할 수 있다.  
또한, Task 간의 순서도 DAG로 정의하여, 내가 정의한 작업들을 안전하게 병렬로 수행할 수 있다.  
이렇게 아름다운 장점을 가진 Airflow을 도입하기로 했고, 4월초부터 삽질을 시작했다.  
삽질은 ..

## 2. Airflow
아주 간략하게 나마 사용자 입장에서 Airflow에 대해서 정리하자면,  
Airflow는 사용자가 workflow를 만들고, 실행할 수 있게 관리해주는 플랫폼이다.  
Workflow는 DAG(Directed Acyclic Graph)로 표현하고, DAG의 노드는 Task로 구성된다.  
DAG는 task 간의 dependency를 표현하여, workflow가 사용자가 원하는 순서대로 실행될 수 있도록 한다.   
  
결국 사용자는 DAG(workflow)를 구성하고, 각 Task에 해당하는 작업들을 정의하면 된다.

## 3. QuickStart 
### 3.1 Environment

## 4. Reference
* Airflow Concept: [https://airflow.apache.org/docs/apache-airflow/stable/concepts/overview.html](https://airflow.apache.org/docs/apache-airflow/stable/concepts/overview.htmlo)



