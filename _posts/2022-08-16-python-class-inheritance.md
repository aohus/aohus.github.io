---
layout: post
title: "[Python] 파이썬 클래스 상속"
subtitle:
categories: Python
tags: ["파이썬 코딩 도장"]
---
  
# UNIT 36 클래스 상속하기  
  
클래스에서 상속(inheritance)이란, 기반 클래스(Parent Class, Super class)의 내용(속성과 메소드)을 파생 클래스(Child class, sub class)에게 물려주는 것입니다. 기본적인 사용법은 아래와 같습니다.   
  
```python  
class ParentClass :  
	pass  
  
class ChildClass(ParentClass):  
	pass  
```  
  
**상속 관계 확인하기**  
클래스의 상속 관계를 확인하고 싶을 땐, `issubclass` 함수를 사용합니다.   
```python  
issubclass(ChildClass, ParentClass)   # True  
```  
  
  
### 36.3 기반 클래스의 속성 사용하기  
클래스의 상속은 기반 클래스의 능력(속성과 메서드)을 그대로 활용하면서**(기반 클래스의 능력을 활용하지 않는다면 상속을 통해 클래스를 만들지 않는 것이 좋습니다.) 기능을 추가해 새로운 클래스를 만들 때 사용합니다.   
  
파생 클래스는 super()를 통해 기반 클래스에 접근할 수 있습니다. 파생 클래스에서 `__init__` 메서드를 정의할 때 **`super.__init__()` 으로 기반 클래스를 초기화해주어 기반 클래스 속성이 만들어집니다.**  
  
```python   
class Person:  
	def __init__(self):  
		print("Person 생성자")  
	  
class Teacher(Person):  
	def __init__(self):  
		super().__init__()  
  
teacher = Teacher() # Person 생성자  
```  
  
```python   
class Person:  
	def __init__(self, name):  
		self.name = name  
	  
	def greeting(self):  
		print("Hi~")  
  
class Teacher(Person):  
	def __init__(self, name, subject):  
		super().__init__(name)         # super()로 기반 클래스의 __init__ 호출  
		self.subject = subject  
  
	def show_info(self):  
		print(f"teacher- name: {self.name}, subject: {self.subject}")  
  
Jane = Teacher('Jane', 'Math')  
Jane.greeting()                   # Hi~ : Person의 greeting 호출  
Jane.show_info()                  # teacher- name: Jane, subject: Math   
```  
  
  
파생 클래스에 `__init__` 메서드가 없다면 기반 클래스의  `__init__` 이자동으로 호출되어 기반 클래스의 속성을 사용할 수 있습니다.   
```python  
class Student(Person):  
	def show_info(self):  
		print(f"student- name: {self.name} ")  
  
Anna = Student('Anna')  
Anna.greeting()                   # Hi~ : Person의 greeting 호출  
Anna.show_info()                  # student- name: Anna  
```  
  
  
### 36.5 다중 상속 사용하기   
  
다중 상속은 여러 기반 클래스로부터 상속을 받아서 파생 클래스를 만드는 방법입니다.   
```python   
class Person:  
	def __init__(self):  
		print("Person 생성자")  
	  
	def person_method(self):  
		print("person method")  
  
class School:  
	def __init__(self):  
		print("School 생성자")  
	  
	def school_method(self):  
		print("school method")  
		  
class Teacher(Person, School):  
	def __init__(self):  
		super().__init__()  
  
teacher = Teacher()       # Person 생성자  
teacher.person_method()   # person method  
teacher.school_method()   # school method  
```  
  
Person, School을 상속받은 파생 클래스 Teacher의 인스턴스(teacher)를 통해 person_method와 school_method 모두를 호출 할 수 있었습니다. **하지만, 인스턴스를 생성할 때 `super().__init__()` 을 통해 Person의 생성자만 호출된 것을 볼 수 있습니다.** 상속받는 순서를 바꿔봅시다.   
  
```python  
class Teacher(School, Person):  
	def __init__(self):  
		super().__init__()  
  
teacher = Teacher() # School 생성자  
```  
  
이번에는 School의 생성자만 호출되었습니다. 즉, 다중 상속을 받은 클래스에서 super() 를 통해 부모 클래스로 접근을 할 때는 순서상 가장 먼저 상속받은 클래스로 접근을 하게 됩니다. 그러므로 다중 상속을 할 때 모든 부모 클래스의 생성자를 호출하려면 다음과 같이 명시적으로 각 부모 클래스의 이름을 통해서 접근해야 합니다.  
  
```python  
class Teacher(Person, School):  
	def __init__(self):  
		Person.__init__(self)  
		School.__init__(self)  
  
teacher = Teacher()   
# Person 생성자  
# School 생성자  
```  
  
### 36.6 추상 클래스 사용하기  
  
파이썬은 추살 클래스(abstract class)라는 기능을 제공합니다. **추상 클래스는 메서드의 목록만 가진 클래스이며 상속받는 클래스에서 메서드 구현을 강제하기 위해 사용합니다.**  
  
추상 클래스를 만들려면 abc 모듈을 이용해야합니다.   
  
```python  
from abc import *  
  
class PersonBase(metaclass=ABCMeta):  
	@abstractmethod  
	def method1(self):  
		pass  
	  
	@abstractmethod  
	def method2(self):  
		pass  
```  
  
추상 클래스를 상속받기 위해서는 추상 클래스에서 정의한 추상 메서드를 모두 구현해야합니다. 또한 추상 클래스는 인스턴스로 만들 수 없습니다. 추상 클래스로 인스턴스를 만들려고 하면 `TypeError` 가 발생합니다. 따라서 추상 클래서는 오직 상속에만 사용합니다.   