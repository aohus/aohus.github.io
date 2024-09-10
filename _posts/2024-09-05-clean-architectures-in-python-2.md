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
엔터티 계층은 비즈니스 로직의 중요한 데이터를 다루는 객체이다. 

```rentomatic/domain/room.py
import dataclasses
import uuid


@dataclasses.dataclass
class Room:
    code: uuid.UUID
    size: int
    price: int
    longitude: float
    latitude: float

    @classmethod
    def from_dict(cls, init_dict: dict) -> "Room":
        return cls(**init_dict)

    def to_dict(self) -> dict:
        return dataclasses.asdict(self)
```
- "객실"을 대여하는 서비스인 rent-o-matic에서는 객실을 담는 `Room` 객체가 핵심 엔터티이다.  

### 2. 유스케이스 계층
유스케이스 계층은 핵심 비즈니스 로직을 담고 있는 계층이다. 

```rentomatic/use_cases/room_list.py
from rentomatic.responses import (
    ResponseFailure,
    ResponseSuccess,
    ResponseTypes,
    build_response_from_invalid_request,
)


def room_list_use_case(repo, request):
    if not request:
        return build_response_from_invalid_request(request)

    try:
        rooms = repo.list(filters=request.filters)
        return ResponseSuccess(rooms)
    except Exception as exc:
        return ResponseFailure(ResponseTypes.SYSTEM_ERROR, exc)

```
- `use_cases`에는 조건에 맞는 객실을 필터링하고 성공시 `ResponseSuccess`를 실패시 `ResponseFailure`을 반환하는 **"핵심 비즈니스 로직"**을 담고있다. 
- `room_list_use_case`는 `repo`와 `request`를 받아 동작한다. 
    - 핵심적인 역할은 주입받은 `repo`의 `list` 메소드를 호출하고 filtering 조건에 따른 결과를 받아오는 부분이라고 할 수 있다.
    - 클린 아키텍처의 데이터 흐름에 따라 `repo`는 저장소 wrapper, `list`는 저장소의 interface 라는 것을 예상해 볼 수 있다.
    - 이외 에도 `request`와 filtering 로직 결과에 대한 예외처리가 포함되어 있다. 

```rentomatic/requests/room_list.py
from collections.abc import Mapping


class RoomListInvalidRequest:
    def __init__(self):
        self.errors = []

    def add_error(self, parameter, message):
        self.errors.append({"parameter": parameter, "message": message})

    def has_errors(self):
        return len(self.errors) > 0

    def __bool__(self):
        return False


class RoomListValidRequest:
    def __init__(self, filters=None):
        self.filters = filters

    def __bool__(self):
        return True


def build_room_list_request(filters=None):
    accepted_filters = ["code__eq", "price__eq", "price__lt", "price__gt"]
    invalid_req = RoomListInvalidRequest()

    if filters is not None:
        if not isinstance(filters, Mapping):
            invalid_req.add_error("filters", "Is not iterable")
            return invalid_req

        for key, value in filters.items():
            if key not in accepted_filters:
                invalid_req.add_error(
                    "filters", "Key {} cannot be used".format(key)
                )

        if invalid_req.has_errors():
            return invalid_req

    return RoomListValidRequest(filters=filters)
```
- `build_room_list_request` 함수는 필터를 검사하고 유효성 검사를 수행한다. 어떤 필터가 가능한지, 가능하지 않는지 결정하는 역할은 비즈니스 로직에 해당한다. 
- 즉, 애플리케이션이 어떤 동작을 해야 하는지를 결정하는 과정이기 때문에 유스케이스 계층이라고 볼 수 있다. 


```rentomatic/responses.py
class ResponseTypes:
    PARAMETERS_ERROR = "ParametersError"
    RESOURCE_ERROR = "ResourceError"
    SYSTEM_ERROR = "SystemError"
    SUCCESS = "Success"


class ResponseFailure:
    def __init__(self, type_, message):
        self.type = type_
        self.message = self._format_message(message)

    def _format_message(self, msg):
        if isinstance(msg, Exception):
            return "{}: {}".format(
                msg.__class__.__name__, "{}".format(msg)
            )
        return msg

    @property
    def value(self):
        return {"type": self.type, "message": self.message}

    def __bool__(self):
        return False


class ResponseSuccess:
    def __init__(self, value=None):
        self.type = ResponseTypes.SUCCESS
        self.value = value

    def __bool__(self):
        return True


def build_response_from_invalid_request(invalid_request):
    message = "\n".join(
        [
            "{}: {}".format(err["parameter"], err["message"])
            for err in invalid_request.errors
        ]
    )
    return ResponseFailure(ResponseTypes.PARAMETERS_ERROR, message)
```
- `build_response_from_invalid_request` 함수도 마찬가지로 비즈니스 로직에 해당하기 때문에 유스케이스 계층이라고 볼 수 있다. 


```rentomatic/repository/repo.py
from abc import ABCMeta, abstractmethod
from typing import List

from rentomatic.domain.room import Room


class Repository(metaclass=ABCMeta):
    @abstractmethod
    def list(self, filters: dict = None) -> List[Room]:
        pass
```
- 저장소에 대한 인터페이스를 제공한다. 유스케이스가 특정 저장소에 접근할 때 사용할 수 있는 **"계약"을 정의하는 역할**을 한다. 
- 이 인터페이스는 실제 저장소의 구현에 의존하지 않는다. 대신, 인터페이스 어댑터 계층에서 이 인터페이스를 상속받아 구체적인 데이터베이스 구현을 제공한다

### 3. 인터페이스 어댑터 계층
인터페이스 어댑터 레이어는 "프레임워크와 드라이버" 레이어와 "유스 케이스" 레이어 통신 간 필요한 어댑터 모듈들이 위치하는 곳이다.

"프레임워크와 드라이버" 레이어에서는 REST API로 외부와 데이터를 주고받게 되는데, 이때 **json 데이터**를 사용한다.
한편 "유스 케이스" 레이어에서는 request_objects/ 패키지와 response_objects/ 속한 클래스 형식을 사용한다.
따라서 json <-> request_objects/, response_objects/ 사이에 변환을 담당하는 어댑터 모듈을 여기서 정의한다.

```rentomatic/serializers/room.py
import json


class RoomJsonEncoder(json.JSONEncoder):
    def default(self, obj):
        try:
            to_serialize = {
                "code": str(obj.code),
                "size": obj.size,
                "price": obj.price,
                "latitude": obj.latitude,
                "longitude": obj.longitude,
            }
            return to_serialize
        except AttributeError:  # pragma: no cover
            return super().default(obj)
```
- `Room` 객체를 받아 json 객체로 serialize하는 역할을 한다.

```rentomatic/repository/memrepo.py
from rentomatic.domain.room import Room
from rentomatic.repository.repo import Repository

class MemRepo(Repository):
    def __init__(self, data):
        self.data = data

    def list(self, filters=None):

        result = [Room.from_dict(i) for i in self.data]

        if filters is None:
            return result

        if "code__eq" in filters:
            result = [r for r in result if r.code == filters["code__eq"]]

        if "price__eq" in filters:
            result = [
                r for r in result if r.price == int(filters["price__eq"])
            ]

        if "price__lt" in filters:
            result = [
                r for r in result if r.price < int(filters["price__lt"])
            ]

        if "price__gt" in filters:
            result = [
                r for r in result if r.price > int(filters["price__gt"])
            ]

        return result
```
- 인메모리 저장소: 별도의 데이터베이스를 안 쓰고, 데이터를 메모리에 저장해두는 방식이다.
- 외부 저장소를 결정하기 전까지 사용했으며, `list` 인터페이스로 데이터베이스와 같은 필터링 결과를 제공하도록 구현했다. 

```rentomatic/repository/postgresrepo.py
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker

from rentomatic.domain import room
from rentomatic.repository.repo import Repository
from rentomatic.repository.postgres_objects import Base, Room


class PostgresRepo(Repository):
    def __init__(self, configuration):
        connection_string = "postgresql+psycopg2://{}:{}@{}:{}/{}".format(
            configuration["POSTGRES_USER"],
            configuration["POSTGRES_PASSWORD"],
            configuration["POSTGRES했HOSTNAME"],
            configuration["POSTGRES_PORT"],
            configuration["APPLICATION_DB"],
        )

        self.engine = create_engine(connection_string)
        Base.metadata.create_all(self.engine)
        Base.metadata.bind = self.engine

    def _create_room_objects(self, results):
        return [
            room.Room(
                code=q.code,
                size=q.size,
                price=q.price,
                latitude=q.latitude,
                longitude=q.longitude,
            )
            for q in results
        ]

    def list(self, filters=None):
        DBSession = sessionmaker(bind=self.engine)
        session = DBSession()

        query = session.query(Room)

        if filters is None:
            return self._create_room_objects(query.all())

        if "code__eq" in filters:
            query = query.filter(Room.code == filters["code__eq"])

        if "price__eq" in filters:
            query = query.filter(Room.price == filters["price__eq"])

        if "price__lt" in filters:
            query = query.filter(Room.price < filters["price__lt"])

        if "price__gt" in filters:
            query = query.filter(Room.price > filters["price__gt"])

        return self._create_room_objects(query.all())
```
- 관계형 데이터베이스로 postgres를 사용하여 저장했다. 
- 코드에서 데이터를 직접 다루기 위한 ORM으로는 sqlalchemy를 사용했다.
    - ORM은 Object Relational Mapping의 약자로, 객체와 데이터베이스의 관계를 매핑해주는 도구이다. 
    - 예를 들어 `query.filter(Room.code == filters["code__eq"])` 코드는 데이터베이스의 `SELECT * FROM room WHERE code == {code}` 쿼리와 매핑된다. 

### 4. 프레임워크와 드라이버 계층

#### Controller
Controller는 요청에 대해 적합한 유스 케이스로 라우팅 한다.
이 과정 중에 인터페이스 어댑터 레이어의 Encoder를 이용하여 외부 요청을 유스 케이스에 맞게 변환하고, 유스 케이스로부터의 출력을 외부 응답에 맞게 변환한다.

````application/rest/room.py

```

- 이 프로젝트에서는 Flask 프레임워크를 사용했다.

#### App Factory
```application/app.py
from flask import Flask
from application.rest import room


def create_app(config_name):
    app = Flask(__name__)
    config_module = f"application.config.{config_name.capitalize()}Config"
    app.config.from_object(config_module)
    app.register_blueprint(room.blueprint)
    return app
```
- app 인스턴스를 생성해준다. 


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