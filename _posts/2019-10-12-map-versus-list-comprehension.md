---
title: <Python> Map vs List Comprehension
date: 2019/10/12
category: Python
comments: true
---

[List comprehension vs Map](https://stackoverflow.com/questions/1247486/list-comprehension-vs-map)
위의 Stack Overflow 질문에 아주 좋은 답변들이 달려 있어서,
Upvote 상위 두 개 답변을 살펴보았다.

#### 첫 번째 답변 :
Map이 몇몇 경우에 아주 약간 더 빠르다. (lambda 안 쓰고, 같은 기능을 하는 함수를 사용할 경우)
List Comprehension은 나머지 경우에서 더 빠르며, 대부분의 파이썬 사용자들은 List Comprehension이 더 직관적이고 명확하다고 생각한다.

```Python
# 터미널에서 아래와 같이 실행해보자

$ python -mtimeit -s'xs=range(10)' 'map(hex, xs)'
100000 loops, best of 3: 4.86 usec per loop
# hex() -> 16진수로 변경

$ python -mtimeit -s'xs=range(10)' '[hex(x) for x in xs]'
100000 loops, best of 3: 5.58 usec per loop
# 그냥 함수를 사용했더니, map이 근소하게 더 빠르다
```
<br>
하지만 lambda를 쓰면 어떨까?

```Python
$ python -mtimeit -s'xs=range(10)' 'map(lambda x: x+2, xs)'
100000 loops, best of 3: 4.24 usec per loop
$ python -mtimeit -s'xs=range(10)' '[x+2 for x in xs]'
100000 loops, best of 3: 2.32 usec per loop

# 속도가 정 반대가 되었다.
```

***

#### 두 번째 답변의 일부 발췌 :
##### Laziness
Python에서 Map은 **게으르다**.<br>
무슨 말인고 하니, 계산 결과 전체를 반환하는 것이 아니라,
계산 로직을 보관하고 있다가 값 요청이 왔을 때 계산하여 값을 제공해준다는 것이다.

```Python
>>> map(str, range(10**100))
<map object at 0x2201d50>
# 리스트가 아니다
```
<br>
List Comprehension이라면 전체 계산결과 리스트를 반환한다. (Not lazy)

```Python
>>> [str(n) for n in range(10**100)]
# 이런 짓 하지 말라는 것이다.
# DO NOT TRY THIS AT HOME OR YOU WILL BE SAD #
```
<br>
'게으른' List Comprehension도 Generator expression의 형태로 지원한다.

```Python
>>> (str(n) for n in range(10**100))
<generator object <genexpr> at 0xacbdef>
```

