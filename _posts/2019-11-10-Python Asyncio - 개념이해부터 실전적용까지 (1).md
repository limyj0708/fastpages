---
title: <Python> Asyncio - 개념이해부터 실전적용까지 (1)
date: 2019/11/10
category: Python
comments: true
---
Cnet의 기사를 크롤링 하려고 하는데, 총 기사 수가 36만개가 넘는다.
https://www.cnet.com/sitemaps/articles/

음, 좀 많은데? 기사 링크 수집에만 50분이 걸리는데?
내용 수집은 도대체 얼마나 걸릴까?

기사 수집을 더 빠르게 하고 싶다.
한 번에 여러 개의 기사를 수집하게 할 수 없을까?
그래서 이것저것 알아보았는데, 이렇게 긴 여정이 될 거라고는 상상하지 못했다...

<hr style="border-width: 2px;">

# 배경 지식
## Global Interpreter Lock (GIL)
Python에는 Global Interpreter Lock이라는 것이 있다.
{% blockquote Wikipedia https://en.wikipedia.org/wiki/Global_interpreter_lock Global interpreter lock %}
A global interpreter lock (GIL) is a mechanism used in computer-language interpreters to synchronize the execution of threads so that only one native thread can execute at a time.[1] An interpreter that uses GIL always allows exactly one thread to execute at a time, even if run on a multi-core processor. Some popular interpreters that have GIL are CPython and Ruby MRI.
{% endblockquote %}

응~ 한 번에 Thread 하나만 작동시킬꺼야~ 멀티코어여도 안돼~
그 이유는 이 글을 보도록 하자. 너무 쉽게 잘 설명했다.
["What is the Python Global Interpreter Lock (GIL)?"](https://realpython.com/python-gil/)

### 위 글의 요약
* 간단하게 말하면, Global Interpreter Lock은 딱 하나의 thread만 python interpreter에서 동작하게 하는 [mutex(or a lock)](https://en.wikipedia.org/wiki/Mutual_exclusion)다.
* Python의 메모리 관리 : Reference Counting
    * Python에서 생성된 객체는, 객체가 참조된 수를 추적하는 reference count 변수를 가지고 있다. 이 count가 0이 되면, 그 객체에 의해 점유되고 있던 메모리가 해제된다.
    ```Python
    import sys
    a = []
    b = a
    print(sys.getrefcount(a))
    # 출력값 : 3
    ```
    refcount가 3인 이유는, 빈 리스트 `[]`가 a, b, sys.getrefcount의 argument 이렇게 세 곳에서 참조되고 있기 때문이다.
* [Race Condition](https://jhnyang.tistory.com/35) :
    * 두 개 이상의 thread들이 동기화 메커니즘 없이 공유된 자원에 접근하려고 하는 상황.
    * reference count에 영향을 미치는 두 thread가 있다. 동시에 count를 바꾸려고 한다.
        * 메모리가 영원히 해제되지 않거나, 오브젝트가 아직 존재하는데 메모리를 해제해 버릴 수도 있다. 그래서...
* Race Condition으로부터 refernce Count를 보호하기 위해, thread들 간에 공유되는 모든 data structure들에 lock을 걸어줘야 한다.
    * 그런데 여러 객체, 혹은 객체 묶음에 여러 lock을 걸어주는 것은 [Deadlock](https://namu.wiki/w/Deadlock), 성능 저하(할당하고 풀고 할당하고 풀고...) 등의 문제를 가져온다.
* GIL은 interpreter의 단일 lock인데, "python bytecode의 실행은 interpreter lock의 획득을 필요로 한다" 라는 규칙을 추가한다. lock이 하나뿐이므로 deadlock도 없고, lock의 할당과 해제에서 발생하는 overhead도 없다. 그리고 필연적으로 한 번에 실행되는 thread도 하나가 된다.

#### 그렇구나. 하지만 난 Single thread로는 만족할 수 없는 몸인걸...?
<hr style="border-width: 0.3px;">

## Multi-Threading
* Python에서 제공되는 threading 이라는 모듈을 사용한다.
* Multi 라고 말은 하지만 그렇게 보이는 것 뿐이다. 결국 한 번에 실행되는 thread는 하나다. GIL의 속박에서 벗어날 수 없으리라!
![Oops, Some network ants ate this pic!](https://drive.google.com/uc?id=1W4CxrjSP-2R7Uj7aiZWLom3D8xSxCrb7 "Truth of MultiThreading") Source : [Concurrent Programming in Python is not what you think it is.](https://hackernoon.com/concurrent-programming-in-python-is-not-what-you-think-it-is-b6439c3f3e6a)


* Lock 할당과 해제 작업이 계속 일어나기 때문에, CPU-bound 프로그램에서는 Single-thread보다 속도가 느린 참사가 발생한다. < [예시](https://realpython.com/python-gil/#the-impact-on-multi-threaded-python-programs)
* I/O bound한 작업에서는, thread가 대부분의 시간을 I/O에 할당하고 있고, 이는 CPU에 관련된 작업이 아니다. URL 요청이라고 하면, 아래와 같은 그림이 될 것이다. 성능 향상 가능! < [예시](https://medium.com/towards-artificial-intelligence/the-why-when-and-how-of-using-python-multi-threading-and-multi-processing-afd1b8a8ecca)
![Oops, Some network ants ate this pic!](https://drive.google.com/uc?id=1IIPZgoP0ZneP0XgpM-J0sIbcolDhV_cC "IO Bound Multithreading") Source : [Speed Up Your Python Program With Concurrency](https://realpython.com/python-concurrency/)

* Threading 튜토리얼은 [여기](hhttps://realpython.com/intro-to-python-threading/)를 참고하자.

## Multi-Processing
* 요즘 CPU들 중에 멀티코어가 아닌 것이 있을까?
* 아래 두 장의 사진으로 컨셉을 설명 가능하다. 
![Oops, Some network ants ate this pic!](https://drive.google.com/uc?id=1X1a4VSZQzvIfCq7HxTroSpCkChNLggIl "IO Bound Multithreading") Source : [Speed Up Your Python Program With Concurrency](https://realpython.com/python-concurrency/#multiprocessing-version)
![Oops, Some network ants ate this pic!](https://drive.google.com/uc?id=1wm9CZDE4PXycgOnDibSfWANP2L-Sr6r4 "IO Bound Multithreading") Source : [MULTIPROCESSING IN PYTHON](https://datanoon.com/blog/multiprocessing_in_python/)
    * 별도의 메모리를 가지는 프로세스를 생성하여, 각자 다른 코어에 할당, 목표를 처리한다.
* Multiprocessing은 CPU-bound 프로그램의 속도 향상에 유리하다. 수치 계산이라던가.
* Multiprocessing 튜토리얼은 [여기](https://datanoon.com/blog/multiprocessing_in_python/)를 참고하자.

크롤링의 경우 I/O bound 작업이므로 Multi-Threading을 사용하면 될 것 같다. 그런데, Multi-Threading에는 마음에 안 드는 점이 있다.
* Multi-Threading 방식에서는, OS가 thread 사이의 전환을 관리한다. [preemptive multitasking](https://en.wikipedia.org/wiki/Preemption_%28computing%29#Preemptive_multitasking) 이라고 불리는데, OS가 context switch를 하기 위해 thread를 잠시 멈출 수 있는 방식이다.
    * 코드에 switching에 관련된 내용을 추가하지 않아도 되기 때문에 편리하지만, 언제 switching 될지 며느리도 모른다는 것이 문제다. 전환되면 안 되는 구문에서 전환되면 어떻게 할 것인가?
* thread를 여러 개 늘리게 되면, 메모리를 많이 점유하게 된다.

어쩌지?

<hr style="border-width: 0.3px;">

# Asyncio
* [Asynchronous I/O](https://docs.python.org/3/library/asyncio.html)
* <b style="color:darkred">반드시 먼저 읽기</b> : [Asychronous, Synchronous, Blocking, Non-Blocking](https://limyj0708.github.io/2019/11/11/Asynchronous,%20Synchronous,%20Blocking,%20Non-Blocking/)
* Asyncio 라이브러리는 [cooperative multitasking](https://en.wikipedia.org/wiki/Cooperative_multitasking) 이라는 방식을 사용하는데, (a.k.a non-preemptive multitasking) OS가 context switching을 시작시키지 않고 process, 혹은 thread가 자발적으로 "나 멈추고 CPU 점유 넘길게요!" 라고 선언하는 방식이다.
* scheduling 계획 수행을 위해 모든 프로그램들이 "협력" 해야 하므로, "cooperative"라는 형용사가 붙었다.
* 코드를 좀 변경하고 추가해야 하지만, ***언제*** switching이 일어날 지 알 수 있게 된다.

오, 좋아 보인다. 불확실한 요소가 없는 게 좋지.
그래서 어떻게 작동되는 걸까? 일단 알아야 할 것들은 다음과 같다.

## Generator
* [전에 작성한 이 글](https://limyj0708.github.io/2019/10/12/iterator%20iterable%20generator/) 에서 컨셉을 알아보았었다. 컨셉은 알겠는데, 명확한 정의와 실제로 어떻게 사용할 수 있는지가 궁금하므로, 더 알아보자. [Python docs](https://docs.python.org/3/glossary.html#term-generator)에서는 어떻게 정의하고 있냐면...
 > A function which returns a generator iterator. It looks like a normal function except that it contains **yield** expressions for producing a series of values usable in a for-loop or that can be retrieved one at a time with the next() function. Usually refers to a generator function, but may refer to a generator iterator in some contexts. In cases where the intended meaning isn’t clear, using the full terms avoids ambiguity.

 <br>
 
 generator는 generator iterator를 반환하는 함수다. 평범한 함수처럼 보이나, 값 목록을(series of values) 생성하기 위한 [Yield](https://docs.python.org/3/reference/expressions.html?highlight=yield%20expression#yield-expressions) 표현식을 가지고 있다. 이 값 목록은 for-loop에서 사용될 수 있다. 혹은 값 목록에서 값을 next() 함수로 하나씩 끄집어 낼 수 있다. generator는 보통 generator function을 의미하나, 문맥에 따라 generator iterator를 의미하는 경우도 있으므로, 혼동될 수 있는 상황에서는 용어를 명확히 할 것.

 <br>

 [전 글](https://limyj0708.github.io/2019/10/12/iterator%20iterable%20generator/)에서보다 좀 더 실용적인 코드 예시를 살펴보자. 엄청 긴 txt 파일을 메모리에 몽땅 올려놓기 싫어서 만든 간단한 함수다.

 ```Python
def read_txt_linebyline(filepath):
    """
    TXT 파일을 한 줄씩 읽어오는 Generator
    다 읽어오면 StopIteration 에러를 반환한다.
    filepath에는 txt 확장자는 뺴고 적을 것.
    """
    f = open(filepath + '.txt', 'r', encoding='utf-8', newline='')
    while True:
            line = f.readline() # 한 줄씩 읽는다.
            if line == '': # 끝까지 다 읽으면 다음부터는 빈 문자열을 읽어오게 된다.
                f.close()
                break
            yield line

readlines_generator = read_txt_linebyline('적당한 폴더이름' + 'url_list')
print(readlines_generator)
    # <generator object read_txt_linebyline at 0x10d61fa50> 가 출력된다.
next(readlines_generator).replace('\n', '')
    #### 출력값 
        # https://www.theverge.com/2011/05/23/hello-again-chad-mumm
        # txt 파일에서 한 줄 읽어온 내용이 출력된다. replace로 마지막에 붙어있던 개행 문자를 없앴다.
 ```
 보통 함수는 한 번 호출되면 return값을 call한 곳으로 반환 후 종료되며, 메모리에서 클리어 된다. 그런데 위의 함수를 실행하면,
 1. read_txt_linebyline이 arguments들을 받고 호출된다.
 1. read_txt_linebyline은 generator object를 반환한다. `dir()`로 generator object가 가지고 있는 메소드를 확인해보면,
 ```Python
 print(dir(readlines_generator))
 # 출력값 :
 ['__class__', '__del__', '__delattr__', '__dir__'
 '__doc__', '__eq__', '__format__', '__ge__','__getattribute__',
 '__gt__', '__hash__','__init__', '__init_subclass__', '__iter__',
 '__le__', '__lt__', '__name__', '__ne__', '__new__', '__next__',
 '__qualname__', '__reduce__', '__reduce_ex__', '__repr__',
 '__setattr__', '__sizeof__', '__str__','__subclasshook__',
 'close', 'gi_code', 'gi_frame', 'gi_running', 'gi_yieldfrom', 'send', 'throw']
 ```
    `__next__`가 보인다. iterator임을 알 수 있다. [`__iter__`도 있는데, 직접 실행해 보니 자기 자신을 반환한다.](https://docs.python.org/3/library/stdtypes.html#typeiter) iterator가 맞다.
 1. `next(readlines_generator)`로 처음 값을 요청하면, 코드가 실행되다가 처음으로 yield를 만나게 된다. 여기서 txt 파일에서 읽어온 line이 반환되는데(`yield line`), 보통의 함수였다면 반환하고 끝났겠지만 여기서는 함수의 실행이 일단 멈춘다.
 1. `next(read_txt_linebyline)`를 한번 더 호출하면, yield문에서부터 다시 함수가 실행되기 시작한다. `while True:` 안에서 동작하고 있으므로, 반복문의 처음으로 돌아가서 다시 다음 한 줄을 불러오게 된다.
 1. 이후 계속 반복...

 Generator의 메모리 효율성이나 Laziness는 저번에 알아봤으니 굳이 언급하지 않는다.
 그런데, 비동기 프로그래밍에서 왜 generator 이야기를 하는걸까? 좀 더 인내심을 가지고 다음 내용을 살펴보자.


> 이후의 내용은 : [How does asyncio actually work?](https://stackoverflow.com/questions/49005651/how-does-asyncio-actually-work/51116910#51116910)의 흐름을 따라가면서, 더 짚고 넘어가야 할 개념에 대한 설명과 예제를(많이) 추가하였다. 흑흑...

### Generator와 통신하기
 [`send()`](https://docs.python.org/3/reference/expressions.html#generator.send)와 [`throw()`](https://docs.python.org/3/reference/expressions.html#generator.throw) 메소드를 통해 generator와 통신할 수 있다.

 {% blockquote Python Docs https://docs.python.org/3/reference/expressions.html#generator.send generator.send(value) %}
Resumes the execution and “sends” a value into the generator function. The value argument becomes the result of the current yield expression. The send() method returns the next value yielded by the generator, or raises StopIteration if the generator exits without yielding another value. When send() is called to start the generator, it must be called with None as the argument, because there is no yield expression that could receive the value.
 {% endblockquote %}
 <br>
 generator.send(value)로 value를 보내며 다음 값을 호출하면, value가 현재 yield 표현식의 결과값이 된다. 다음 반환값이 없으면 StopIteration 오류를 발생시킨다. generator를 처음 호출할 때 send()를 쓰면 반드시 None을 argument로 넣어줘야 하는데, value를 받아줄 yield 표현식이 처음엔 없기 때문이다.
 
 그럼, 이 내용을 다음 예시 코드로 실습해보자.

 ```Python
def test():
    val = yield 1
    print(val)
    yield 2
    yield 3
gen = text()
print(next(gen), 'N1')
print(next(gen), 'N2')
print(next(gen), 'N3')
#### 출력값. 그냥 출력하면 다음과 같이 나온다.
1 N1
None # (yield 1) 표현식에 아무런 값도 할당이 안 되었으므로 None이 된다.
2 N2
3 N3
###################################


gen2 = test()
gen2.send('sth')
#### 출력값. Generator 시작 시에는 None을 보내야 한다. 값을 받을 yield 표현식이 없으므로 에러가 난다.
#TypeError    Traceback (most recent call last)
#<ipython-input-75-df11fcdec14b> in <module>
#----> 1 get2.send('sth')
#
#TypeError: can't send non-None value to a just-started generator
##################################


gen3 = test()
print(gen3.send(None))
print(gen3.send('sth'))
print(gen3.send('sth'))
#### 출력값.
1 # send의 argument를 None을 넣었더니 제대로 나온다.
sth # val = yield 1에서, (yield 1) 표현식의 값이 sth가 된다.
# 순서가 중요한데, 1 반환 후, (yield 1)부터 다시 시작하게 될 때 (yield 1) 표현식에 값이 할당된다.
2
3
 ```
 `throw()`는 yield로 인해 generator가 멈추는 지점에서 예외를 발생시킨다.

 ```Python
def test2():
    while True:
        try : 
            sth = yield 1
            print(sth)
        except Exception as e:
            print(e)
            print(type(e))

gen = test2()
gen.send(None)
print(gen.send(100))
#### 출력값 #
100
1
###########


print(gen.throw(ArithmeticError, 'hohoho'))
print(get.throw(BufferError, 'huhuhu'))
#### 출력값 ###############
#hohoho
#<class 'ArithmeticError'>
#1
#huhuhu
#<class 'BufferError'>
#1
#########################
 ```

### Generator에서 값 반환
아니 이게 된다고? 네 됩니다.
```Python
def return_test():
    yield 1
    return 'sth'

gen = return_test()

try : 
    next(gen)
except Exception as e:
    print(type(e))
    print(e.value)
    print(dir(e))

# try 블록을 두 번 실행해서 StopIteration을 발동시키면, 출력값이 아래와 같이 된다.

<class 'StopIteration'>
sth
['__cause__', '__class__', '__context__', '__delattr__',
'__dict__', '__dir__', '__doc__', '__eq__', '__format__',
'__ge__', '__getattribute__', '__gt__', '__hash__',
'__init__', '__init_subclass__', '__le__', '__lt__', '__ne__',
'__new__', '__reduce__', '__reduce_ex__', '__repr__',
'__setattr__', '__setstate__', '__sizeof__', '__str__',
'__subclasshook__', '__suppress_context__',
'__traceback__', 'args', 'value', 'with_traceback']
```
[Python 공식 문서에 따르면](https://docs.python.org/3/library/exceptions.html#StopIteration), return 값은 StopIteration class의 'value'라는 attribute에 할당되는데, 이 value는 StopIteration class의 constructor의 parameter다. `dir(e)`의 실행결과를 보면 value가 있음을 알 수 있다.

### 새로운 키워드를 맞이하라! : yield from
* Python 3.4에서 [yield from](https://docs.python.org/3/reference/expressions.html#yield-expressions)이라는 키워드가 생겼다. 이 키워드는 `yield from [expression]`의 구조로 사용되는데, expression을 일종의 iterator(subiterator)로 취급한다. expression에서 생성된 값을 계속 받아오는 것이다.
* `send()`, `throw()`로 expression에 값을 보낼 수 있는데, expression 내부에 이 보내진 값을 활용할 수 있는 메소드가 있다면 잘 사용된다.
 * 그런 메소드가 없으면 `send()`는 AttributeError나 TypeError를 발생시키며, `throw()`는 보내려던 예외를 발생시킨다. 

도대체 이게 무슨 말일까? 예제 코드를 통해 알아보자.

```Python
# Nested generator. generator 안에서 generator를 사용하는 구조다.
def inner():
    print((yield 'a'))
    yield 'b'
    yield 'c'
    return 'boom!'

def outer():
    yield 1
    val = yield from inner()
    print(val)
    yield 4
```
```Python
# 일단 inner()에게 값을 여러 번 호출해 보자.
gen_inner = inner()
next(gen_inner) #1
next(gen_inner) #2
next(gen_inner) #3
next(gen_inner) #4

### 출력값
a #1
None #2
b #2
c #3
StopIteration: boom! #4
```
```Python
gen = outer()
next(gen) #1
next(gen) #2
gen.send('Ignition!') #3
next(gen) #5
next(gen) #4

### 출력값
1 #1
a #2
Ignition! #3
b #3
c #4
boom! #5
4 #5
```
1. #1에서는 outer()의 `yield 1`에서 1이 반환된다.
1. #2에서는 `val = yield from inner()`를 만나는데, yield from은 inner()의 생산값을 iterator에서 불러오는 것처럼 받아오므로, inner()의 첫 번째 생산값인 a를 반환한다.
1. #3에서는 `send('Ignition!')`이 yield from의 대상인 inner()에게 'Ignition!'을 전달한다. `inner().send('Ignition!')`처럼 실행되며, (yield 'a')표현식의 반환값이 'Ignition!'이 되므로 Ignition!이 출력된다. 이후 `yield 'b'`가 b를 반환한다.
1. #4에서는 inner()안의 `yield 'c'`가 c를 반환한다.
 * inner() 내부를 전부 탐색할때까지 outer()의 다음 코드가 실행되지 않는다. 마치 iterator에서 값 받아오는 것 같다.(subiterator)
1. #5에서는, inner()의 `return 'boom!'`에 도달한다. 더 반복할 것이 없으므로 StopIteration 예외가 발동되며, boom!이 StopIteration의 value로 묻어 나오게 되는데, 이게 (yield from inner()) 표현식의 반환값이 된다. 그래서 `val`이 boom!이 되고, `print(val)`이 boom!을 출력한다. 이후 outer()의 `yield 4`가 4를 반환한다.

### 이 요소들을 다 합치면...
generator안에 generator를 만들 수 있음을 알게 되었다.
안쪽 generator와 바깥쪽 generator 사이에 데이터 전달도 가능함도 알게 되었다. 여기서 generator에게 새로운 의미가 주어지는데, ***Coroutine***이라는 개념이다.

<hr style="border-width: 0.3px;">

## Coroutine
**Coroutine**은 함수의 일반화된 형태로, 실행 도중 멈춤/재실행(suspend/resume)이 가능하다. 최초 실행 시의 진입점과, 재실행 시의 진입점이 따로 있으므로, 진입점이 여러 개(multiple entry point)인 것이 특징이라고 말해지기도 한다.

{% blockquote Wikipedia http://masnun.com/2015/11/20/python-asyncio-future-task-and-the-event-loop.html Coroutine %}
Coroutines are computer program components that generalize subroutines for non-preemptive multitasking, by allowing execution to be suspended and resumed. 
{% endblockquote %}
응? 그럼 coroutine이 generator와 다른 점은 뭐지?
{% blockquote Python Docs https://docs.python.org/3/reference/expressions.html#yieldexpr Yield expressions%}
All of this makes generator functions quite similar to coroutines; they yield multiple times, they have more than one entry point and their execution can be suspended. The only difference is that a generator function cannot control where should the execution continue after it yields; the control is always transferred to the generator's caller.
{% endblockquote %}
generator의 경우, 잠깐 멈췄을 때(yield) 실행을 어디로 넘길 것인지 조정할 수가 없다. 무조건 그 generator의 caller에게 가게 된다. coroutine은 이걸 조정 가능한 것 같은데, 예를 들면 자신의 caller가 아니라 다른 coroutine에게 넘기는 식이다. 코드도 그런 식으로 작동하고.

그럼, asyncio에서는 coroutine을 어떻게 사용하는걸까?
coroutine은 [`async def`](https://docs.python.org/3/reference/compound_stmts.html#coroutine-function-definition)라는 키워드를 통해 정의된다. coroutine은 generator가 yield from을 사용하는 것처럼, [`await`](https://docs.python.org/3/reference/expressions.html#await)라는 키워드를 사용한다. 이 두 키워드가 Python 3.5버전에서 만들어지기 전에는, coroutine을 만들 때 generator처럼 yield from을 사용했었다.

짤막한 예제를 하나 보자. [출처](https://docs.python.org/3.5/library/asyncio-task.html#example-chain-coroutines)
```Python
import asyncio

async def compute(x, y): # 나는 coroutine!
    print("Compute %s + %s ..." % (x, y))
    await asyncio.sleep(1.0) # 1초 기다릴래! 그런데 기다리는 동안 난 제어 권한을 넘길(yield)거야! 다른 작업 해볼래?
    return x + y

async def print_sum(x, y): # 나도 coroutine!
    result = await compute(x, y) # compute가 완료될 때까지 기다릴래! 그런데 기다리는 동안 다른 작업을 실행할 수 있어!
    print("%s + %s = %s" % (x, y, result))

loop = asyncio.get_event_loop()
loop.run_until_complete(print_sum(1, 2))
loop.close()

# 최신 고수준 API에서는
asyncio.run(print_sum(1,2))
# 라고 하면 위의 loop 3줄을 대체할 수 있다.
# 예제가 Python 3.5.9 기준이라 loop 3줄을 쓰고 있다.
# 하지만 기본 원리를 직관적으로 알고 싶으므로 이대로 진행해보자.

# 출력값은 아래와 같다.
# Compute 1 + 2 ...
#  1 + 2 = 3
```
출력값은 별거 없어 보이는데, 뒤에서 어떻게 작동되냐면...

> compute() is chained to print_sum(): print_sum() coroutine waits until compute() is completed before returning its result.

![Oops, Some network ants ate this pic!](https://drive.google.com/uc?id=1F0RcFEy62nMG14Hv1XaOZtysM24Waiwk "Asyncio example background") Source : [Python 3.5.9 : Tasks and coroutines](https://docs.python.org/3.5/library/asyncio-task.html#example-chain-coroutines)

> The “Task” is created by the AbstractEventLoop.run_until_complete() method when it gets a coroutine object instead of a task. The diagram shows the control flow, it does not describe exactly how things work internally. For example, the sleep coroutine creates an internal future which uses AbstractEventLoop.call_later() to wake up the task in 1 second.

**Event loop**와 **Task**는 뭘까? 그림만 보면, Event loop는 Task의 요청을 기다리는 무한 루프고, Task는 coroutine의 실행 스케쥴을 관리하는 객체처럼 보이는데.

## Event loop
* 말 그대로 loop인데, 여러 task들을 실행하고, 멈추고, 실행을 취소(cancel)시키는 loop다.
* 여러 비동기적 함수들(async function)을 event loop에 할당하여 스케쥴링을 하게 된다.
* 예를 들어, A라는 함수를 loop가 실행하다가, A가 IO 대기를 한다고 가정해 보자. event loop는 A의 실행을 멈추고 다른 함수를 실행한다. A의 IO가 완료되면, 다시 A를 실행한다.
 {% blockquote PYTHON ASYNCIO: FUTURE, TASK AND THE EVENT LOOP http://masnun.com/2015/11/20/python-asyncio-future-task-and-the-event-loop.html Event Loop %}
 The internals of the event loop is quite complex and we don’t need to worry much about it right away. We just need to remember that the event loop is the mechanism through which we can schedule our async functions and get them executed.
 {% endblockquote %}

## Futures / Tasks
* Future : 결과를 받을 것으로 '예상'되는 객체다. Future 객체가 결과를 받기를 기다리는 동안 다른 연산을 수행할 수 있다.
* Task : Future의 하위 클래스(subclass)인데, coroutine을 wrapping한다. coroutine이 완료되면, task가 그 결과값을 받는다.

위의 예제에서는, task 객체가 생성되어 coroutine print_sum()과 compute()를 감싸고, 이들의 실행결과를 기다리다가-받는다. task들은 event loop에 등록되며, event loop가 task들의 실행을 관리한다.

이제 어떻게 돌아가는지 알았다.
그럼 이걸로 어떻게 비동기적으로 http 요청을 받고, 저장할 수 있을까?

# 비동기적으로 Http 응답 받기
requests는 [blocking](https://nesoy.github.io/articles/2017-01/Synchronized) 라이브러리다. http 응답을 기다리는 동안 아무것도 못 한다는 것이다. [requests로도 어떻게 할 수는 있는데](https://sjquant.tistory.com/14?category=797018), 역시나 내부적으로는 multi-threading으로 동작하는 것이어서 비동기적 처리가 아니다.

비동기적 http 요청을 보낼 수 있는 `aiohttp` 라이브러리를 사용하자.
간단한 예제 코드를 살펴보면,
```Python
import asyncio
import aiohttp
from bs4 import BeautifulSoup
import time

async def get_text_from_url(url):
    async with aiohttp.ClientSession() as session: # 다 끝나면 세션을 닫을것이다.
        async with session.get(url, headers={'user-agent': 'Mozilla/5.0'}) as req: # 다 끝나면 get 요청도 닫을것이다.
            html = await req.text() # 여기서 시간이 오래 걸리므로 기다린다.
    print(f'Get response from {url}')
    soup = BeautifulSoup(html, 'html.parser')
    print(soup.text[:100].strip())

async def main():
    urls = ['http://google.com', 'http://naver.com', 'http://daum.net'\
    ,'http://depromeet.com', 'http://facebook.com']*4

    # 실행해서 결과 받아올래~ 라는 coroutine 목록을 만든다.
    futures = (get_text_from_url(url) for url in urls) 
    # generator expression으로 했는데, list comprehension으로 해도 된다.
    # 대상이 엄청나게 많을 경우를 생각한다면 url도 generator로 하나씩  생성하는 형태로 만들고,
    # coroutine 생성도 generator expression으로 하는게 좋아 보인다.

    await asyncio.gather(*futures)
    # 첫 번째 인자로 받은 awaitable 객체들을 실행한다.
    # 받은 객체들이 coroutine이면, 알아서 task로서 스케쥴링이 된다.
    # awaitable 객체들이 다 끝나면, 각 awaitable 객체들의 반환값을 리스트로 묶어서 반환한다.
    # 리스트 내의 반환값의 순서는 첫 번째 인자 - 여기서는 *futures -
    # 내의 awaitable 객체들의 순서를 따라간다.
    # https://docs.python.org/3/library/asyncio-task.html

start = time.time()
asyncio.run(main())
end = time.time()
print(f'걸린 시간 : {end-start}')
```
실행하면 대략 2.5초가 걸린다. 완전히 같은 동작을 하는 코드를 requests로 작성했을 때 12.2초가 걸리는 것을 생각하면 엄청난 속도향상이라고 볼 수 있겠다.

그럼 이제 실전으로 들어가보자.
Asyncio로 작동되는 코드까지 다 완성해서 돌리고 있는데,
글이 너무 길어지는 바람에 실제 활용 설명은 2편에서...