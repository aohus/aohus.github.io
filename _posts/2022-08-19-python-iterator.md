---
layout: post
title: "[Python] 파이썬 이터레이터"
subtitle:
categories: Python
tags: ["파이썬 코딩 도장"]
---
  
# UNIT 39 이터레이터 사용하기   
  
이터레이터는 값을 차례대로 꺼낼 수 있는 객체입니다. 이터레이터를 보기 전에 우리에게 익숙한 반복 가능한 객체를 먼저 살펴봅시다.   
  
### 반복 가능한 객체(iterable)  
반복 가능한 객체는 요소가 여러 개 들어있고, 한 번에 하나씩 꺼낼 수 있는 객체입니다. 객체가 반복 가능한 객체인지는 객체에 `__iter__` 메서드가 들어있는지 확인하면 됩니다.   
  
반복 가능한 객체에서 `__iter__`를 호출하면 이터레이터가 나옵니다. 이터레이터를 변수에 저장한 뒤 `__next__` 메서드를 호출해보면 요소를 차례대로 꺼낼 수 있습니다. 설명에서 알 수 있듯이 반복 가능한 객체(iterable)와 이터레이터(iterator)는 별개의 객체입니다.   
  
```python  
print(dir([1,2,3]))  
# ['__add__',  ...    '__iter__'   ..., 'append', 'clear']  
  
it = [1,2,3].__iter__()  
  
print(it.__next__()) # 1  
print(it.__next__()) # 2  
print(it.__next__()) # 3  
print(it.__next__()) # StopIteration  
```  
  
 **for 와 반복 가능한 객체**  
```python  
for i in [1,2,3] :  
	print(i)  
```  
  
for는 다음과 같이 동작합니다.   
1) 반복 가능한 객체에서 `__iter__()`로 이터레이터를 얻는다.  
2) for를 통해 반복할 때마다 이터레이터에서 `__next__()`로 요소를 꺼내서 i 에 저장한다.   
3) `__next__()` 에서 `StopIteration`을 만나면 반복을 끝낸다.   
  
cf) 반복 가능한 객체는 그럼 시퀀스 객체일까요?  
아닙니다. 시퀀스 객체는 요소의 순서가 정해져 있고 연속적으로 이어져있어야합니다. 하지만 딕셔너리와 세트는 요소의 순서가 정해져 있지 않습니다. 따라서 시퀀스 객체가 반복 가능한 객체보다 좁은 개념입니다.   
  
![[Pasted image 20230901094455.png]]  
  
### 이터레이터  
이터레이터는 반복 가능하며, `__next__` 메서드를 사용해 값을 차례로 꺼낼 수 있는 객체입니다. 클래스에 `__iter__`, `__next__` 메서드를 구현하여 이터레이터를 만들 수 있습니다.   
  
```python  
class 이터레이터이름 :  
	def __iter__(self):  
		pass  
	def __next__(self):  
		pass  
```  
  
**인덱스로 접근할 수 있는 이터레이터**  
`__getitem__` 메서드를 구현하면 인덱스로 접근하는 이터레이터를 만들 수 있습니다.   
  
```python  
class 이터레이터이름 :  
	def __getitem__(self, index):  
		return index  
```  
  
### iter, next 함수  
파이썬에는 `iter`, `next` 라는 내장함수가 있습니다. `iter(object)` 는 object의 `__iter__`을 호출해주고 (`object.__iter__()`), `next(object)` 는 object의 `__next__`를 호출해줍니다.(`object.__next__()`)  
  
- iter(호출가능한 객체, 반복을 끝낼 값)  
- next(반복가능한 객체 , 반복을 끝낼 값)  
  
  