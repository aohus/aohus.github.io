---
layout: post
title: "[Python] 파이썬 클래스 사용하기"
subtitle:
categories: Python
tags: ["파이썬 코딩 도장"]
---

# UNIT34 클래스 사용하기  

클래스 : 객체를 표현하기 위한 문법  
클래스 속성 : 클래스 내의 데이터  
클래스 메서드 : 클래스 내의 함수  

### 34.3 비공개 속성 사용하기  

클래스의 비공개 속성은 클래스 안에서만 사용할 수 있는 클래스 속성입니다. `__속성` 과 같이 이름이 밑줄 두 개로 시작해야합니다.   

```python  
class Person :  
	def __init__(self, name, wallet):  
		slef.name = name  
		# __wallet : 비공개 속성  
		self.__wallet = wallet  

	def pay(self, amount):  
		if amount > self.__wallet:  
			return "잔액이 부족합니다."  
		self.__wallet -= amount  
```  

```python  
gildong = Person('홍길동', 10000)  
print(gildong.name) # value  
print(gildong.__wallet)  
# AttributeError: 'Person' object has no attribute '__wallet'  
```  

`__wallet` 은 사람의 통장 잔고라고 합시다. 외부에서 `__wallet` 에 직접 접근하여 잔고 보다 큰 금액을 빼려한다면 오류가 생길 것입니다. 비공개 속성을 이용하여 외부에서 잔고에 접근할 수 없도록 하고, 수정이 필요할 때는 클래스 내부에서   검증을 거친 후 수정하도록 하면 안전하게 관리할 수 있습니다.   

속성 뿐만 아니라 메서드도 클래스 안에서만 사용하는 비공개 메서드를 만들수 있습니다. `__메서드` 와 같이 이름이 밑줄 두 개로 시작해야합니다.   