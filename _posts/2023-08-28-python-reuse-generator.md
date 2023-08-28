---
layout: post
title: "[파이썬] generator 재사용하는 방법(itertools.tee() 이용한 복제)"
subtitle:
categories: python
tags: ["generator", "itertools.tee"]
---

### generator

generator를 재사용하는 방법을 보기 이전에 generator에 대해 간단하게 짚고 넘어가자. generator는 iterator를 생성해주는 함수이며 함수의 내부 로컬 변수를 통해 내부상태가 유지된다.
generator는 모든 값을 순서대로 호출하고 나면 StopIteration Error를 반환하며 더 이상 값을 저장하지 않는다.

앞으로의 예제를 위해 간단한 generator 객체를 생성하여 사용해보자.

```shell
>>> def test_generator():
...     yield 1
...     yield 2
...     yield 3
...
>>> gen = test_generator()
>>> next(gen)
1
>>> next(gen)
2
>>> next(gen)
3
>>> next(gen)
StopIteration
```

generator를 재사용하기 위해서는 제너레이터 객체를 리스트로 만들어 재사용하거나

```shell
>>> gen = test_generator()
>>> gen_list = list(gen)
>>> for x in gen_list: print(x)
1
2
3
>>> for x in gen_list: print(x)
1
2
3
```

itertools.tee() 함수로 복사하여 재사용하는 방법이 있다.

```shell
>>> from itertools import tee

>>> gen = test_generator()
>>> gen_copy1, gen_copy2, gen_copy3  = tee(gen, 3)
>>> for x in gen_copy1: print(x)
1
2
3
>>> for x in gen_copy2: print(x)
1
2
3
```

오늘은 itertools.tee() 함수로 generator 를 재사용하는 방법을 알아보려고한다.

### itertools.tee(iterable, n=2)

itertools.tee 함수는 iterable 객체를 받아 n개의 독립적인 iterator 객체들을 반환한다.

간단히 사용해보자

```shell
>>> from itertools import tee

>>> gen = test_generator()
>>> gen_copy1, gen_copy2, gen_copy3  = tee(gen, 3)
>>> list(gen_copy1)
[1, 2, 3]
>>> list(gen_copy2)
[1, 2, 3]
```

이때, 유의할 점은 tee() 를 통해 복제본을 생성한 후에는 원본 iterable 객체는 사용하면 안된다는 것이다.

[itertools 문서](https://docs.python.org/3/library/itertools.html#itertools.tee)에서는 tee 함수의 동작에 대한 이해를 돕기위해 아래와 같은 코드를 제공한다.

```python
def tee(iterable, n=2):
    it = iter(iterable)
    deques = [collections.deque() for i in range(n)]
    def gen(mydeque):
        while True:
            if not mydeque:             # when the local deque is empty
                try:
                    newval = next(it)   # fetch a new value and
                except StopIteration:
                    return
                for d in deques:        # load it to all the deques
                    d.append(newval)
            yield mydeque.popleft()
    return tuple(gen(d) for d in deques)
```

```shell
>>> from itertools import tee

>>> gen = test_generator()
>>> gen_copy1, gen_copy2, gen_copy3  = tee(gen, 3)
>>> gen_copy1
<generator object tee.<locals>.gen at 0x10ffa97b0>
```

값을 호출하기 전의 복제본들은 아직 gen 함수가 실행되지 않은 상태의 객체이다. 원복 객체의 값은 gen 함수 내부에서 소비되는 동시에 복제본으로 복제된다. 따라서 gen 함수 실행 전, 즉 복제본의 값을 호출하기 전에 원본 객체의 값을 호출한다면, 원본 generator 객체의 포인터는 이동한 상태가 된다. 이 상태에서 복제본의 값을 호출하면 그 때 gen함수가 실행되며 원본 객체의 포인터가 이동된 부분부터 값을 복제하게된다.

```shell
>>> from itertools import tee
>>> gen = test_generator()
>>> gen_copy1, gen_copy2, gen_copy3  = tee(gen, 3)
>>> next(gen) # gen의 포인터는 다음 값인 2으로 이동하여 대기
1

>>> list(gen_copy1) # 복제된 값이 호출될 때, gen 함수가 실행되고 gen 의 2부터 소비하며 값을 복제
[2, 3]

>> next(gen) # gen 내부에서 원본 객체 e는 모든 값을 소비한 상태가 된다.
StopIteration

>>> list(gen_copy2) # gen_copy1 호출될 때, gen 이 실행되었기 때문에 나머지 복제본들도 독립적인 상태로 전환됨.
[2, 3]
```

정리하자면 tee로 복제한 객체들은 **값이 호출될 때** 원본 객체 e에 대한 복제가 이루어지며 독립적인 상태가 되고 원본 객체 gen은 모든 값을 소비한 상태가 된다.

또한 itertool에는 상당한 보조 저장소가 필요할 수 있다(저장해야 하는 임시 데이터의 양에 따라 다름). 일반적으로 한 반복자가 다른 반복자가 시작되기 전에 데이터의 대부분 또는 전부를 사용하는 경우 tee() 대신 list()를 사용하는 것이 더 빠르다고한다.

### itertools.islice()

tee()는 thread-safe하지 않다. thread-safe한 대안으로는 itertools.islice() 함수가 있다. islice() 함수는 한 번에 하나씩 값을 반환하는 이터레이터를 생성하는 동안 사용 가능한 값을 제공한다. 이 함수는 복사본 대신에 원본 이터레이터를 사용하여 값을 생성하므로, thread-safe하게 동작한다.
