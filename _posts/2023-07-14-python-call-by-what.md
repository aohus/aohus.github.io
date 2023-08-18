---
layout: post
title: "[파이썬] 가변형, 불변형 객체에 따른 동작 방식"
subtitle:
categories: python
tags: ["call by object reference"]
---

### Call by value ? Call by reference ?

파이썬의 Call by object reference에 대해 알아보자. 파이썬 Tutorial 에는 파이썬은 Call by object reference하게 동작한다고 소개한다. Call by object reference 는 무엇일까?
먼저 Call by value, Call by reference에 대해 간단히 알아보자. `Call by value`는 인자로 받은 값을 복사하여 처리하고 `Call by reference`는 인자로 받은 값의 주소를 참조하여 값에 영양을 주는 것으로 처리하는 방식을 말한다.
값을 복사해서 처리하는지, 직접 참조를 하는지의 차이이다.

**Call by value 처럼 보이는 예시**

```python
a=1
b=a
b+=1

print(a is b)
print(f"a={a}, b={b}")
# >>> false
# >>> a=1, b=2
```

위 예시에서는 python이 a의 값을 b에 복사(call by value)하고 b+=1을 처리한 것처럼 보인다.
실제로 int는 immutable 객체여서 b에 1을 더하는 순간 다른 메모리 주소를 할당하고 거기로 b의 포인터를 옮긴다. 따라서 둘의 id는 다르며 값도 다르게 출력된다.

**Call by reference 처럼 보이는 예시**

```python
a=["apple"]
b=a
b+=["banana"]

print(a is b)
print(f"a={a}, b={b}")
# >>> true
# >>> a=["apple", "banana"], b=["apple", "banana"]
```

위 예시에서는 python이 a의 주소값을 b가 참조하고, b+="banana"를 처리하여 a와 b 모두 바뀐 것으로 보인다.
실제로 list 는 mutable 객체여서 b에 ["banana"]를 더하면 메모리 주소는 그대로인채로 value만 바뀐다.

### Call by object reference

같은 언어인데 값을 할당하고 처리하는 방식에 왜 차이가 생기는 것처럼 보일까?
Python에는 immutable한 변수형이 있고, mutable한 변수형이 있다. 변수는 사실 그들의 자료형에 대한 메모리 주소(reference)를 가지고 있다. Python은 함수를 실행할 때 실제로는 메모리 주소를 넘겨주지만 **immutable한 변수형은 내용의 변경이 불가능하기 때문에 새 값을 만들고 변수의 메모리 주소가 다른 값으로 변경이 되고, mutable한 변수형은 변경 가능하기 때문에 변수 메모리 주소는 그대로 둔 채로 해당 메모리 주소에 저장된 값이 변경되는 것이다.**

```python
a=1
```

파이썬3 인터프리터는 아래와 같이 자료형과 값을 저장하는 형태로 작동한다.

1. 1을 a에 할당
2. a -> PyObject_HEAD -> typecode 를 int로 지정
3. a -> val = 1

### 참조가 되지 않도록 값 자체를 복사하려면?

**list**

```python
a = [1,2,3]
b = a       # a의 메모리 주소 참조
c = a[:]    # 값만 복사, 메모리 주소는 따로 할당됨
```

a[:]로 처리한 변수 c는 다른 ID를 갖는다. 이 외에도 좀 더 직관적으로 처리하는 방법은 다음과 같이 명시적으로 copy() 메소드를 사용하는 방법이다.

```python
d = a.copy()
```

a처럼 단순한 list는 [:], .copy()로도 충분하지만, 복잡한 중첩 리스트의 경우 copy.deepcopy()를 이용해야한다.

```python
import copy
a=[1,2,[3,5],4]
b = copy.deepcopy(a)
```

**reference**
[모호한 파이썬 튜토리얼, Call by what?](https://item4.blog/2015-07-18/Some-Ambiguousness-in-Python-Tutorial-Call-by-What/)
