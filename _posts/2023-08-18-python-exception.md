---
layout: post
title: "[Python] 파이썬 예외 처리"
subtitle:
categories: Python
tags: ["파이썬 코딩 도장"]
---
  
# UNIT 38 예외 처리 사용하기  
  
```python  
def ten_div(x):  
	y = 10 / x  
	print(f"10 / {x} = {y}")  
	return  
  
ten_div(2)  
# 10 / 2 = 5.0  
  
ten_div(0) # 10 / x 에서 Error 발생하여 스크립트 실행이 중단되고 print 문 실행되지 않음  
# ZeroDivisionError: division by zero   
```  
  
예외란 코드를 실행하는 중에 발생한 에러를 뜻합니다. 예외가 발생하면 스크립트의 실행이 중단됩니다. 예외가 발생했을 때도 스크립트 실행을 중단하지 않고 계속 실행하게 해주는 예외 처리 방법에 대해 알아보겠습니다.   
  
### try-except  
```python  
def ten_div(x):  
	try :  
		y = 10 / x  
		print(f"10 / {x} = {y}")  
	except :  
		print("예외 발생")  
	print("스크립트 끝까지 실행")  
  
ten_div(0)  
# 예외 발생  
# 스크립트 끝까지 실행  
```  
  
**특정 예외만 처리하기**  
  
어떤 코드에 대해 모든 에러를 특정한 방식으로 처리해준다면 예상치 못한 에러를 놓치거나, 에러 처리 자체에 문제가 생길 수 있습니다. 0으로 나누려고 할 때 에러가 발생한다는 것을 발견하고 에러가 발생하면 x = 1 로 바꿔주고 결과를 출력하는 예외 처리를 해주었다고 합시다.   
  
```python  
def ten_div(x):  
	try :  
		y = 10 / x  
		print(f"10 / {x} = {y}")  
	except :  
		print("0으로 나누는 예외 발생")  
  
ten_div(0)  
# 0으로 나누는 예외 발생  
  
ten_div('5')  
# 0으로 나누는 예외 발생  
```  
   
 `x = 0`을 입력했을 때는 원하는 결과가 나옵니다. 하지만 실수로 `x = 5` 대신 `x = '5'`을 입력했을 때는 except 구문이 `x = 0` 일 때만 대응하고 있어 엉뚱한 결과가 나와버립니다. 이 때 발생하는 에러 이름으로 **특정 예외에만 대응하는 에러처리**를 해줄 수 있습니다.   
  
```python  
def ten_div(x):  
	try :  
		y = 10 / x  
		print(f"10 / {x} = {y}")  
	except ZeroDivisionError :  
		print("0으로 나누는 예외 발생")  
	except TypeError :  
		print("Type error 발생")  
  
ten_div(0)  
# 0으로 나누는 예외 발생  
  
ten_div('5')  
# Type error 발생  
```  
  
예외가 발생할 때 에러메시지를 그대로 보고 싶다면 아래와 같이 에러 `as e` 구문을 이용합니다.   
```python  
def ten_div(x):  
	try :  
		y = 10 / x  
		print(f"10 / {x} = {y}")  
	except ZeroDivisionError as e:  
		print("0으로 나누는 예외 발생: ", e)  
	except TypeError as e :  
		print("type error 발생: ", e)  
	except Exception as e : # Exception은 모든 에러에 대응   
		print(e)  
		raise   
```  
  
### else와 finally  
```python  
try :  
	실행할 코드  
except :  
	예외가 발생했을 때 처리하는 코드  
else :  
	예외가 발생하지 않았을 때 실행할 코드  
finally :  
	예외 발생 여부와 상관없이 항상 실행  
```  
  
### 예외 발생시키기 : raise  
지금까지 파이썬에서 정의한 예외만 처리했습니다. 우리가 만든 프로그램에서 예외를 발생시키고 싶을 땐 `raise` 를 이용합니다.   
  
```python  
try :  
	x = int(input('3의 배수를 입력하세요'))  
	if x % 3 :  
		raise Exception('3의 배수가 아닙니다.')  
	print(x)  
except Exception as e :  
	print(e)  
```  
  
### 예외 만들기(사용자 정의 예외)  
  
위에서 발생시킨 에러를 사용자 정의 예외로 만들어 발생시켜봅시다.   
  
```python  
class NotThreeMultipleError(Exception):  
	def __init__(self):  
		super.__init__('3의 배수가 아닙니다.')  
  
try :  
	x = int(input('3의 배수를 입력하세요'))  
	if x % 3 :  
		raise NotThreeMultipleError  
	print(x)  
except NotThreeMultipleError as e :  
	print(e)  
```  
  
  