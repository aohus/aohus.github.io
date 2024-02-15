---
layout: post
title: "[Nginx] Nginx는 어떻게 많은 요청을 처리하는가?"
subtitle:
categories: Nginx
tags: []
---

Nginx의 내부 구조와 요청 처리 방식에 대해 정리해보려고합니다. 
Nginx는 고성능, 고가용성을 제공하는 웹 서버이자 리버스 프록시 서버로, 그 효율성과 확장성 때문에 널리 사용됩니다. 

## Nginx 아키텍처  

#### Master Process  
![Alt text](https://aosabook.org/static/nginx/architecture.png)

#### Worker Process  

####  


## 요청 처리 방식  
Nginx가 요청을 처리하는 방식을 다루기 위해 Linux 2.6+ 이상에서 사용 가능한 epoll 구조체를 통한 요청 처리방식을 살펴보겠습니다.   

Nginx가 실행되면서 생성된 Master Process가 설정 파일을 읽어(보통 `nginx.conf`) Listen Socket과 Worker Process 등을 생성합니다.  
Worker Process는 `epoll_create` 시스템콜을 통해 kernel 공간에 있는 `epoll event table`을 생성하고, `epoll_ctl` 시스템콜을 사용하여 Listen Socket의 file descriptor를 등록합니다.
Worker Process는 kernel 공간에 생성된 Listen Socket을 공유하며, 각각의 `epoll event table`을 가집니다. Listen Socket Queue로 들어온 요청을 어떤 Worker Process에게 전달할지는 운영체제의 메커니즘을 따릅니다. 

그럼 이제 하나의 Worker Process가 어떻게 요청을 처리하는지 자세히 살펴봅시다. 중요한 것은 Listen Socket과 epoll 구조체 간의 상호작용입니다.
새로운 요청이 Listen Socket으로 들어오면 해당 Worker Process로 전달되었다고 합시다. 그럼 Worker Process 는 `accept` 시스템콜을 통해 새로운 연결을 수락합니다. 

## References  
<https://aosabook.org/en/v2/nginx.html>  
<https://cyuu.tistory.com/172>