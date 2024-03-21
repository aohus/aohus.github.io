---
layout: post
title: "[Python] 파이썬 비동기의 핵심 코루틴"
subtitle:
categories: Python
tags: ["파이썬 코딩 도장"]
---
  
# UNIT41 코루틴 사용하기  
지금까지 함수를 호출한 뒤 함수가 끝나면 현재 코드로 다시 돌아왔습니다. 메인 루틴에서 서브 루틴을 호출하면 서브 루틴의 코드를 실행한 뒤 다시 메인 루틴으로 돌아오고, 서브 루틴이 끝나면 서브 루틴의 내용은 모두 사라졌습니다.   
  
코루틴은 각 루틴이 종속적인 관계가 아닌 대등한 관계로서, 특정 시점에 상대방의 코드를 실행하게 되어있는 함수입니다. 일반 함수를 호출하면 코드를 한 번만 실행할 수 있지만, **코루틴은 코드를 여러 번 실행할 수 있습니다.** 함수의 코드를 실행하는 지점을 진입점(entry point)라고 하는데, 코루틴은 진입점이 여러 개인 함수입니다.   
  
### 코루틴에 값 보내기   
  
코루틴은 제너레이터의 특별한 형태입니다. 제너레이터에서 값을 발생 시킬 때 사용했던 yield로 코루틴은 값을 받아옵니다. 코루틴의 값은 메인루틴에서 send 메서드를 이용해 보내줍니다. send를 통해 받은 값을 사용하기 위해 `yield`를 괄호로 묶어 변수에 할당해줍니다.   
- `코루틴객체.send(값)`  
- `변수 = (yield)`   
  
```python  
def number_coroutine():  
	while True :       # 코루틴을 계속 유지하기 위해 무한 루프 사용  
		x = (yield)    # 코루틴 바깥에서 값을 받아옴  
		print(x)  
  
co = number_coroutine()  
next(co)               # 코루틴 안의 yield 까지 코드 실행(최초 실행)  
  
co.send(1)     # 코루틴에 숫자 1을 보냄 -> number_coroutine은 1을 받아서 출력하고 다시 yield에서 대기  
co.send(2)     # 코루틴에 숫자 2을 보냄  
co.send(3)     # 코루틴에 숫자 3을 보냄  
```  
  
### 코루틴 바깥으로 값 전달하기   
코루틴은 바깥으로 값을 전달할 수도 있습니다. (yield 변수) 형식으로 yield에 변수를 지정한 뒤 괄호로 묶어주면 값을 받아오면서 바깥으로 값ㅇ르 전달합니다. 그리고 yield를 사용하여 바깥으로 전달한 값은 next 함수와 send 메서드의 반환값으로 나옵니다.   
  
```python  
def sum_coroutine():  
	total = 0  
	while True :  
		x = (yield total)   # 코루틴 바깥으로 값을 전달하면서 바깥에서 값을 받아옴  
		total += x  
  
co = sum_coroutine()  
print(next(co))      # 0 -> yield 0 전달하는 것까지 실행하고 대기  
  
print(co.send(1))    # 1 -> yield가 1을 받고, 다음 yield 까지 실행해서 total 값 전달  
print(co.send(2))    # 3  
print(co.send(3))    # 6  
```  
  
### 코루틴 종료하고 예외 처리하기   
보통 코루틴은 실행 상태를 유지하기 위해 while True: 를 사용하여 끝나지 않는 무한 루프로 동작합니다. 코루틴을 강제로 종료하고 싶다면 close 메서드를 사용합니다.   
- `코루틴객체.close()`  
  
**GeneratorExit 예외 처리하기**  
```python  
def number_coroutine():  
	try:  
		while True:  
		x = (yield)  
		print(x, end=' ')  
	except GeneratorExit:      # 코루틴 종료될 때 GeneratorExit 예외 발생  
		print()  
		print('코루틴 종료')  
  
co = number_coroutine()  
next(co)  
  
for i in range(20):  
	co.send(i)  
  
co.close()  
```  
  
**코루틴 안에 예외 발생시키기**  
```python  
def sum_coroutine():  
	try:  
		while True:  
		x = (yield )  
		total += x  
	except RuntimeError as e:        
		print(e)  
		yield total  
  
co = number_coroutine()  
next(co)  
  
for i in range(20):  
	co.send(i)  
  
print(co.throw(RuntimeError, '예외로 코루틴 끝내기'))  
```  
  
### 하위 코루틴 값 가져오기  
제너레이터에서 yield from 을 사용하면 값을 바깥으로 여러 번 전달한다고 했습니다. 하지만 코루틴에서는 yield from에 코루틴을 지정하면 해당 코루틴에서 return 으로 반환한 값을 가져옵니다.   
  
```python  
def accumulate():  
	total = 0  
	while True :  
		x = (yield)  
		if x is None :  
			return total  
		total += x  
  
def sum_coroutine():  
	while True :  
		total = yield from accumulate()  
		print(total)  
  
co = sum_coroutine()  
next(co)               # accumulate의 yield 까지 가서 기다림  
  
for i in range(1,11):  
	co.send(i)         # accumulate 에 숫자 보냄  
co.send(None)          # accumulate 에 None을 보내서 return total 하고 끝남  
                       # 그리고 yield from 만나서 다시 accumulate 의 yield에서 대기  
  
for i in range(1, 101):  
	co.send(i)  
co.send(None)  
```  
  
### StopIteration 예외 발생 시키기  
```python  
def accumulate():  
	total = 0  
	while True :  
		x = (yield)  
		if x is None :  
			raise StopIteration(total)  
		total += x  
  
def sum_coroutine():  
	while True :  
		total = yield from accumulate()  
		print(total)  
  
co = sum_coroutine()  
next(co)               # accumulate의 yield 까지 가서 기다림  
  
for i in range(1,11):  
	co.send(i)         # accumulate 에 숫자 보냄  
co.send(None)          # accumulate 에 None을 보내서 return total 하고 끝남  
                       # 그리고 yield from 만나서 다시 accumulate 의 yield에서 대기  
  
for i in range(1, 101):  
	co.send(i)  
co.send(None)  
```  
  
  
코루틴은 함수가 종료되지 않은 상태에서 값을 주고받을 수 있는 함수이며 이 과정에서 현재 코드의 실행을 대기하고 상대방의 코드를 실행합니다. **보통 코루틴은 시간이 오래 걸리는 작업을 분할하여 처리할 때 사용하는데 주로 파일 처리, 네트워크 처리 등에 활용**합니다.   