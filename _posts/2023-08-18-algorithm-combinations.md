---
layout: post
title: "[Algorithm] 조합"
subtitle:
categories: algorithm
tags: []
---

문제 : [리트코드 조합](https://leetcode.com/problems/combinations/)

```
전체 수 n을 입력받아 k개의 조합을 리턴

Ex.
Input: n = 4, k = 2
Output: [[1,2],[1,3],[1,4],[2,3],[2,4],[3,4]]
```

### dfs 이용한 풀이

#### 첫 시도

```python
def combineDirty(self, n: int, k: int) -> List[List[int]]:
    answer = []
    def dfs(elements, rest):
        for r in rest :
            elements.append(r)
            if len(elements) == k :
                answer.append(elements[:])
                elements.pop()
                continue
            next = [ i for i in rest if i > r ]
            dfs(elements, next)
            elements.pop()

    elements = [ i for i in range(1, n+1) ]
    dfs([],elements)
    return answer
```

첫 시도에는 combine을 복사해서 answer에 추가하는 것을 잊고 combine을 참조하도록 했다(`answer.append(combine)` 이렇게) answer 내부에 combine이 자꾸 바뀌어서 아차차하고 `answer.append(combine) -> answer.append(combine[:])`으로 변경하였으나 코드가 알아보기 어려웠다.
속도와 메모리 사용율도 형편없는 풀이였다.

RunTime: 951ms (Beats 5.06%)
Memory: 64.96mb (Beats 7.57%)

#### 수정

```python
def combine(self, n:int, k:int) -> List[List[int]]:
    answer=[]
    def dfs(elements, start: int, k: int):
        if k == 0 :
            answer.append(elements[:])
            return
        for i in range(start, n+1):
            elements.append(i)
            dfs(elements, i+1, k-1)
            elements.pop()
    dfs([],1,k)
```

위 코드는 두 가지를 수정한 것이다.

1. dfs 종료 조건문을 for문 밖으로 분리
   - dfs 종료 조건문을 밖으로 분리하며 종료의 기준이 더 잘 보이게 되었음
   - combine.pop()을 두 번 하지 않아도 되게 되어 중복을 제거했음
2. dfs 함수에 elements(추가한 원소보다 큰 원소의 리스트)를 넘겨주지 않고, start integer와 앞으로 추가해야할 원소의 수(k)를 넘겨줌
   - list comprehention 안에서 매번 넘겨받은 모든 원소를 읽고 조건문을 처리하지 않아도 되어 속도가 빨라짐

코드는 깔끔해졌지만, 리트코드 기준 Runtime, memory는 크게 다르지 않았다. 2번을 수정한 덕분에 조금 더 빠른 정도로 나온 것으로 보인다.
RunTime: 897ms (Beats 9.00%)
Memory: 64.96mb (Beats 7.57%)

**++ 추가**

dfs 함수 for 문의 코드도 아래처럼 줄여 메모리를 조금 더 효율적으로 사용할 수 있다.(여전히 Beats 12.58로 그닥 효율적인 편은 아니지만)

```python
# 기존
elements.append(i)
dfs(elements, i+1, k-1)
elements.pop()

# 변경
dfs(elements+[i], i+1, k-1)
```

### itertools 활용

내장된 라이브러리를 사용할 수 있다면 itertools를 활용하는 것이 좋은 선택이다.

```python
def combineTools(self, n, k):
        return list(itertools.combinations(range(1,n+1), k))
```

RunTime: 736ms (Beats 15.12%)
Memory: 62.1mb (Beats 14.70%)
