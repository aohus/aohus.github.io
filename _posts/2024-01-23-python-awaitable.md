---
layout: post
title: "[Python] Awaitable 객체란?(coroutine, Task, Future)"
subtitle:
categories: Python 비동기프로그래밍
tags: []
---

`asyncio` API의 `await` 키워드 뒤에는 `Awaitable`한 객체가 와야합니다. 이때, `Awaitable`한 객체란 무엇일까요? 
[Coroutines and Tasks](https://docs.python.org/3/library/asyncio-task.html#awaitable)에서 답을 찾아봅시다. 

우리는 객체가 [`await`](https://docs.python.org/ko/3/reference/expressions.html#await) 표현식에서 사용될 수 있을 때 Awaitable 객체라고 말합니다. 
많은 asyncio API는 어웨이터블을 받아들이도록 설계되었습니다.
Awaitable 객체에는 세 가지 주요 유형이 있습니다: **코루틴**, **태스크** 및 **퓨처**.

## coroutine

파이썬 코루틴은 Awaitable이므로 다른 코루틴에서 기다릴 수 있습니다:
```python
import asyncio

async def nested():
    return 42

async def main():
    # Nothing happens if we just call "nested()".
    # A coroutine object is created but not awaited,
    # so it *won't run at all*.
    nested()

    # Let's do it differently now and await it:
    print(await nested())  # will print "42".

asyncio.run(main())
```

## Task
Task는 코루틴을 동시에 예약하는 데 사용됩니다. 
코루틴이 [`asyncio.create_task()`](https://docs.python.org/ko/3/library/asyncio-task.html#asyncio.create_task "asyncio.create_task")와 같은 함수를 사용하여 Task로 싸일 때 코루틴은 곧 실행되도록 자동으로 예약됩니다:

```python
import asyncio

async def nested():
    return 42

async def main():
    # Schedule nested() to run soon concurrently
    # with "main()".
    task = asyncio.create_task(nested())

    # "task" can now be used to cancel "nested()", or
    # can simply be awaited to wait until it is complete:
    await task

asyncio.run(main())
```

## Future
[`Future`](https://docs.python.org/ko/3/library/asyncio-future.html#asyncio.Future)는 비동기 연산의 **최종 결과**를 나타내는 특별한 **저수준** 어웨이터블 객체입니다.
`Future` 클래스는 `Task`의 상위 클래스로 루프와 관련된 모든 기능을 제공합니다. `Future`는 어떤 동작의 미래에 일어날 완료 상태를 나타내고 루프에 의해 관리됩니다. (=`Future` 객체를 기다릴 때, 그것은 코루틴이 `Future`가 다른 곳에서 해결될 때까지 기다릴 것을 뜻합니다.)
콜백 기반 코드를 `async/await`와 함께 사용하려면 asyncio의 `Future` 객체가 필요합니다.
일반적으로 응용 프로그램 수준 코드에서 `Future` 객체를 만들 필요는 없다고 합니다.

때때로 라이브러리와 일부 asyncio API에 의해 노출되는 Future 객체를 기다릴 수 있습니다:
```python
async def main():
    await function_that_returns_a_future_object()

    # this is also valid:
    await asyncio.gather(
        function_that_returns_a_future_object(),
        some_python_coroutine()
    )
```

#### Future 인스턴스의 기능
- `'result'` 값을 설정할 수 있다. (값을 쓸 때는 `.set_result(value)`를 사용하고, 값을 읽어올 때는 `.result()`를 사용)
- `.cancel()`로 작업을 취소할 수 있다. (`.cancelled()`로 취소 여부 확인)
- `Future`가 완료되었을 때 실행할 콜백 함수들을 가진다. 

`Task`를 사용하는 편이 더 일반적이지만 `Future`를 사용해야 하는 경우도 있습니다. [`loop.run_in_executor()`](https://docs.python.org/ko/3/library/asyncio-eventloop.html#asyncio.loop.run_in_executor)를 사용할 때는 `Future` 인스턴스를 반환 받습니다.


## 정리
```
             ┌──Coroutine
             │
Awaitable◄───┤
             │
             └──Future◄────────Task
```
`await` 키워드를 붙일 수 있는 최소 조건이 Awaitable입니다. 그래서 3가지 모두 `await`으로 실행이 끝나길 기다릴 수 있습니다. 하지만 코루틴은 Eventloop에 등록되지 않으면 실행되지 않기 때문에, `await`을 붙이거나 `Future`나 `Task`로 감싸야 합니다.

---
## References
<https://docs.python.org/3/library/asyncio-task.html#awaitables>