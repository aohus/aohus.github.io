---
layout: post
title: "[Python] 파이썬 비동기 라이브러리 Asyncio"
subtitle:
categories: Python, 비동기 프로그래밍
tags: []
---

## Asyncio 소개
### Asyncio로 해결할 수 있는 것은?
Asyncio의 목표는 대기를 필요로 하는 여러 개의 작업을 동시에 잘 수행하는 것입니다. 즉, 이 작업이 완료되길 기다리는 동안 다른 작업을 수행할 수 있도록 하는 것입니다. 

I/O 위주 작업에 스레드 기반 병행 처리보다 비동기 기반 병행 처리를 적용해야하는 두 가지 이유가 있습니다. 
- Asyncio는 스레드를 사용하는 선점형 멀티태스킹보다 안전한 대안이 될 수 있습니다. 단순하지 않은 스레드 기반 애플리케이션에서 때때로 발생하는 오류, 경합 조건, 혹은 비결정론적 위험 요소가 발생하지 않습니다. 
- Asyncio를 통해 동시에 수천개의 소켓 연결을 간단히 처리할 수 있습니다. 또한 웹소켓이나 사물인터넷을 위한 MQTT 같은 신기술에서 지원하는 수명이 긴 연결도 처리할 수 있습니다. 

프로그래밍 모델 관점에서 보면 스레딩의 여러 CPU와 공유 메모리(스레드 간 효율적 통신의 수단)를 사용하는 방식이 계산 위주 작업을 가장 잘 수행할 수 있어 계산 위주 작업이 많은 분야에 가장 적합합니다. 하지만 다른 문제들을 발생시킬 수 있어 필요악입니다. 
네트워크 프로그래밍은 스레딩을 필요로하는 영역은 아닙니다. 네트워크 프로그래밍의 중요한 특징은 '어떤 일들이 일어나기를 기다림' 이라는 많은 작업들로 구성되어 있다는 점입니다. 따라서 여러 CPU에 작업들을 효율적으로 분배하기 위한 운영체제와의 연계 작업이 필요없습니다. 

## 스레드에 관한 진실
스레드는 OS에서 제공되는 기능으로 소프트웨어 개발자가 OS에 프로그램 일부를 병렬로 실행하겠다고 알리는 기능입니다. OS는 해당 프로그램의 일부에 CPU 자원을 얼마나 할당할지를 결정하고, 이때 OS에서 실행 중인 다른 프로그램들에 할당한 CPU 자원도 고려하여 결정합니다. 

Asyncio를 설명하며 스레드 내용을 다루지 않아도 되지만, 
1) Asyncio가 스레딩의 대체제로 제안되었다는 점
2) Asyncio를 익히더라도 여전히 스레드와 프로세스를 써야할 가능성이 크다는 점
에 의해 스레딩에 대해 알아보도록 하겠습니다. 

### 스레딩의 장점
- 읽기 쉬운 코드: 동시에 실행할 코드를, 단순한 하향식 코드로 작성할 수 있습니다. 
- 공유 메모리를 통한 병렬 처리: 스레드간 공유 메모리를 통해 통신하면서 코드에서 여러 CPU를 이용할 수 있습니다. 공유 메모리를 사용하지 않고 프로세스별 메모리 영역 복제를 통해 여러 프롯ㅔ스 간에 대량의 데이터를 상호 전달할 경우 상당히 많은 CPU 및 메모리 전송 자원을 필요로합니다. 
- 노하우 및 기존 코드 활용 가능

### 스레딩의 단점
- 어려움: 스레드 프로그램에서 발생하는 스레드 관련 오류나 경합 조건은 가장 고치기 어렵습니다. 
- 자원이 소모적임: 스레드는 더 많은 OS 자원을 사용합니다. 사전 할당되는 스레드별 스택 공간의 경우 프로세스의 가상 메모리 공간을 선점적으로 소모합니다.(1개 스레드에 8메가바이트의 스택 메모리가 필요합니다.)
- 처리량에 영향을 줄 수 있음: 매우 높은 병행 수준(스레드 5000개 이상)에서는 콘텍스트 전환 비용으로 인해 처리량(throughput)에 영향이 있을 수 있습니다. 
- 유연하지 않음: OS는 어떤 스레드가 작업을 수행할 준비가 되었는지와는 관계없이 모든 스레드가 CPU 시간을 지속적으로 공유합니다. 어떤 스레드는 소켓으로 데이터가 도달하길 기다리고 있을 수 있습니다. 하지만 OS 스케쥴러는 데이터가 도달하여 스레드의 실행 재개가 필요하기 전까지 수천 번에 걸쳐 콘텍스트 전환을 의미없이 수행할 것입니다. 
	- 비동기 방식의 경우 `select()` 시스템 함수를 호출하여 소켓에 대해 대기 중인 코루틴의 실행 재개가 필요한지 확인할 수 있고, 필요하지 않은 경우 코루틴의 실행을 재개하지 않으므로, 콘텍스트 전환 비용을 완전히 절감할 수 있습니다. 


## Asyncio 공략
> Asyncio는 파이썬의 병행(concurrent) 프로그래밍 도구로 스레드나 멀티 프로세싱 대비 가벼운 편입니다. 이벤트 루프를 통해 일련의 태스크를 실행하는 방식입니다. 다른 방식들과 가장 큰 차이점은 각 태스크에서 이벤트 루프로 제어권을 다시 넘겨줄 시점을 지정한다는 것입니다.(필립 존스, Understanding Asyncio)

### Asyncio의 일곱 가지 기능
'PEP 492'의 저자이자 비동기 파이썬에 대한 주요 기여자인 유리 셀리바노프는 `asyncio` 모듈의 많은 API는 프레임워크 개발자를 대상으로 설계했다고 설명했다. 또한 최종 사용자 개발자들이 사용해야하는 주요 기능을 별도로 강조하여 설명했다. 전체 asyncio API 중 일부는 아래와 같이 요약할 수 있습니다. 
- asyncio 이벤트 루프 시작하기
- async/await 함수 호출하기
- 루프에서 실행할 태스크 작성하기 
- 여러 개의 태스크가 완료되길 기다리기
- 모든 병행 태스크 종료 후 루프 종료하기

### Asyncio의 탑
계층과 계층별 이름은 `asyncio` 저자의 주관적 해석이며, `asyncio` API에 대해 설명하기 위한 목적입니다. 

| 단계 | 구성 | 구현 |
| ---- | ---- | ---- |
| 9 | 네트워크: 스트림 | StreamReader, StreamWriter, asyncio.open_connection(), asyncio.start_server() |
| 8 | 네트워크: TCP&UDP | Protocol |
| 7 | 네트워크: 트랜스포트 | BaseTransport |
| 6 | 도구 | asyncio.Queue |
| 5 | 별개의 스레드와 프로세스 | run_in_executor(), asyncio.subprocess |
| 4 | Task | asyncio.Task, asyncio.create_task() |
| 3 | Future | asyncio.Future |
| 2 | 이벤트 루프 | ayncio.run(), BaseEventLoop |
| 1 | 코루틴 | async def, async with, async for, await |

#### 계층1: 코루틴
가장 기초적인 단계입니다.

#### 계층2: 이벤트 루프
코루틴은 그 자체만으로는 유용하지 않습니다. 코루틴을 실행할 루프가 있어야합니다. `asyncio` 라이브러리에서는 이벤트 루프를 위한 사양인 `AbstractEventLoop`와 구현인 `BaseEventLoop`를 제공합니다. 
사양과 구현이 명확히 구분되어있어 서드파티 개발자들이 이벤트 루프에 대한 대체제를 개발할 수 있습니다. FastAPI에서 사용하는 라이브러리인 `uvloop` 는 `asyncio` 라이브러리의 계층 구조 중 루프 관련된 부분만 플러그인 되어 단순 대체하고 있습니다.(`asyncio` 표준 라이브러리에서 제공하는 이벤트 루프에 비해 훨씬 빠릅니다.)

> **cf. uvloop 이 빠른 이유**
> - **libuv 기반**: uvloop의 이벤트 루프는 libuv의 이벤트 루프를 사용하여 구현되었습니다. libuv는 Node.js에서 사용되며, 비동기 I/O, 이벤트 핸들링, 타이밍, 스트림, 파일 시스템 작업 등을 위한 크로스 플랫폼 비동기 I/O 라이브러리입니다. libuv는 이러한 작업을 효율적으로 처리하기 위해 운영체제의 비동기 인터페이스를 사용합니다. 
> - **운영체제의 비동기 I/O 활용**: uvloop는 운영체제 수준의 비동기 I/O 기능(예: epoll, kqueue, IOCP)을 활용하여 더 높은 I/O 처리량을 달성합니다. 이 기능들은 수천 개의 네트워크 연결을 더 효율적으로 관리할 수 있게 합니다.
> - **Cython 최적화**: uvloop는 Cython으로 작성되어 있습니다.

#### 계층3~4: Future, Task
`Task`는 `Future`의 하위 클래스입니다. `Future` 인스턴스는 이벤트 루프에서 실행 중인 태스크로 알림을 통해 결과를 반환합니다. `Task` 인스턴스는 이벤트 루프에서 실행 중인 코루틴입니다. 

#### 계층5: 별개의 스레드와 프로세스
별개의 스레드 혹은 별개의 프로세스에서 작동해야하는 작업을 시작하고 대기하는 기능들이 있습니다. 
- 비동기 애플리케이션에서 블로킹 코드를 사용할 때 익스큐터를 활용

#### 계층6: 도구
`asyncio.Queue`와 같은 추가적인 비동기 기반 도구들이 있습니다. `queue` 스레드에 안전한  `Queue` 와 매우 흡사합니다. 차이점은 `asyncio`의 `Queue`를 사용할 때는 `get()`, `put()`에 `await` 키워드를 사용해야한다는 것입니다. 
- 한 개 이상의 긴 시간 동안 실행하는 코루틴에 데이터를 전달해야한다면 `asyncio.Queue`가 가장 적합한 방법입니다. 

#### 계층 7~9: 네트워크
네트워크 I/O 계층이 분포되어 있습니다. 최종 사용자 개발자 관점에서 가장 유용한 API는 9계층의 `Streams API`입니다. 

### 코루틴
아래는 `asyncio` 라이브러리에서 코루틴을 어떻게 다루는지를 보기 위한 예제와 설명들입니다.  `Python 3.5` 이전의 코루틴은 `generator`를 기반으로 구현했다. `async`, `await`을 사용하는 네이티브 코루틴은 `Python 3.5`부터 지원되었습니다. 

> 파이썬의 코루틴에 대해 좀 더 깊이 이해하기 원한다면 `generator`의 개념과 작동 방식을 공부하길 추천합니다. `generator`롤 틍해 비동기 프로그래밍의 역사적 맥락과 그 진화 과정을 더 잘 이해할 수 있고 코루틴의 핵심 메커니즘인 실행의 일시 중지 및 재개, 값의 생성과 소비 등을 이해할 수 있습니다. 
   - [python 코루틴(coroutine) - iterator, generator, asyncio, async, await 그리고 코루틴 (2)](https://velog.io/@qlgks1/python-%EC%BD%94%EB%A3%A8%ED%8B%B4coroutine-iterator-generator-asyncio-async-await-%EA%B7%B8%EB%A6%AC%EA%B3%A0-%EC%BD%94%EB%A3%A8%ED%8B%B4-2)

#### `async def`: 코루틴 선언

**[간단한 형태의 코루틴 선언]**
``` python
async def f():
	return 123

coro = f()

type(f) # <class 'function'> 
type(coro) # <class 'coroutine'> 
```
- f는 비동기 "함수"일 뿐 코루틴이 아닙니다.(코루틴 함수라고도 합니다.)
- 비동기 함수를 호출했을 때 반환하는 객체가 "코루틴 객체"입니다.

코루틴은 완료되지 않은 채 일시 정지 했던 함수를 재개할 수 있는 기능을 가진 객체입니다. 파이썬에서 코루틴 객체들이 어떻게 사용되는지 확인하고 코루틴들 사이에서 실행을 '전환'하는 방식을 보겠습니다. 

**[코루틴의 시작과 끝]**
```python
async def f():
	return 123
coro = f()

try:
	coro.send(None) # --------------------- 1)
except StopIteration as e: # -------------- 2)
	print('The answer was: ', e.value)

# The answer was: 123
```
1), 2)에 설명한 과정을 이벤트 루프가 실행하여 코루틴 실행을 제어하기 때문에 최종 사용자 개발자는 `async def` 함수가 `return` 을 통해 반환 값을 받는 것과 동일하게 느낄 수 있습니다. 위 예시는 코루틴 사이의 전환이 어떻게 이루어지는지 자세히 살펴보기 위해 작성된 코드입니다. 

1) 코루틴의 시작점: 코루틴 함수를 작성하고 생성한 코루틴 객체 `coro`를 `loop.create_task(coro)` 혹은 `await coro`로 실행하면 이벤트 루프는 내부적으로, `coro.send(None)` 을 통해 코루틴을 초기화합니다. 
2) 코루틴의 끝점: 코루틴이 반환할 때 `StopIteration`이라는 특별한 예외가 발생합니다. 예외의 `value` 속성을 통해 코루틴의 반환값을 확인합니다. 

#### `await`: 코루틴 완료까지 blocking
`await` 키워드는 항상 매개변수 하나를 필요로합니다. 허용되는 `type`은 `awaitable` 로 불리며 다음 중 하나여야합니다. (Awaitable이란? 다음 글을 참고)
- 코루틴(즉, async def 함수의 반환 값)
- `__await__()`라는 특별 메서드를 구현한 모든 객체. 이 메서드는 반드시 이터레이터를 반환해야합니다. (이 부분은 다루지 않습니다.)

**[await 사용해보기]**
```python
async def f():
	await asyncio.sleep(0)
	return 124

async def main():
	result = await f() # ----------------- 1)
	return result
```
1) `f()`를 호출하면 코루틴을 반환합니다. 이는 `f()`에 대해 `await` 할 수 있다는 뜻입니다. `f()`가 완료되면 `result`의 변수의 값은 `123`이 될 것입니다. 

#### throw(): 코루틴 예외 주입(코루틴 취소)

```python
coro = f()
coro.send(None)
coro.throw(Exception, 'cancel!!')

# Traceback (most recent call last):
#     File "<stdin>", line 1, in <module>
#     File "<stdin>", line 2, in f
# Exeption: cancel!!
# cancel!!
```

코루틴 초기화 후 `throw()`를 호출하여 예외 클래스와 값을 전달하면 코루틴 내에 예외가 발생합니다. `asyncio` 내에서 `throw()`메서드를 사용하여 태스크 취소를 수행합니다. 

```python
import asyncio

async def f():
	try:
		while True: await asyncio.sleep(0)
	except asyncio.CancelledError:
		print('I was cancelled!')
	else:
		return 111

coro = f()
coro.send(None)
coro.send(None)
coro.throw(asyncio.CancelledError)
# I was cancelled!
# Traceback (most recent call last):
#     File "<stdin>", line 1, in <module>
# StopIteration 
```

이벤트 루프는 코루틴 함수의 실행과 취소를 위해 `send()`와 `throw()`메서드를 실행합니다. 태스크가 취소되며 예외가 전파되지 않도록 코루틴이 `return` 되도록하고 코루틴은 **정상종료**합니다. (`StopIteration` 예외는 코루틴이 종료하는 일반적인 방법입니다.)

#### 이제 진짜 `asyncio`의 `loop`를 사용해보자. 
```python
async def f():
	await asyncio.sleep(0)
	return 111

loop = asyncio.get_event_loop() #---------------- 1)
coro = f()
loop.run_until_complete(coro) #------------------ 2)
# 111
```
1) 이벤트 루프를 얻습니다.
2) 코루틴을 실행하고 완료합니다. 이벤트 루프가 앞서 본 예제에서 한 일을 수행합니다. (`.send(None)` 호출하고 `StopIteration` 예외를 통해 코루틴의 완료 및 반환값을 확인합니다.)

### 이벤트 루프
`asyncio`의 이벤트 루프는 `send()`, `throw()`를 통한 비동기 처리 이외에도 코루틴 간 전환, StopIteration 예외 처리, 소켓과 파일 디스크립터의 이벤트 수신도 처리합니다. 

한 개의 이벤트 루프는 단일 스레드에서 동작합니다. 코루틴은 이벤트 루프의 Task Queue에 등록되며, 이벤트 루프와 등록된 코루틴들은 서로 제어권을 주고받으며 실행됩니다. 이벤트 루프는 우선 순위가 높은 코루틴에게 제어권을 주고 코루틴이 I/O 바운드 작업을 요청하거나, 대기가 필요한 경우 다시 제어권은 이벤트 루프로 넘어옵니다. 

![img](https://github.com/aohus/aohus.github.io/blob/main/assets/images/posts/2024-01-24-eventloop.png?raw=true)


## 이걸로 무엇을 할 수 있을까?
`Asyncio`와 그 기반이 되는 코루틴, 이벤트 루프에 대해 공부했습니다. 
다음엔 **네트워크 I/O 바운드 작업에 비동기 코드를 활용하고 실제 이점이 무엇인지 살펴보고**, **`Future`와 `Task`에 대하여** 조금 더 공부하고 기록해보려고합니다. 

----
## References
- 책: 파이썬 비동기 라이브러리 Asyncio
- [PEP 492 – Coroutines with async and await syntax](https://peps.python.org/pep-0492/)
