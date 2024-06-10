---
layout: post
title: "[Python] 파이썬 클래스 속성과 정적, 클래스 메서드"
subtitle:
categories: Python
tags: ["파이썬 코딩 도장"]
---
  
# UNIT35 클래스 속성과 정적, 클래스 메서드  
  
### 35.1 클래스 속성과 인스턴스 속성  
  
클래스 속성은 클래스 안에 바로 할당하는 데이터입니다. 인스턴스를 통해(`self.속성` 으로) 접근해야하는 인스턴스 속성과 달리 클래스는 클래스 이름으로 바로 접근 가능합니다. 물론 클래스로 생성한 인스턴스는 클래스의 모든 속성을 가지고 있으므로 클래스 속성은 인스턴스를 통해서도 접근가능합니다. **따라서 클래스 속성은 여러 객체, 즉 클래스로 생성한 모든 인스턴스가 공유한다는 것을 유의해야합니다.** 예시를 보며 확인해봅시다.  
  
```python  
class Cls :  
	attr = []  
  
	def put_stuff(self, stuff):  
		Cls.attr.append(stuff)   
		# self.attr.append(stuff) 로도 접근 가능하지만,   
		# 클래스 속성에 접근할 때는 클래스 이름으로 접근하면 코드가 더 명확해집니다.   
  
#c1, c2는 Cls의 인스턴스!   
c1 = Cls()  
c2 = Cls()  
  
# c1 인스턴스를 통해 put_stuff 호출, put_stuff 내에서 클래스 속성 Cls.attr을 수정합니다.  
c1.put_stuff('stuff')   
  
print(Cls.attr)  # ['stuff'] - 클래스 이름으로 바로 접근  
print(c1.attr)   # ['stuff'] - c1 인스턴스를 통해 접근  
print(c2.attr)   # ['stuff']  
  
print(Cls.attr is c1.attr is c2.attr) # True  
```  
  
`c2` 인스턴스를 통해서 `attr` 에 접근하지는 않았지만, `c2.attr` 도 `c1.attr` 와 같은 결과를 얻습니다. `c1.attr is c2.attr` 로 확인해보아도 Cls의 서로 다른 객체를 통해 접근한 `attr` 이 같은 주소를 가리키고 있다는 사실을 알 수 있습니다.   
  
**속성, 메서드 이름을 찾는 순서**  
파이썬에서는 속성, 메서드 이름을 찾을 때 인스턴스, 클래스 순으로 찾습니다.   
```python  
class Cls :  
	attr = []  
	cls_attr = []  
	  
	def __init__(self):  
		self.attr = []  
		self.inst_attr = []  
  
c = Cls()  
c.attr.append(1)   
  
print(Cls.attr) # []  
print(c.attr)   # [1]  
```  
  
인스턴스나 클래스에서 `__dict__` 속성을 출력해보면 현재 인스턴스와 클래스의 속성을 딕셔너리로 확인할 수 있습니다.   
  
`c.attr` 를 찾는 순서  
```python  
c.__dict__   
# {'attr':[1], 'inst_attr':[] }  
  
Cls.__dict__   
# mappingproxy({'__module__': '__main__', 'attr': [], cls_attr:[], '__init__': <function Cls.__init__ at 0x1031a6b80>, '__dict__': <attribute '__dict__' of 'Cls' objects>, '__weakref__': <attribute '__weakref__' of 'Cls' objects>, '__doc__': None})  
```  
`c.__dict__`에 `attr`이 있는지 ?   
	y `return c.attr`  
	n `Cls.__dict__`에 `attr` 있는지 ?  
		y `return Cls.attr`  
		n `AttrubuteError`  
  
  
### 35.2 & 35.3 staticmethod 와 classmethod  
  
정적메소드는 인스턴스를 통하지 않고 클래스에서 직접 접근할 수 있는 메소드입니다. 파이썬에서는 클래스에서 직접 접근할 수 있는 메소드, staticmethod와 classmethod가 있습니다(staticmethod를 정적메소드, classmethod를 클래스메소드로 칭하기도합니다.). 하지만 파이썬에서는 다른 언어와 다르게 정적 메소드임에도 **인스턴스에서도 접근이 가능합니다.**  
  
```python  
class Calculator:   
	count = 0  
	  
	def __init__(self):   
		# count는 Calculator의 인스턴스의 수(인스턴스 생성할 때마다 count+1)  
		Calculator.count += 1   
  
	@staticmethod   
	def add(a, b):   
		return a + b  
		  
	@classmethod   
	def get_count(cls):  
		return cls.count         # cls를 통해 Calculator 전달  
  
c = Calculator()  
  
print(CalCul.add(3, 5))          # 8  
print(CalCul.get_count())        # 1 : Calculator 객체 수  
```  
  
staticmethod는 메서드의 실행이 외부 상태에 영향을 끼치지 않는 순수함수를 만들 때 사용합니다. staticmethod는 self를 받지 않으므로 인스턴스 속성에 접근할 수 없습니다.   
  
classmethod는 첫 번째 매개변수를 통해 클래스 객체를 전달받으며 변수 이름은 관례적으로 `cls`를 사용합니다.(인스턴스 메소드에서 인스터스를 `self`를 통해 전달받는 것 처럼)  클래스 메서드는 메서드 안에서 클래스 속성, 클래스 메서드에 접근해야할 때 사용합니다.  