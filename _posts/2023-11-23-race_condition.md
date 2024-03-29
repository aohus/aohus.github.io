---
layout: post
title: "[OS] Race Condition"
subtitle:
categories: OS
tags: []
---
  
# Race Condition  
  
### 임계 구역  
공유 자원 접근 순서에 따라 실행 결과가 달라지는 프로그램의 코드 영역을 임계구역(critical section)이라고 한다. 즉, 임계 구역 안에서 race condition이 발생하는 것이다.  
### 경쟁 조건(Race Condition)  
race condition이란 두 개 이상의 프로세스가 공통 자원을 병행적으로(concurrently) 읽거나 쓰는 동작을 할 때, 공용 데이터에 대한 접근이 어떤 순서에 따라 이루어졌는지에 따라 그 실행 결과가 같지 않고 달라지는 상황을 말한다. Race의 뜻 그대로, 간단히 말하면 경쟁하는 상태, 즉 두 개의 스레드가 하나의 자원을 놓고 서로 사용하려고 경쟁하는 상황을 말한다.  
  
### 임계 구역 문제 해결 조건  
critical section 문제를 해결할 수 있는 방법은 다음 3가지 조건을 만족해야 한다.  
1) **상호 배제(mutual exclusion)**    
    한 프로세스가 임계구역에 들어가면 다른 프로세스는 임계구역에 들어갈 수 없다.  
      
2) **한정 대기(bounded waiting)**    
    상호 배제 때문에 기다리게 되는 프로세스가 무한 대기하지 않아야 한다. 즉, 틍정 프로세스가 임계구역에 진입하지 못하면 안된다.  
      
3) **진행의 융통성(progress flexibility, progress)**    
    임계구역에 프로세스가 없다면 어떠한 프로세스라도 들어가서 자원을 활용할 수 있다. 즉, 두 프로세스가 자원을 번갈아 쓴다고 가정할 때 한 쪽에서 자원을 안쓰고 있다고해서 다른 한 쪽 프로세스가 자원을 쓰고싶어도 자신의 turn이 아니라고 기다리는 것은 효율적이지 못하다는 것이다.  
  
### 임계 구역 문제 해결 방법  
Mutex와 Semaphore는   
- 상호배제 / 한정 대기 만족  
	- 공유 자원(RS)이 free인지 확인하고 lock(or RS--)을 거는 작업을 atomic 하게 하여 context switching 타이밍으로 인해 두 프로세스가 공유 자원에 접근하거나, 무한대기에 빠지는 상황을 방지한다.   
- 진행의 융통성  
	- while을 무한으로 돌며   
  
1) Mutex  
	Mutex란 1개의 스레드만이 공유 자원에 접근할 수 있도록 하여, 경쟁 상황(race condition)를 방지하는 기법이다. 공유 자원을 점유하는 thread가 lock을 걸면, 다른 thread는 unlock 상태가 될 때까지 해당 자원에 접근할 수 없다.  
  
2) Semaphore  
	Semaphore란 S개의 thread만이 공유 자원에 접근할 수 있도록 제어하는 동기화 기법이다. Semaphore 기법에서는 정수형 변수 S(세마포) 값을 가용한 자원의 수로 초기화하고, 자원에 접근할 때는 `S--` 연산을 수행하여 세마포 값을 감소시키고 자원을 방출할 때는 `S++` 연산을 수행하여 세마포 값을 증가시킨다. 이 때 세마포 값이 0이 되면 모든 자원이 사용 중임을 의미하고, 이후 자원을 사용하려는 프로세스는 세마포 값이 0보다 커질 때까지 block 된다.  
  
### 교착상태(Dead Lock)  
Mutex나 Semaphore 등 상호배제 매커니즘을 사용하면 교착상태에 빠질 수 있다. 교착상태란 **둘 이상의 스레드가 서로의 자원이 반납되기를 기다리는 상태에서 무한대기에 빠지는 것**을 말한다.  
  
deadlock이 발생하는 조건 (아래 네 조건 동시에 만족해야한다.)  
- 상호 배제(mutual exclusion)  
- 점유 대기(hold-and-wait)  
- 비선점(no preemption)  
- 순환 대기(circular wait)  
  
deadlock 문제를 해결하는 방법  
- 무시, 예방, 회피, 탐지-회복  
  
  