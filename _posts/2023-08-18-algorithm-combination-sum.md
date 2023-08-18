---
layout: post
title: "[알고리즘] 조합의 합"
subtitle:
categories: algorithm
tags: []
---

문제: [리트코드 조합의 합](https://leetcode.com/problems/combination-sum)

```
숫자 집합 candidates를 조합하여 합이 target이 되는 원소를 나열.
각 원소는 중복으로 나열 가능함.

Ex.
Input: candidates = [2,3,6,7], target = 7
Output: [[2,2,3],[7]]
```

### dfs 이용한 풀이

#### 첫 시도

```python
def combinationSum(self, candidates: List[int], target: int) -> List[List[int]]:
    answer=[]
    def dfs(elements:List[int], start:int) :
        if sum(elements) == target :
            answer.append(elements[:])
            return
        if sum(elements) > target :
            return

        for i in candidates[start:]:
            elements.append(i)
            dfs(elements, candidates.index(i))
            elements.pop()
    dfs([],0)
    return answer
```

첫 시도는 [리트코드 조합](https://aohus.github.io/python/2023/08/18/algorithm-combinations.html)을 푼 이후에 풀어서 비슷하게 접근했다.
하지만 속도나 메모리 활용 모두가 별로여서 코드를 수정해야했다.

RunTime: 78ms (Beats 41.20%)
Memory: 16.5mb (Beats 24.20%)

#### 수정

```python
def combinationSum(self, candidates: List[int], target: int) -> List[List[int]]:
    answer=[]
    def dfs(csum, index, path) :
        if csum ==  0:
            answer.append(path)
            return
        if csum < 0 :
            return

        for i in range(index, len(cantidates)):
            dfs(csum-candidates[i], i, path+[candidates[i]])
    dfs(target,0,[])
    return answer
```

1. dfs가 target - (원소들의 합)를 인자로 받게하여 sum 함수를 이용하지 않고 바로 종료 조건 판단하도록 함
2. for 문을 index 기준으로 돌도록 하여, 원소를 받아 cadidates 전체의 index를 탐색하는 부분을 수정
3. `path.append(candidates[i])`로 원소를 추가하지 않고 `path+[candidates[i]]`를 인자로 넘겨주어 불필요하게 append하고 pop하는 과정을 생략함

RunTime: 59ms (Beats 85.10%)
Memory: 16.2mb (Beats 97.18%)
