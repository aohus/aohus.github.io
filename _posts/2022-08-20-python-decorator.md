---
layout: post
title: "[Python] 파이썬 데코레이터"
subtitle:
categories: Python
tags: ["파이썬 코딩 도장"]
---
  
# UNIT42 데코레이터 사용하기  
  
### 데코레이터 만들기  
데코레이터는 함수 안에서 함수를 만들고 반환하는 클로저입니다. 어떤 함수가 있을 때 해당 함수를 직접 수정하지 않고 함수에 기능을 추가하고자 할 때 데코레이터를 사용합니다.  
```python  
def trace(func):  
	def wrapper():  
		print(func.__name__, '함수 시작')  
		func()  
		print(func.__name__, '함수 끝')  
	return wrapper  
  
def hello():  
	print('hello')  
  
trace_hello = trace(hello) # wrapper 함수  
trace_hello() # wrapper -> print, func, print 실행  
  
def world():  
	print('world')  
	  
trace_world = trace(world)  
trace_word()  
```  
  
  
데코레이터는 @를 사용하여 간편하게 사용할 수 있습니다.   
```python  
@데코레이터  
def 함수이름():  
	코드  
```  
  
위에서 만든 데코레이터, trace 함수를 hello 함수에 동일하게 적용해봅시다.   
```python  
@trace  
def hello():  
	print('hello')  
  
hello()  
```  
  
  
### 매개변수와 반환값을 처리하는 데코레이터  
```python  
def trace(func):  
	def wrapper(a,b):  
		func(a,b)  
		print(f"{func.__name__},(a={a}, b={b}) -> {r})  
		return r  
	return wrapper  
  
@trace  
def add(a,b):  
	return a+b  
```  
  
클래스를 만들면서 메서드에 데코레이터를 사용할 때는 self를 주의해야합니다. 인스턴스 메서드는 항상 self를 받으므로 데코레이터를 만들 때도 wrapper함수의 첫 번째 매개변수는 self로 지정해야합니다. 마찬가지로  func를 호출할 때도  self와 매개 변수를 그대로 넣어야합니다.   
  
### 가변 인수 함수 데코레이터  
```python  
def trace(func):  
	def wrapper(*args, **kwargs):  
		r = func(*args, **kwargs)  
		print(f"{func.__name__},(args={args}, kwargs={kwargs}) -> {r})  
		return r  
	return wrapper  
  
@trace  
def get_max(*args):  
	return max(args)  
  
@trace  
def get_min(**kargs):  
	return min(kwargs.values())  
	  
get_max(10, 20)  
get_min('x'=10, 'y'=20)  
```  
  
### 매개변수가 있는 데코레이터 만들기  
```python  
def is_multiple(x):              # 데코레이터가 사용할 매개변수를 지정  
    def real_decorator(func):    # 호출할 함수를 매개변수로 받음  
        def wrapper(a, b):       # 호출할 함수의 매개변수와 똑같이 지정  
            r = func(a, b)       # func를 호출하고 반환값을 변수에 저장  
            if r % x == 0:       # func의 반환값이 x의 배수인지 확인  
                print('{0}의 반환값은 {1}의 배수입니다.'.format(func.__name__, x))  
            else:  
                print('{0}의 반환값은 {1}의 배수가 아닙니다.'.format(func.__name__, x))  
            return r             # func의 반환값을 반환  
        return wrapper           # wrapper 함수 반환  
    return real_decorator        # real_decorator 함수 반환  
   
@is_multiple(3)     # @데코레이터(인수)  
def add(a, b):  
    return a + b  
   
print(add(10, 20))  
print(add(2, 5))  
```  
  
매개변수가 있는 데코레이터를 여러 개 지정할 수 있습니다.   
```python  
@데코레이터1(인수)  
@데코레이터2(인수)  
def 함수이름(인수):  
	코드  
	  
함수이름(인수)  
  
def 함수이름(인수):  
	코드  
데코레이터1(인수)(데코레이터2(인수)함수이름)(인수)  
```  
  
### 클래스로 데코레이터 만들기  
클래스로 데코레이터를 만들때는 인스턴스를 함수처럼 호추라게 해주는  `__call__` 메서드를 구현해야합니다. 다음은 함수의 시작과 끝을 출력하는 데코레이터입니다.   
  
```python  
class Trace:  
	def __init__(self, func):  
		self.func = func  
  
	def __call__(self):  
		print(self.func.__name__, '함수 시작')  
		self.func()  
		print(self.func.__name__, '함수 끝')  
  
@Trace  
def hello():  
	print('hello')  
  
hello()  
  
# 데코레이터 @으로 지정하지 않고 호출하는 방법  
def hello():  
	print('hello')  
  
trace_hello = Trace(hello)  
trace_hello() #   
```  
  
### 클래스로 매개변수와 반환값을 처리하는 데코레이터 만들기  
```python  
class Trace:  
    def __init__(self, func):    # 호출할 함수를 인스턴스의 초깃값으로 받음  
        self.func = func         # 호출할 함수를 속성 func에 저장  
   
    def __call__(self, *args, **kwargs):    # 호출할 함수의 매개변수를 처리  
        r = self.func(*args, **kwargs) # self.func에 매개변수를 넣어서 호출하고 반환값을 변수에 저장  
        print('{0}(args={1}, kwargs={2}) -> {3}'.format(self.func.__name__, args, kwargs, r))  
                                            # 매개변수와 반환값 출력  
        return r                            # self.func의 반환값을 반환  
   
@Trace    # @데코레이터  
def add(a, b):  
    return a + b  
   
print(add(10, 20))  
print(add(a=10, b=20))  
```  
  
  
### 클래스로 매개변수가 있는 데코레이터 만들기  
```python  
class IsMultiple:  
    def __init__(self, x):         # 데코레이터가 사용할 매개변수를 초깃값으로 받음  
        self.x = x                 # 매개변수를 속성 x에 저장  
   
    def __call__(self, func):      # 호출할 함수를 매개변수로 받음  
        def wrapper(a, b):         # 호출할 함수의 매개변수와 똑같이 지정(가변 인수로 작성해도 됨)  
            r = func(a, b)         # func를 호출하고 반환값을 변수에 저장  
            if r % self.x == 0:    # func의 반환값이 self.x의 배수인지 확인  
                print('{0}의 반환값은 {1}의 배수입니다.'.format(func.__name__, self.x))  
            else:  
                print('{0}의 반환값은 {1}의 배수가 아닙니다.'.format(func.__name__, self.x))  
            return r               # func의 반환값을 반환  
        return wrapper             # wrapper 함수 반환  
   
@IsMultiple(3)    # 데코레이터(인수)  
def add(a, b):  
    return a + b  
   
print(add(10, 20))  
print(add(2, 5))  
```  