---
layout: post
title: "[Architecture] 파이썬으로 구현하는 클린 아키텍처 (2)python 프로젝트 구현하기"
subtitle:
categories: Architecture
tags: []
---

## 들어가며
이전 포스트에서 "클린 아키텍처"에 대해 간단하게 알아봤다. 이번 포스트에서는 객실을 대여하는 Rent-O-Matic이라는 프로젝트에서 **객실(Room)을 조회하는 기능**을 구현하며, 클린 아키텍처의 계층별 시스템 구조와 의존성 방향에 대해 정리한다. 코드는 [여기서](https://github.com/aohus/rentomatic/tree/second-edition) 볼 수 있다. 

## Rent-O-Matic
Rent-O-Matic은 REST API를 통해 Room 이라는 데이터를 조회(GET)할 수 있는 기능을 제공한다. 
기능은 아래와 같다. 
- GET /rooms 요청에 대해, 데이터를 실어 응답한다.
- GET 요청의 파라미터로 조건(filter)을 지정해 원하는 조건의 객실만 조회할 수 있다.

## 전체 구조


## 계층별 설명

### 1. 엔터티 계층
`rentomatic/domain/room.py`

### 2. 유스케이스 계층
- `rentomatic/use_cases/room_list.py`

- `rentomatic/requests/room_list.py`
`build_room_list_request` 함수는 필터를 검사하고 유효성 검사를 수행한다. 어떤 필터가 가능한지, 가능하지 않는지 결정하는 역할은 비즈니스 로직에 해당한다. 즉, 애플리케이션이 어떤 동작을 해야 하는지를 결정하는 과정이기 때문에 유스케이스 계층이라고 볼 수 있다. 

- `rentomatic/responses.py`
`build_response_from_invalid_request` 함수도 마찬가지로 비즈니스 로직에 해당하기 때문에 유스케이스 계층이라고 볼 수 있다. 

- `rentomatic/repository/repo.py`
저장소에 대한 인터페이스를 제공한다. 유스케이스가 특정 저장소에 접근할 때 사용할 수 있는 "계약"을 정의하는 역할을 한다. 이 인터페이스는 실제 저장소의 구현에 의존하지 않는다. 대신, 인터페이스 어댑터 계층에서 이 인터페이스를 상속받아 구체적인 데이터베이스 구현을 제공한다

### 3. 인터페이스 어댑터 계층
`rentomatic/serializers/room.py`
`rentomatic/repository/memrepo.py`
`rentomatic/repository/postgresrepo.py`

### 4. 프레임워크와 드라이버 계층
`application/app.py`
`application/rest/room.py`


## 마치며
클린 아키텍처에서 사용하는 용어와 계층적인 구조로 인해, 프로젝트 레파지토리의 구조만 봐도 어떤 코드에서 어떤 동작을 할지 예상할 수 있다. 
- `domain/room.py` 는 room이라는 도메인을 다루는 엔티티 계층
- `use_cases/room_list.py` 는 실제 비즈니스 로직을 다루며, room list를 반환하는 유즈케이스 계층
- `repository/postgresrepo.py` 는 postgresql의 스토리지 인터페이스를 제공하는 외부 시스템 계층

클린 아키텍처를 설계하고 코드를 작성해나가는데 이 책은 TDD 방법론을 사용했다. 계층별 격리와 테스트가 잘 작성되어있어 TDD를 맛보기에도 좋은 책인것 같다. 나는 요구사항을 보고 코드를 작성 후 책과 비교하며 프로젝트를 진행했는데, 테스트가 습관이 안되어서 자꾸 코드를 작성 후 테스트를 작성하게 되었다. 특히 mock 객체를 사용하는 부분은 익숙하지 않았는데 좀 더 연습을 해봐야 할 것 같다. 


**이전 글**
> [\[Architecture\] 파이썬으로 구현하는 클린 아키텍처 (1)클린 아키텍처란?](https://aohus.github.io/architecture/2024/09/05/clean-architectures-in-python-1.html/)

## Reference  
[Clean Architecture in Python](https://leanpub.com/clean-architectures-in-python)