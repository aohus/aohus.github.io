---
layout: post
title: "[Python] 파이썬 제너레이터"
subtitle:
categories: Python
tags: ["파이썬 코딩 도장"]
---

  
# UNIT40 제너레이터 사용하기  
제너레이터는 이터레이터를 생성해주는 함수입니다. 이터레이터는 클래스에 `__iter__, __next__` 또는 `__getitem__` 메서드를 구현해야 하지만 제너레이터는 함수 안에 `yield` 라는 키워드만 사용하면 끝입니다.   
  
### 제너레이터 만들기   
```python  
def number_generator():  
	yield 1  
	yield 2  
	yield 3  
  
for i in number_generator():  
	print(i)  
# 1   
# 2   
# 3  
  
g = number_generator()  
dir(g)  
```  
  
제너레이터는 제너레이터 객체에서 `__next__` 메서드를 호출할 때마다 함수 안의 `yield`까지 코드를 실행하며 `yield` 에서 값을 발생시킵니다.   
  
**제너레이터 동작**  
제너레이터 객체에서 `next`를 호출하면 제너레이터(next_generator())내부에서 yield 1이 실행되어 1을 전달한 뒤 함수 바깥의 코드가 실행되도록 양보합니다. 바깥 함수는 print(i)로 반환된 값을 출력하고 다시 제너레이터를 호출합니다. 제너레이터는 다음 yield까지 실행 후 2를 넘겨주며 다시 함수 바깥 코드가 실행되도록 양보합니다.   
  
**제너레이터와 return**  
제너레이터 함수는 끝까지 도달하면 `StopIteration` 예외가 발생합니다. 제너레이터 내부에서 `return`에 반환값을 지정하면 `StopIteration` 예외가 발생하며 반환값은 예외의 에러메시지로 들어갑니다.   
### yield from으로 값을 여러번 바깥으로 전달하기  
`yield from` 에는 반복 가능한 객체, 이터레이터, 제너레이터 객체를 지정합니다.(파이썬 3.3 부터 사용가능) 이렇게 하면 `next` 함수를 호출할 때마다 반복가능한 객체/이터레이터/제너레이터의 요소를 한 개씩 밖으로 전달합니다.   
  
```python  
def number_generator():  
	x = [1,2,3]  
	yield from x  
  
for i in number_generator():  
	print(i)  
# 1   
# 2   
# 3  
```  
  
### yield from 에 제너레이터 객체 지정하기  
  
```python  
def number_generator(stop):  
	n = 0  
	while n < stop :  
		yield n  
		n += 1  
  
def three_generator():  
	yield from number_generator(3)  
  
for i in three_generator():  
	print(i)  
```  
  