---
title: <Python> Decorator가 뭐지? with Scope, Namespace
date: 2019/10/30
category: Python
comments: true
---
# Decorator
* Decorator : 오브젝트의 구조를 변경하지 않고 새 기능을 추가 할 수 있게 해 주는 디자인 패턴. Python에서는 @를 키워드로 사용한다.
Decorator의 간단한 예시와 실행결과를 보자.

```Python
def print_line(func): # Decorator가 될 함수
    def wrapper_mine():
        print('-'*30) # func의 사전작업이라 할 수 있다.
        func()
        print('#'*30) # func의 사후작업이라 할 수 있다.
    return wrapper_mine # wrapper_mine을 호출한 게 아니고, 함수 객체를 그냥 반환한 거다.
```
```Python
def my_function():
    print("my_function 실행")

m = print_line(my_function) # print_line에 직접 argument를 전달하여 실행해 보자
m()

#실행결과
------------------------------
my_function 실행
******************************
```
```Python
@print_line # decorator를 사용하자
def my_function2():
    print("my_function2 실행2")

my_function2()

#실행결과
------------------------------
my_function2 실행2
******************************
```

여기서 몇 가지 궁금한 점이 생긴다.

1. function(여기서는 wrapper_mine)을 value처럼 막 return 하네?
1. wrapper_mine이 어떻게 print_line의 argument에 접근할 수 있지?
 * 그게 그냥 된대~ 하고 사용해 왔지만 정확한 철학을 알고 싶다

일단 1번부터 알아 보자.
왜 함수를 return이 가능하죠?

* * *

## First class citizen
Wikipedia의 친절한 설명을 보자.
Python first class citizen 이라고 검색하면 나오는 포스트들 다 여기서 내용 가져온 거다.

{% blockquote Wikipedia https://en.wikipedia.org/wiki/First-class_citizen First-class citizen %}
In programming language design, a first-class citizen (also type, object, entity, or value) in a given programming language is an entity which supports all the operations generally available to other entities. These operations typically include being passed as an argument, returned from a function, modified, and assigned to a variable.
{% endblockquote %}

> First Class Citizen은 Argument로 넘기기, 함수에서 return되기, 조작되기, 변수에 할당되기와 같은, 다른 독립체들과 상호작용 할 수 있는 연산을 지원한다.

그리고 Python에서는 [함수도 first class object다.](https://dbader.org/blog/python-first-class-functions)
**"function을 value처럼 막 return 하네?"** 해결. 그럼 다음 문제, wrapper_mine이 어떻게 print_line의 argument에 접근할 수 있지?

## Namespace, Scope - LEGB
### [Namespace](https://docs.python.org/3/tutorial/classes.html#python-scopes-and-namespaces)
* Namespace? : 객체와 이름이 매핑된 공간. 대부분의 네임스페이스는 딕셔너리로 적용된다.
    * 네임스페이스의 예시
        * built-in된 이름들 : `abs()`같은 함수명이나, 예외명들.
        * 모듈 내의 전역 이름들 (global names in module)
        * 함수 내의 지역 이름들 (local names)
        * e.g) zzz.real과 ddd.real은 real이라는 attribute 이름은 같을지라도, 전혀 다른 네임스페이스에서 가져온 것이기 때문에 아무런 연관도 없다.
    * 네임스페이스의 수명주기(lifetime)
        * 각각 다른 시기에 생겨나며, 각각 다른 수명을 가짐
        * built-in name을 보유한 네임스페이스는 인터프리터가 시작할 때 생겨나고, 인터프리터가 꺼질 때까지 사라지지 않음.
        * 모듈의 글로벌 네임스페이스는 모듈의 정의가 읽혀질 때 생겨나며, 인터프리터가 꺼질 때까지 사라지지 않음.
        * 함수의 지역 네임스페이스는 함수가 호출될 때 생겨나고, 함수가 결과를 반환하거나 함수 내부에서 처리되지 않는 에러를 내보낼 때 사라진다.

### Scope
* Scope? : 네임스페이스가 '직접 접근'할 수 있는 구문 영역. '직접 접근'이라는 건, [unqualified reference](https://stackoverflow.com/questions/17403941/what-is-a-qualified-unqualified-name-in-python)(비-제한 참조라고 하면 좋을까?)로 네임스페이스 안의 이름을 찾을 수 있는 것을 말한다.
    * [unqualified reference?](https://stackoverflow.com/questions/17403941/what-is-a-qualified-unqualified-name-in-python) : `someclass.target` 처럼 '나 어디 있소'라고 `someclass.` 를 앞에 붙이지 않고, 바로 `target`으로 이름을 찾는 참조법.
* LEGB : Python Scope 탐색 규칙
    * Local : 함수 안에 정의된 이름 중 Global로 정의되지 않은 것
    * Enclosing-function : 함수를 내포하고 있는 함수(enclosing function)의 영역 안에 있는 것
    * Global (module) : 모듈 파일의 가장 상위 레벨에서 정의되었거나(어디 클래스나 함수 안에서 정의된 것 아니고) 함수 내부에서 `global`키워드로 실행된 것 
    * Built-in (Python) : Python에서 기본으로 정의하고 있는 것

아래 코드를 살펴보자.
```Python
x1 = -1 # Global
class spam:    
    x2 = -2 # class body
    def ham(self, bar):
        # enclosing function
        x3 = -3
        print(f'print global x1 : {x1}') # global에서 받아옴
        def egg():
            x4 = -4
            x1 = 'local x1'
            print(f'print local x1 : {x1}') # local에서 정의된 x1을 먼저 받아옴
            print(f'print class body x2 : {self.x2}') # class body에 정의된 객체를 가져오려면 self 키워드 필요.
            # 여기서 그냥 x2로 가져오려고 하면, local에도 없고 enclosing에도 없으니
            # global에서 찾게 된다 : print(f'print x2 : {x2}') 그리고 못 찾아서 에러를 낸다.
            print(f'print enclosing x3 : {x3}') # enclosing function 영역에서 받아옴
            print(f'print abs of x4 : {abs(x4)}') # Built-in에서 print, abs를 찾는다. local에서 x4를 찾는다.
            print(bar) # enclosing-function의 argument를 받는다.
        egg()

spam = spam()
spam.ham('foo')
```
```
출력결과
print global x1 : -1
print local x1 : local x1
print class body x2 : -2
print enclosing x3 : -3
print abs of x4 : 4
foo
```
Class body영역은 scope에서 enclosing-function도 아니고 global도 아닌 독특한 위치를 차지하고 있는데, [이 StackOverFlow 답변](https://stackoverflow.com/questions/291978/short-description-of-the-scoping-rules/23471004#23471004)을 참고하자.

다시 원점으로 돌아오면,
```Python
def print_line(func): # Decorator가 될 함수
    def wrapper_mine():
        print('-'*30) # func의 사전작업이라 할 수 있다.
        func()
        print('#'*30) # func의 사후작업이라 할 수 있다.
    return wrapper_mine # wrapper_mine을 호출한 게 아니고, 함수 객체를 그냥 반환한 거다.
```
**"wrapper_mine이 어떻게 print_line의 argument에 접근할 수 있지?"** 도 해결되었다.

* * * 

# 어디다 쓰지?
말 그대로, 이걸 어디다 쓸까? 간단한 예제를 살펴보자.
코드 출처 : https://khanrc.tistory.com/entry/decorator%EC%99%80-closure
```Python
def verbose(func): 
    def new_func(*args, **kwargs):
        print("Begin", func.__name__)
        func(*args, **kwargs)
        print("End", func.__name__)
    return new_func
```
함수 호출의 시작과 끝을 print 출력으로 알리는 함수다. 아무 함수나 들어올 수 있게 parameter가 설정되어 있다.

```Python
@verbose
def simple_sum(x,y):
    print(x+y)
    return x+y

simple_sum(1,2)
```
```
출력값
Begin simple_sum
3
End simple_sum
```
어떤 함수를 집어넣어도 호출 시에, 연산 끝났을 시에 함수 이름과 함께 알려주는 재미있는 Decorator다.

* * *

# Chaining Decorators in Python
[이 글을 보다보니](https://www.programiz.com/python-programming/decorator), 재미있는 예제를 찾을 수 있었다. Decorator가 중첩되어 있으면 어떤 것 부터 적용될까?
아래 코드를 보자.
```Python
def star(func):
    def inner(*args, **kwargs):
        print("*" * 30)
        func(*args, **kwargs)
        print("*" * 30)
    return inner

def percent(func):
    def inner(*args, **kwargs):
        print("%" * 30)
        func(*args, **kwargs)
        print("%" * 30)
    return inner

@star
@percent
def printer(msg):
    print(msg)
printer("Hello")
```
```
출력값
******************************
%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%
Hello
%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%
******************************
```
더 위쪽에 적혀진 Decorator부터 적용됨을 알 수 있다.


### Reference
1. https://dbader.org/blog/python-first-class-functions
1. https://docs.python.org/3/tutorial/classes.html
1. https://stackoverflow.com/questions/291978/short-description-of-the-scoping-rules
1. https://blog.mozilla.org/webdev/2011/01/31/python-scoping-understanding-legb/
1. https://khanrc.tistory.com/entry/decorator%EC%99%80-closure
1. https://www.programiz.com/python-programming/decorator