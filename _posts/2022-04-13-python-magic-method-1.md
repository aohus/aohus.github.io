---
layout: post
title: "[Python] 매직 메소드(1) __call__, __getattribute__"
subtitle:
categories: Python
tags: []
---

클래스 안에 정의된 함수를 '메소드'라고 부른다. 메소드 중에서 `__`로 시작해서 `__`로 끝나는 메소드들 이 있는데, 이를 매직 메소드 또는 특별 메소드라고 부른다. 가장 유명한 매직 메소드는 `__init__`이라는 생성자이며, 생성자는 어떤 클래스의 인스턴스가 생성될 때 파이썬 인터프리터에 의해 자동으로 호출되는 메소드이다.

instance에서 접근 가능한 메소드와 변수를 보기 위해 'dir' 명령으로 아래와 같이 살펴보면 내가 정의하지 않은 많은 메소드들이 나온다. 클래스 내부에 이미 정의되어있는 매직 메소드들이다.(객체마다 다른 매직 메소드들이 있고 직접 정의할 수도 있다.)

```python
#임의의 인스턴스 이름 'instance'
dir(instance)

>> ['__class__', '__delattr__', '__dict__', '__dir__', '__doc__', '__eq__',
'__format__', '__ge__', '__getattribute__', '__gt__', '__hash__', '__init__',
'__init_subclass__', '__le__', '__lt__', '__module__', '__ne__', '__new__',
'__reduce__', '__reduce_ex__', '__repr__', '__setattr__', '__sizeof__',
'__str__', '__sub__', '__subclasshook__', '__weakref__', '_name', '_price']
```

이 매직 메소드들은 파이썬 코드를 더 쉽고 효율적으로 구현할 수 있도록 도와주지만, 매직 메소드의 동작을 알지 못하는 사람(나..)에게는 코드를 해석하기 어렵게 만드는 요소이기도하다. 이번에는 '파이썬 클린코드'를 읽으며 마주친 매직 메소드를 예시와 함께 소개한다.

### `__call__`

파이썬에서 함수의 호출은 함수의 이름에 '( )'를 붙여주면 된다. 왜 '( )'를 붙여주면 함수가 호출될까? 어떤 클래스(타입)의 객체가 있을 때 '( )'를 붙여주면 해당 클래스에 정의된 매직 메소드인 `__call__`이 호출되기 때문이다.

```python
class MyFunc:
	def __call__(self, *args, **kwargs):
		print("호출됨")
MyFunc            # (1)

f = MyFunc()
f                 # (2)
f()               # (3) == MyFunc()()
```

결과

```python
<class '__main__.MyFunc'>
<__main__.MyFunc object at 0x107512b20>
호출됨
```

혹시 어떻게 사용되는지 궁금하다면(그리고 파이썬 데코레이터에 대해 이해하고 있다면) 조금 응용된 예시를 보자. \[파이썬 클린코드 2nd Edition- Chapter5.데코레이터] 에서는 다음과 같이 사용했다. Serialization class가 선언되고, LoginEvent class를 인자로 받으면서 `__call__` 함수가 호출된다.

```python
from dataclasses import dataclass
from datetime import datetime


def hide_field(field) -> str:
    return "**redacted**"


def format_time(field_timestamp: datetime) -> str:
    return field_timestamp.strftime("%Y-%m-%d %H:%M")


def show_original(event_field):
    return event_field


class EventSerializer:
    def __init__(self, serialization_fields: dict) -> None:
        self.serialization_fields = serialization_fields

    def serialize(self, event) -> dict:
        return {
            field: transformation(getattr(event, field))
            for field, transformation in self.serialization_fields.items()
        }


class Serialization:
    def __init__(self, **transformations):
        self.serializer = EventSerializer(transformations)

    def __call__(self, event_class):
        def serialize_method(event_instance):
            return self.serializer.serialize(event_instance)

        event_class.serialize = serialize_method
        return event_class


@Serialization(
    username=str.lower,
    password=hide_field,
    ip=show_original,
    timestamp=format_time,
)
@dataclass
class LoginEvent:
    username: str
    password: str
    ip: str
    timestamp: datetime
```

### `__getattribute__`

파이썬은 점(.)으로 인스턴스 혹은 클래스의 메소드에 접근할 수 있다. 이제는 너무 익숙해져서 당연스럽게 쓰고 있지만 역시 그냥되는 것은 없다.(다 누가 만들어 놔서 편하게 쓰는 것이다.) 클래스의 메소드(객체)에 접근할 수 있는 이유는 `__getattribute__`에 숨겨져있다.

```python
class Stock:
    def __getattribute__(self, item):
        print(item, "객체에 접근하셨습니다.")

s = Stock()
s.data
```

결과

```python
data 객체에 접근하셨습니다.
```

#### reference

[3.3. Special method names](https://docs.python.org/3/reference/datamodel.html#special-method-names)
