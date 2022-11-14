---
layout: post
title: "PostgreSQL Document Reading - Preface"
date: 2022-11-13 23:32+0900
tags: [postgresql, database]
---

## 1. Motivation
Relational Database는 학과 수업 3학년 때, 들은 뒤로 더 이상 자세히 알아보지도 않고 개발에 필요한 부분만 깔짝깔짝 습득해나가면서 대충 써왔다. 그랬더니 회사에 오니 벅차다... ㅜㅜ
특히, Postgresql를 사용하는데, MySQL과 비슷하면서 다른 점이 있어 곤혹스러울 때가 많았다. OTL

이번 기회에 PostgreSQL을 알아보고자, 공식 문서를 전체적으로 짬짬이 읽어보고 간단하게 정리해보고자 한다! (미래의 나에게 도움이 될 만한 Post가 되길~ :P )
* Document 보고 너무 잘 하게 되서, 다시 안 봤으면 한다. ㅋㅋㅋ

본 글에서는 PostgreSQL 14 버전의 Preface를 다루고, 차차 Chapter 2, 3, ... 등으로 글을 계속 이어나갈 예정이다.
Link: [https://www.postgresql.org/docs/14/index.html](https://www.postgresql.org/docs/14/index.html)

## 2. Preface
### 2.1. What is PostgreSQL?
Postgres(ver 4.2) 기반의 Object-Relational Database Management System(ORDBMS).
Postgres는 University of California at Berkeley의 컴퓨터 과학부에서 개발하였고, RDBMS에서 사용될 많은 컨셉들을 개발하였음.

PostgreSQL은 오픈소스로 아래 일반적인 RDBMS에서 지원하는 기능을 지원
* complex queries
* foreign keys
* triggers
* updatable views
* transactional integrity
* multiversion concurrency control

User에 의해서 아래 기능들이 추가될 수 있음(extension 인가?)
* Data types
* Functions
* Operators
* Aggregate Functions
* Index Methods
* Procedural Languages

이제, 각 기능에 대해서 문서를 읽어가면서 알아볼 예정

### 2.2. Brief History of PostgreSQL
#### 2.2.1. The Berkeley Postgres Project

POSTGRES
1. 1987년 demo 출시
2. 1988년 ACM-SIGMOD Conf Version 1
3. 1990년 Version 2 출시
4. 1991년 Version 3 출시
5. Postgres95 출시전까지는 Reliability와 Portability에 집중
  * Postgres의 시스템은 상업적으로도 사용되었는데, 특히 Illustra Information Technologies에서 POSTGRES를 사용하여 Commercialized 함.
  * Illustra는 나중에 Informix에 합병되고, 현재는 IBM이 Informix 소유

Postgres95
* 1994년 Postgres95에 Andrew Yu와 Jolly Chen이 SQL 언어를 Postgres에 추가
* 이 때, Postgres95는 웹에 오픈소스로 공개되었음.
* Postgres95는 ANSI C로 작성되었고, 코드를 25%로 줄이고, 위스콘신 벤치마크에서 POSTGRES보다 속도를 30~50%를 개선
* 또한 Postgres95에서 아래 기능들이 추가/개선되었다
  * PostQUEL -> SQL로 변경(libpq was named after PostQUEL)
  * psql(interactive SQL) 제공
  * TCL-based clients 제공(libpgctl) - 무엇인지 잘 모르겠음...
  * Large-object interface 시스템 개선됨
  * Instance-level rule removed - 무엇인지 잘 모름... 긁
  * GNU make로 build 가능 / GCC로 컴파일이 가능하다!

PostgreSQL
* 1996년에 Postgre95에서 PostgreSQL로 변경하고, Versioning을 6.0부터 시작
* 많은 사람들이 Postgres라고 사용하는데, Alias 또는 Nickname으로 취급 (이제부터 여기서도 Postgres라고 명)
* Postgres95에서는 서버 코드의 문제점을 찾아내고, 이해하고, (해결하는 것이) 주된 개발이었다면, PostgreSQL에서는 feature와 capabilities를 많이 추가(augmented)하는 것이 주된 개발!

### 2.3. Convention
* Brakets - []: Optioanl parts를 의미
* Braces - {} or Vertical Line - | : Choose alternative(One)
* Dots(...) - preceding element can be repeated
* Admin: Postgres를 설치하고 관리하는 사람(DBA)
* User: Postgres를 사용하는 사람(서버 개발자, 데이터 엔지니어 등등)

* [Further Information](https://www.postgresql.org/docs/14/resources.html)은 위키(FAQ)와 웹사이트 등에 대해서 설명
* [Bug Report Guidelines](https://www.postgresql.org/docs/14/bug-reporting.html)에서는 버그의 의미와, 리포트 하는 곳에 대해서 설

## Reference
* [PostgreSQL Document Tables](https://www.postgresql.org/docs/14/index.html)