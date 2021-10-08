---
title: <Python> Iterable, Iterator, Generator - 너희는 누구니?
date: 2019/10/12
category: Python
comments: true
---
[Iterables vs Iterators vs Generators](https://nvie.com/posts/iterators-vs-generators/)
를 보고 그대로 작성하였다.

![Oops, Some network ants ate this pic!](https://drive.google.com/uc?id=1m5WUphiPHDPecjasiirM40shy2Vhu8qg "relationship between objects")

## Iterable
 - Iterator를 반환할 수 있다면, 어떤 오브젝트여도 Iterable함
 (Data structure일 필요는 없음)
 - 그럼 Iterator와 Iterable의 차이는 뭐지?
```Python
>>> x = [1, 2, 3]
>>> y = iter(x)
>>> z = iter(x) 
#The iter() method returns an iterator for the given object.
>>> next(y)
1
>>> next(y)
2
>>> next(z)
1
>>> type(x)
<class 'list'>
>>> type(y)
<class 'list_iterator'>
```
x는 iterable, y와 z는 iterator인 instance이며, iterable x로부터 값을 만들어낸다. 
y와 z는 둘 다 상태를 저장하고 있다. 
(next(y)를 두 번 호출할 시, 처음 한 번 호출한 상태를 기억하고 있다.)
x는 data structure지만(list), iterable 객체가 꼭 data sturcture일 필요는 없다.

>**참고사항 :**
iterable class가 __iter()__와 __next()__를 모두 가지고 있고, __iter()__가 self(자기 자신)를 반환하는 경우가 있다. 
이는 class를 iterable인 동시에 자기 자신의 iterator로 만든다. 

<br>
그럼 이제 아래와 같이 쓰면:
```Python
x = [1, 2, 3]
for elem in x:
    ...
```
실제로는 이런 일이 일어난다.

![Oops, Some network ants ate this pic!](https://drive.google.com/uc?id=1NmzViVcqdWTguVsGz6-iYPIIxNDaPCIh "The scene when you apply")

이 코드를 disassemble 해 보면, iter(x)를 실행하는 데 필요한 GET_ITER의 호출을 볼 수 있다. FOR_ITER는 next()를 반복적으로 호출하는 것과 같은 동작을 하는 명령어([Instruction](https://www.computerhope.com/jargon/c/compinst.htm))다. 그런데 이는 이진 코드의 형태를 보여주지는 않는데, interpreter의 속도를 위해 최적화되어 있기 때문이다.

```Python
>>> import dis
>>> x = [1, 2, 3]
>>> dis.dis('for _ in x: pass')
  1           0 SETUP_LOOP              14 (to 17)
              3 LOAD_NAME                0 (x)
              6 GET_ITER
        >>    7 FOR_ITER                 6 (to 16)
             10 STORE_NAME               1 (_)
             13 JUMP_ABSOLUTE            7
        >>   16 POP_BLOCK
        >>   17 LOAD_CONST               0 (None)
             20 RETURN_VALUE
```

## Iterator
그럼 도대체 Iterator가 뭐냐? next()를 호출하였을 때, 다음 값을 발생시키는 [Stateful](https://softwareengineering.stackexchange.com/questions/331158/stateful-vs-stateless-non-web-app-applications)(상태를 기억하여 다음 동작에 활용이 가능한) helper object다.
(현재 몇 번째인지 기억해야 다음 값을 불러올 수 있을테니 Stateful이어야 함이 직관적으로 이해된다.)

`__next__()` 메소드를 가지고 있는 모든 오브젝트는 Iterator다.
어떻게 다음 값을 발생시키는지는 상관이 없다.

Iterator는 값 공장이라고 할 수 있다. Iterator에 "다음 값"을 요청할 때마다, Iterator는 저장해 둔 내부 상태를 기반으로 다음 값을 생산하여 반환한다.

**itertools** 모듈의 함수들을 예시로 살펴보자.
몇몇 함수들은 무한한 순서 값들(sequences)을 발생시킨다:
```Python
>>> from itertools import count
>>> counter = count(start=13)
>>> next(counter)
13
>>> next(counter)
14
```
<br>
몇몇 함수들은 유한한 순서 값들에서 무한한 순서 값들을 발생시킨다: 
```Python
>>> from itertools import cycle
>>> colors = cycle(['red', 'white', 'blue'])
>>> next(colors)
'red'
>>> next(colors)
'white'
>>> next(colors)
'blue'
>>> next(colors)
'red'
```
<br>
몇몇 함수들은 무한한 순서 값들에서 유한한 순서 값들을 발생시킨다:
```Python
>>> from itertools import islice
>>> colors = cycle(['red', 'white', 'blue'])  # infinite
>>> limited = islice(colors, 0, 4)            # finite
>>> for x in limited:                         # so safe to use for-loop on
...     print(x)
red
white
blue
red
```
<br>
Iterator의 내부 구조에 대한 더 나은 이해를 얻기 위해서, 
피보나치 수열을 생성하는 Iterator를 만들어 보자:

```Python
>>> class fib:
...     def __init__(self):
...         self.prev = 0
...         self.curr = 1
... 
...     def __iter__(self):
...         return self
... 
...     def __next__(self):
...         value = self.curr
...         self.curr += self.prev
...         self.prev = value
...         return value
...
>>> f = fib()
>>> list(islice(f, 0, 10))
[1, 1, 2, 3, 5, 8, 13, 21, 34, 55]
```
이 클래스가 iterable이면서 (`__iter()__`를 가지고 있기 때문), 
자기 자신의 iterator임을 (`__next()__`를 가지고 있기 때문) 기억하자.

이 iterator의 내부 상태는 prev와 curr에 저장되며, iterator의 다음 호출에 사용된다.
next() 호출은 다음의 두 가지 중요한 일을 한다:
1. 다음 next() 호출을 위해 상태를 변경한다.
1. 현재 호출을 위한 결과를 생성한다.
<br>

>**주요 개념** : 게으른(lazy) 공장
밖에서 보면, iterator는 값을 요청하기 전까지 정지해 있는 게으른 공장처럼 보인다. 이 게으른 공장은 요청을 받으면, 값 하나를 생산한 후에 다시 정지 상태로 돌아가게 된다.

## Generator
드디어 목적지에 도차했다! Generator는 내가(글쓴이가) 완전 좋아하는 Python의 특징이다. Generator는 Iterator의 우아한 버전이다.

Generator는 `__iter()__`와 `__next()__` 메소드를 사용하지 않고도,
Iterator를 작성할 수 있게 해 준다.

직관적으로 표현하면:
1. 모든 Generator는 Iterator이다. (반대는 아니다!)
2. 모든 Generator는, 그러므로, 값을 게으르게 생산하는 공장이다.

여기 Generator로 쓰여진, 피보나치 수열 공장이 있다:

```Python
>>> def fib():
...     prev, curr = 0, 1
...     while True:
...         yield curr
...         prev, curr = curr, prev + curr
...
>>> f = fib()
>>> list(islice(f, 0, 10))
[1, 1, 2, 3, 5, 8, 13, 21, 34, 55]
```

우아하지 않은가? 이 아름다움을 담당하는 마법의 키워드를 보자:
> yield

<br>

뭐가 일어나는지 파헤쳐보자: 처음에는, fib가 평범한 함수로 정의된다. 특별할 것이 없다. 그러나, 함수 안에 Return 키워드가 없다. 반환값은 Generator가 된다.

이제 **f=fib()** 가 호출되면, Generator가 instance화 되고 반환된다. 여기까지는 어떤 코드도 실행되지 않을 것이다. Generator는 초기의 정지 상태에서 시작한다. 
더 직관적으로 보면, **prev, curr = 0,1** 은 아직 실행되지 않았다.

그리고, 이 Generator instance는 islice()에 감싸진다. 
스스로 Iterator이기 때문에, 아직도 아무것도 일어나지 않는다.

그 다음엔, 이 Iterator는 list()에 의해 감싸진다. 그리고, 이 Iterator는 결과적으로 f instance에서 next()를 호출하는 islice() instance 위에서 next()를 호출하기 시작한다.

한 단계씩 보자. 처음에 이 코드는 **prev, curr = 0,1** 을 실행할 것이고, while True 루프에 들어가서, yield curr 구문과 만난다. 이 구문은 현재 curr 변수에 있는 값을 생산하고
(현재 curr 변수에 할당된 값과 동일한 값을 반환한다는 의미) 다시 정지 상태가 될 것이다.

이 값은 islice() wrapper로 전달되고, list()는 값 1을 리스트에 추가할 수 있게 된다.

그리고, list()는 islice()에 다음 값을 요청한다. islice()는 f에 다음 값을 요청하고, f의 정지상태를 해제한다. f는 **prev, curr = curr, prev + curr** 구분부터 다시 시작하게 된다. 이 다음에 **while** 루프의 다음 반복에 재진입하고, **yield curr** 구문을 만나고, 다음 **curr** 값을 반환한다.

이 동작은 10번 반복된 후, list()가 islice()에 11번째 값을 요청하고, islice()가 끝에 도달했음을 의미하는 **StopIteration** 예외를 발생시킬때까지 일어난다. 그리고 list()는 피보나치 수열 처음 10개 값을 가진 리스트를 반환하게 된다. Generator가 11번째 **next()** 호출을 받지 않았음에 주목해라. 모든 작업이 끝나면 이 Generator는 다시 사용되지 않고, Garbage Collector에 의해 처리된다.

## Type of Generators
두 가지 종류의 Generator가 있다: Generator **function** 과 Generator **expression** 이다.
Generator function은 body에 yield를 가지고 있는 함수이다. 방금 이에 대한 예를 봤다. (피보나치 수열 Generator)
yield 키워드가 있기만 해도 함수는 Generator function이 된다.

다른 타입의 Generator는 list comprehension과 동일한 Generator다.
제곱수의 리스트를 만드는 여러가지 예를 살펴보자.
```Python
>>> numbers = [1, 2, 3, 4, 5, 6]
>>> [x * x for x in numbers]
[1, 4, 9, 16, 25, 36]
```
<br>
set comprehension으로도 동일한 일을 할 수 있다.
```Python
>>> {x * x for x in numbers}
{1, 4, 36, 9, 16, 25}
```
<br>
혹은 dict comprehension으로도:
```Python
>>> {x: x * x for x in numbers}
{1: 1, 2: 4, 3: 9, 4: 16, 5: 25, 6: 36}
```
<br>
Generator expression으로도 사용할 수 있다. (주의 : tuple comprehension 아님)
```Python
>>> lazy_squares = (x * x for x in numbers)
>>> lazy_squares
<generator object <genexpr> at 0x10d1f5510>
>>> next(lazy_squares)
1
>>> list(lazy_squares)
[4, 9, 16, 25, 36]
```
**lazy_squares** 의 첫 번째 값을 **next()** 로 읽었기 때문에, 상태가 "두 번째" 아이템이 되었다.
그래서 list() 호출을 통해서 전부 부르려고 하면, 제곱수의 나머지 리스트만 반환된다. ('게으른 행동'. 마지막 출력결과를 보면 "첫 번째" 아이템인 1이 반환되지 않았다.)

# 요약
Generator는 프로그래밍에 적용 가능한 아주 강력한 구조다. 
streaming code를 더 적은 중간 변수들과 데이터 구조들을 사용하여 만들 수 있게 해주고, 메모리와 CPU 사용 측면에서도 더 효과적이다. 더 적은 라인을 차지함은 물론이다.

Generator 사용 팁 : 코드에서 다음과 같은 일을 할 수 있는 곳을 찾아보자.
```Python
def something():
    result = []
    for ... in ...:
        result.append(x)
    return result
```
<br>
위의 예시를 다음과 같이 바꿔보자.
```Python
def iter_something():
    for ... in ...:
        yield x

# def something():  # Only if you really need a list structure
#     return list(iter_something())
```