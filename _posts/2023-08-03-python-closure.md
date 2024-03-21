---
layout: post
title: "[Python] 파이썬 클로저"
subtitle:
categories: Python
tags: ["파이썬 코딩 도장"]
---
  
# UNIT33 클로저  
변수의 사용범위 & 함수를 클로저 형태로 만드는 방법을 알아봅시다.   
- 전역 변수(global) : 스크립트 전체에서 접근할 수 있는 변수, 전역 변수에 접근할 수 있는 범위를 전역 범위라고 한다.   
- 지역 변수(local) : 함수 내부에서 만든 변수, 변수를 만든 함수 안에서만 접근할 수 있다. 지역 변수에 접근할 수 있는 범위를 지역 범위라고 한다.   
  
### 함수 안에서 전역 변수 변경하기   
  
```python  
x = 10  
def foo():  
	x = 20  
	print(x)  
  
foo()            # 20 : foo의 지역 변수 출력  
print(x)         # 10 : 전역 변수  
```  
foo() 내부에서 변수를 만들면, 전역 변수의 변수명과 **이름만 같은 지역 변수**가 생성됩니다. 따라서 foo() 내부에서 x = 20을 할당해주어도 전역 변수 x는 바뀌지 않습니다. 전역 변수를 함수 내부에서 바꿔주고 싶으면 global 키워드를   사용해야합니다.  
  
```python  
x = 10  
def foo():  
	global x     # 전역 변수 x를 foo 내부에서 사용하겠다.   
	x = 20  
	print(x)  
  
foo()            # 20 : 전역 변수  
print(x)         # 20 : 전역 변수  
```  
  
전역 변수로 `x`를 미리 선언하지 않고 global x를 하는 경우 전역변수 x를 새로 생성합니다.   
  
### 함수 안에서 함수 만들기   
  
**지역 변수의 범위**  
```python  
def print_variable():  
    x = 1  
    def print_x():  
        print(x)  
    print_x()  
   
print_variable()   # 1 : print_x()에서 바깥쪽에 있는 x에 접근  
```  
현재 함수의 바깥쪽에 있는 지역 변수 값에 접근할 수 있습니다.   
  
**지역 변수 변경하기**  
```python  
def print_variable():  
    x = 1  
    def print_x():  
	    x = 2      # print_x 의 지역변수 x 만들어지고 x = 2 할당됨  
    print_x()  
    print(x)  
   
print_variable()   # 1 : x가 2로 바뀌지 않음  
```  
  
하지만 현재 함수의 바깥쪽에 있는 지역 변수의 값을 변경하려면 nonlocal 키워드를 사용해야합니다.   
```python  
def print_variable():  
    x = 1  
    def print_x():  
	    nonlocal x # print_variable 의 지역변수 x 사용하겠다  
	    x = 2       
    print_x()  
    print(x)  
   
print_variable()   # 2  
```  
  
### 클로저 사용하기   
이제 함수를 클로저 형태로 만드는 방법을 알아보겠습니다. 다음은 함수 바깥쪽에 있는 지역 변수 a, b를 사용하여 a * x + b를 계산하는 함수 mul_add를 만든 뒤에 함수 mul_add 자체를 반환합니다.  
  
```python  
def calc():  
    a = 3  
    b = 5  
    def mul_add(x):  
        return a * x + b    # 함수 바깥쪽에 있는 지역 변수 a, b를 사용하여 계산  
    return mul_add          # mul_add 함수를 반환  
   
c = calc()  
print(c(1), c(2), c(3), c(4), c(5)) # 8 11 14 17 20  
```  
![](https://dojang.io/pluginfile.php/13868/mod_page/content/3/033004.png)  
  
잘 보면 함수 calc가 끝났는데도 c는 calc의 지역 변수 a, b를 사용해서 계산을 하고 있습니다. 이렇게 함수를 둘러싼 환경(지역 변수, 코드 등)을 계속 유지하다가, 함수를 호출할 때 다시 꺼내서 사용하는 함수를 클로저(closure)라고 합니다.   여기서는 c에 저장된 함수가 클로저입니다.  
  
**람다로 클로저 만들기**  
```python  
def calc():  
    a = 3  
    b = 5  
    return lambda x: a * x + b    # 람다 표현식을 반환  
   
c = calc()  
print(c(1), c(2), c(3), c(4), c(5))  
```  