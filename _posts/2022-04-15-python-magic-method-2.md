---
layout: post
title: "[Python] 매직 메소드(2)"
subtitle:
categories: Python
tags: []
---

클래스 안에 정의된 함수를 '메소드'라고 부른다. 메소드 중에서 `__`로 시작해서 `__`로 끝나는 메소드들 이 있는데, 이를 매직 메소드 또는 특별 메소드라고 부른다.
파이썬 클린코드를 읽다가 만난 매직 메소드 `__call__`, `__getattribute__`를 [이 전 포스팅](https://aohus.github.io/python/2022/04/13/python-magic-method-1.html)에서 소개했는데 이외에도 엄청 많은 매직메소드들이 있다.
자주 쓰이고 궁금했던 것 위주로 정리해두려고 하고, 잘 모르는 매직메소드를 만날 때 마다 여기에 추가해가려고한다! ^0^

### `__new__`

새로운 인스턴스를 만들 때 제일 처음으로 실행되는 메소드이며 새로운 object를 return한다.

### `__init__`

인스턴스가 생성된 후 `__new__()`에 의해 호출되고 호출자에게 반환되기 전에 호출된다. 인수는 클래스 생성자 표현식에 전달된 인수이다.

기본 클래스에 `__init__()` 메서드가 있는 경우 파생 클래스의 `__init__()` 메서드(있는 경우)는 이를 명시적으로 호출하여 인스턴스의 기본 클래스 부분이 적절하게 초기화되도록 해야 한다. 즉, 부모 클래스를 상속받고 부모 클래스의 `__init__()` 매직메소드를 자식 클래스에서 실행한다. 따라서 자식 클래스에도 부모 클래스의 인스턴스 속성과 동일한 속성을 생성해준다.

```python
class Human:
	def __init__(self):
		self.name = '홍길동'
		self.age = 20
		self.city = '서울'

class Student(Human):
	def __init__(self, name):
		super().__init__() # 부모 클래스(Human)의 인스턴스 속성과 동일한 속성 생성
		self.name = name

	def show_name(self):
		print("Student 이름: ", self.name)

	def show_age(self):
		print('Student 나이: ', self.age)

# 출력
S = Student('Peter')
S.show_name()
S.show_age() #부모 클래스의
```

결과

```python
Student 이름 : Peter
Student 나이 : 19
```

### `__del__`

인스턴스가 사라지기 직전(about to be destroyed) 에 호출되는 메소드이다. finalizer 혹은 destructor라고 불리기도한다.

> **참고** > `del x`는 `x.__del__()` 을 직접 호출하지 않습니다. 전자는 `x`에 대한 참조 카운트를 1씩 감소시키고 후자는 `x`의 참조 카운트가 0에 도달할 때만 호출됩니다.

### `__str__`

### `__repr__`

### `__add__`

사용하기 쉽게 +에 `__add__`에 매핑되었다고 생각하면 된다. 아래 두 코드는 같은 일을 한다.

```python
print(n+10)
print(n.__add__(10))
```

### `__getitem__`

시퀀스 객체에서 \[\](대괄호)를 사용하면 실제로는 `__getitem__` 메서드를 호출하여 요소를 가져온다. 따라서 `__getitem__`을 직접 호출하여 요소를 가져와도 동일한 결과가 나온다.

```python
a=[10, 20, 30, 40]

a[1] # 20
a.__getitem__(1) # 20
```

### `__slots__`
인스턴스는 자유롭게 속성을 추가할 수 있지만 특정 속성만 허용하고 다른 속성은 제한하고 싶을 수도 있다. 이때 `__slots__`에 허용할 속성 이름을 리스트로 넣어주면 된다. 

```python
class Person:
	__slots__ = ['name', 'age']

maria = Person()
maria.name = '마리아'
maria.age = 20 
maria.address = '서울시 송파구' # AttributeError: 'Person' object has no attribute 'address'
```

#### reference

[3.3. Special method names](https://docs.python.org/3/reference/datamodel.html#special-method-names)
