---
layout: post
title: [파이썬] 클래스 내부 객체들
subtitle: 클래스/인스턴스 변수, 클래스/인스턴스/스태틱 메소드는 무엇인가?
categories: python
tags:
  [
    "class variable",
    "instance variable",
    "class method",
    "instance method",
    "static method",
  ]
---

### 예제 코드

```python
class Car():
	"""
	Car Class
	Author : Yoo
	Date : 2019.11.08
	"""

	# 클래스 변수
	car_count = 0
	price_per_raise = 1.0

	def __init__(self, company, details):
		# 인스턴스 변수
		self._company = company
		self._details = details
		Car.car_count += 1

	def __str__(self):
		return 'str : {} - {}'.format(self._company, self._details)

	def __repr__(self):
		return 'repr : {} - {}'.format(self._company, self._details)

	def __del__(self):
		Car.car_count -= 1

	# Instance Method
	# self : 객체의 고유한 속성 값 사용
	def detail_info(self):
		print('Current Id : {}'.format(id(self)))
		print('Car Detail Info : {} {}'.format(self._company, self._details.get('price')))

	# Class Method
	@classmethod
	def raise_price(cls, per):
		if per <= 1:
			print('Please Enter 1 or More')
			return
		cls.price_per_raise = per
		return 'Succeed! price increased.'

	# Static Method
	@staticmethod
	def is_bmw(inst):
		if inst._company == 'Bmw':
		return 'OK! This car is {}.'.format(inst._company)
	return 'Sorry. This car is not Bmw.'
```

- doc string
  doc string은 doc이라는 이름에서 알 수 있듯이 코드에 포함된 '문서'이다. 개발자간의 소통을 위해 쓰이며 코드만으로 설명이 부족한 비지니스 로직이나 사용 예시 등의 내용을 포함한다. `Car.__doc__` 로 읽을 수 있다.

- 인스턴스
  아래처럼 Class를 생성했을 때,

```python
car1 = Car('Ferrari', {'color' : 'White', 'horsepower': 400, 'price': 8000})
car2 = Car('Bmw', {'color' : 'Black', 'horsepower': 270, 'price': 5000})
car3 = Car('Audi', {'color' : 'Silver', 'horsepower': 300, 'price': 6000})
```

car1, car2, car3 의 인스턴스가 각각 생성된다. 세 인스턴스의 id는 당연 모두 다르다.
car1, car2, car3의 클래스는 동일하므로 `id(car1.__class__) == id(car3.__class__)` 이다.

### 클래스 내 변수

클래스 변수, 인스턴스변수에 대한 설명

**class variable**
클래스 변수는 클래스에서 직접 접근이 가능하고, 클래스로 선언한 모든 인스턴스에서 같은 값을 공유한다. 어떤 인스턴스 하나에서 잘못 수정/변경한다면 모든 인스턴스에 영향을 미치기 때문에 잘 사용해야한다.

```python
Car.car_count
car1.car_count
```

**instance variable**
인스턴스 변수도 클래스 변수와 동일하게 아래와 같이 접근할 수 있다. 하지만 PEP 문법적으로 권장되지 않는다. 안정성을 위해 클래스 내부 변수에 접근할 때는 메소드를 통해 접근한다. 참고로 인스턴스.variable 로 접근하면, variable을 인스턴스 네임스페이스에서 가장 먼저 검색하고 존재하지 않는다면 클래스 변수 / 부모 클래스 변수에서 검색한다.

```python
car1._company
```

### 클래스 내 메소드

Instance method, class method, static method란?
이들을 구분짓는 것은 PEP8에 정의된 Instance method, class method, static method의 정의에 따라 나누어쓴다.

**instance method**

- class를 통해 선언된 instance를 통해서만 호출이 되는 메서드
- 첫 번째 인자로 instance 자신을 자동으로 전달한다. 이 첫 인자(매개변수)는 self로 칭하며 다른 이름을 사용하는 것은 naming convention에 어긋나는 일이다.

```python
class Car():
	def detail_info(self):
		print('Current Id : {}'.format(id(self)))
		print('Car Detail Info : {} {}'.format(self._company, self._details.get('price')))

#instance 할당
car1 = Car('Ferrari', {'color' : 'White', 'horsepower': 400, 'price': 8000})
car1.detail_info()
>> {'_company': 'Ferrari', '_details': {'color': 'White', 'horsepower': 400, 'price': 8000}}
```

**class method**

- class를 통해 호출되는 메서드
- python 에선 @classmethod 데코레이터로 정의한다. 첫 번째 인자로는 클래스 자신이 전달되고 이 인자를 cls로 칭하며 다른 이름을 사용하는 것은 naming convention에 어긋나는 일이다.
- {ClassName}.{method_name}로 호출
- class 내부에서 일어나는 변화에 관여할 때 사용한다.
- class variable을 수정할 때,

```python
Car.price_per_raise = 1.2
```

로 직접 접근하는 것 보다

```python
class Car():
	...
    price_per_raise = 1.0

	@classmethod
	def raise_price(cls, per):
		if per <= 1:
			print('Please Enter 1 or More')
			return
		cls.price_per_raise = per
		return 'Succeed! price increased.'

Car.raise_price(1.2)
```

classmethod를 만들고, validation check를 한 다음 최종 요구사항을 반영하는 편이 더 안정적인 코드를 만들 수 있다.

**static method**

- 함수내에서 인자를 따로 받지 않는다. class 내부에 정의되어 있을 뿐 일반함수와 같다. 하지만 클래스와 연관성이 있는 함수를 클래스 안에 정의한 것이므로 클래스나 인스턴스를 통해 호출하여 편하게 사용가능하다.

```python
class Car():
    ...

	@staticmethod
	def is_bmw(inst):
		if inst._company == 'Bmw':
		return 'OK! This car is {}.'.format(inst._company)
	return 'Sorry. This car is not Bmw.'
```

referece
[# 파이썬 – OOP Part 4. 클래스 메소드와 스태틱 메소드 (Class Method and Static Method)](https://schoolofweb.net/blog/posts/%ed%8c%8c%ec%9d%b4%ec%8d%ac-oop-part-4-%ed%81%b4%eb%9e%98%ec%8a%a4-%eb%a9%94%ec%86%8c%eb%93%9c%ec%99%80-%ec%8a%a4%ed%83%9c%ed%8b%b1-%eb%a9%94%ec%86%8c%eb%93%9c-class-method-and-static-method/)
