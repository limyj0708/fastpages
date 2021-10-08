---
title: Asynchronous, Synchronous, Blocking, Non-Blocking
date: 2019/11/11
category: 학습_정리
comments: true
---

많은 자료의 산 중에서, 가장 알기 쉽고 직관적으로 설명한 자료는
https://stackoverflow.com/questions/2625493/asynchronous-vs-non-blocking
이 질문의 세 번째 답변이라는 결론을 내렸다.
이 답변의 번역 + 보충 설명을 위한 다른 자료들 + 사족을 섞어서 글을 적어보았다.

* synchronous / asynchronous : 두 모듈 사이의 관계에 대한 표현
* blocking / non-blocking : 모듈 하나의 상태에 대한 표현

예를 들어,
* 모듈 X : 나
* 모튤 Y : 서점
* X가 Y에게 질문 : C++ primer 책 있나요?

#### 1. Blocking
* Y가 X에게 답하기 전까지, X는 기다린다. X는 Blocking 상태에 빠진 것이다. 

#### 2. Non-Blocking
* Y가 X에게 답하기 전에, X는 다른 일을 할 수 있다.
* X가 2분마다 Y가 일을 끝냈는지 확인할까?(Synchronous라면 이렇게 될 것 같다.) 아니면 Y가 다 됐다고 부르면 확인할까?(Asynchronous라면 이렇게 될 것 같다.) 모른다.(= 상관이 없다.)
* 우리가 아는 건 X가 Y가 일을 끝내기 전에 다른 일을 할 수 있다는 것 뿐이다. X는 Non-Blocking이다.

#### 3. Synchronous
* Y가 X에게 답하기 전에는, X는 다른 일을 진행하지 않는다 - 라고 설명하고 있는데, 이러면 Blocking과 정의가 같다. 좀 다르게 생각해 보자.
* Synchronous에는 중요한 두 가지 키워드가 있다.
    * [`작업의 순서를 맞추는 것`](https://en.wikipedia.org/wiki/Synchronization_(computer_science)#Thread_or_process_synchronization). 왜 순서를 맞추냐고? 여러 작업이 동시에(Concurrently) Critical section에 진입하는것을 막기 위해서이기도 하고, 특정 순서에 맞게 작업들을 실행해야 할 필요가 있기(표를 사지도 않고 비행기에 탈 수는 없다) 때문이기도 하다. 작업 순서를 맞출 때 Blocking으로 처리하면 편하기 때문에 Blocking의 개념이 섞여서 등장하는 것 뿐이다.
        * 작업 순서의 관점에서 설명하는 글 두 개
            * https://jins-dev.tistory.com/entry/동기Synchronous-작업과-비동기Asynchronous-작업-그리고-블락Blocking-과-넌블락NonBlocking-의-개념 (이 글의 경우 blocking 부분은 보면 더 헷갈리니 위에만 보자.)
            * https://medium.com/from-the-scratch/wtf-is-synchronous-and-asynchronous-1a75afd039df
    * Caller가 Callee의 완료 상태를 확인하는 것. Callee의 완료 여부가 Caller의 다음 작업에 영향을 미치기 때문으로, `작업의 순서를 맞추는 것`의 하위 개념이다.
        * [좋은 예시](https://stackoverflow.com/a/30707667/11925058) : 상사가 와서 어떤 일을 처리하라고 말한다. 그리고 내 등 뒤에서 시체를 노리는 독수리마냥 나를 쳐다보고 있다. "자네가 일을 다 끝낼 때까지 여기서 기다릴 걸세."
        * 상태 확인의 관점에서 설명하는 글 : https://homoefficio.github.io/2017/02/19/Blocking-NonBlocking-Synchronous-Asynchronous/
* 이 X,Y 예에서는, Synchronous는 **X가 Y에게 책 찾았냐고 물어보고,**(caller가 callee의 상태 확인) Y가 책이 있는지 없는지 X에게 알려 준 이후에야, X가 Y에게 "그래서 이 책 얼마죠?" 라고 물어보던가, "그 책 주문 좀 해주세요" 라고 요청할 수 있는 상황인 것이다.(작업 순서) 책이 있는지 확인한 다음에 가격을 물어보던지 주문을 요청하던지 할 수 있으니까. X가 Y가 책 찾는 동안 다른 무언가를 하는 건 상관이 없다. 서점 앞에서 줄넘기를 할 수도 있지 않은가? 책에 관련된 다음 일을 못하는 것 뿐.
* 이렇게 봐도 상당히 헷갈리기 때문에, 코드를 보자. 아래의 코드는 *Synchronous & Non-Blocking* 한 간단한 코드이다.
 ```Python
# thread X
while (true)
{
    msg = recv(Y, NON_BLOCKING_FLAG);
    if (msg is not empty)
    {
        break;
    }
    # 이 루프 안에서 다른 작업을 할 수 있다.
    sleep(2000); // 2 sec
}

# thread Y
# prepare the book for X
send(X, book);
 ```
 * **X가 2초마다 Y가 답을 줬는지 아닌지 확인한다.** 
 * Y가 결과를 반환하기 위해 준비 중이어도, X의 while 루프는 계속 돌아가고 그 안에서 다른 작업을 진행할 수 있다. **그래서 Non-Blocking이다.**
 * **하지만, while문을 빠져나가서 다른 작업을 할 수는 없다. 그래서 Synchronous다.**
 * 코드의 예시까지 보면, X가 서점을 떠나지 못한다고 해석할 수 있다. Y가 책을 찾아줘서 서점과 관련된 일을 마치기 전에는, 서점을 떠나서 다른 걸 할 수 없다. 서점 앞에서 줄넘기는 가능해도.(while문 안에서 뭔가 다른 작업)
 * Blocking이었다면, X는 아무것도 못하고 기다려야 했겠지만.

#### 4. Asychronous
* Y가 X에게 답하기 전에, X는 다른 곳에 가서 다른 일을 할 수 있다. **X는 Y가 부르기 전까지 돌아오지 않는다.** 이 때 X와 Y는 Asychronous 하다고 말한다.
* 여기서도 Synchronous와 같은 두 가지 키워드로 살펴보자.
    * 작업의 순서가 보장되지 않음 : [Asynchronous](https://en.wikipedia.org/wiki/Asynchronous_system)는 엄밀히 말하면, 작업들이 공통적으로 사용하는 global clock이 없고, 신호나 메세지의 도착 시간이 작업의 신뢰성에 영향을 미치지 않음을 뜻한다. 즉 [작업의 순서가 보장되지 않는다.](https://jins-dev.tistory.com/entry/동기Synchronous-작업과-비동기Asynchronous-작업-그리고-블락Blocking-과-넌블락NonBlocking-의-개념)(A,B,C 순서로 실행되었으나 [완료도 A,B,C 순서일 것이라 보장할 수 없음](https://medium.com/from-the-scratch/wtf-is-synchronous-and-asynchronous-1a75afd039df))
        * **신호나 메세지의 도착 시간이 작업의 신뢰성에 영향을 미치지 않음**, 즉, 각 작업이 서로 연관되지 않아서 분리될 수 있으며, 작업 지연시간이 큰 경우에 잘 활용될 수 있다. (DB 접근, Http 요청, File I/O 등)
    * Callee가 자신의 완료 상태를 확인하며, callback으로 Caller에게 자신의 완료를 알림
        * 근데 사실 완료 통보를 해도 되고 안해도 된다. [완료 통보가 caller에게 의미가 있느냐 없느냐의 문제이다.](https://okky.kr/article/649451)(내가 한 질문이다 :D)
        * [좋은 예시](https://stackoverflow.com/a/30707667/11925058) : 상사가 와서 어떤 일을 처리하라고 말한다. 그리고 다른 일 하러 가버림. 일을 다 끝내면, 나는 상사에게 "나 다함!" 이라고 말한다.
* Y에게 책 있냐고 물어본 후에 X가 카페에 가서 커피를 마시기 시작했지만, 책 찾기보다 커피 마시기가 더 빨리 끝날 수도 있다. 이 둘은 완전히 별개의 작업이며, 작업의 순서가 보장되지 않는다.(Asynchronous & Non-Blocking이라면.) 그리고 X는 Y가 X를 부를 때 서점으로 돌아간다.

각 2개씩의 개념이 있으니, 총 4개의 조합이 나올 것이다.
* Synchronous - Blocking
* Asynchronous - Blocking
* Synchronous - Non-Blocking 
* Asynchronous - Non-Blocking
이 조합들에 대해서는 [여기를](https://homoefficio.github.io/2017/02/19/Blocking-NonBlocking-Synchronous-Asynchronous/) 참고하자. 아래 사진이 핵심인데, 출처의 글에서 가져온 사진이다.

![Oops, Some network ants ate this pic!](https://drive.google.com/uc?id=1SygnLUtk9dxVWnycUvt7HHDSwaqO7ARS "sync-async-block-nonblock_cube")
* 출처 : https://homoefficio.github.io/2017/02/19/Blocking-NonBlocking-Synchronous-Asynchronous/

* * *

* 번외 내용 : 정리하다 보니 핵심 개념을 직관적으로 알기에는 너무 응용에 가깝다고 생각되었던 내용. 지우기는 아까워서 넘겨두었다.
    * [Blocking I/O]((http://www.masterraghu.com/subjects/np/introduction/unix_network_programming_v1.3/ch06lev1sec2.html)) : application이 kernal에 I/O 해줘~ 라고 system call을 날린다. kernal이 I/O를 수행하는 동안, application은 아무것도 못 하고 기다린다. I/O가 완료되면 call에 대한 return값으로 원하던 데이터를 받는다.
    ![Oops, Some network ants ate this pic!](https://drive.google.com/uc?id=1xKPthKaOEic-OzdqwBYvl25Rv2wdyd5O "Blocking IO")

    * [Non-Blocking I/O]((http://www.masterraghu.com/subjects/np/introduction/unix_network_programming_v1.3/ch06lev1sec2.html)) : application이 kernal에 I/O 해줘~ 라고 system call을 날린다. 그림의 recvfrom 함수는, 바로 결과를 return 하는데, 아직 I/O가 완료되지 않았으므로 에러인 EWOULDBLOCK을 return한다. 프로세스는 계속 recvfrom을 call 하게 되고, 데이터가 완료되었으면 그 때 데이터가 return된다. 이렇게 계속 요청하는 걸 *polling* 이라 한다.
    ![Oops, Some network ants ate this pic!](https://drive.google.com/uc?id=1xv9zW0kswg-VxLqFqaVODDmUCYwGWkcE "Non-Blocking IO")
        * [call에 대한 return을 바로 받아서, application이 제어권을 넘겨받고 다른 일을 진행할 수 있는 것이 중요하다.](https://homoefficio.github.io/2017/02/19/Blocking-NonBlocking-Synchronous-Asynchronous/) Blocking I/O와는 정 반대로.
    * [Non-Blocking Algorithm](https://en.wikipedia.org/wiki/Non-blocking_algorithm) : 어떤 쓰레드의 실패(failure)나 멈춤(suspension)이 다른 쓰레드에 영향을 미치지 않게 하는 알고리즘. 몇몇 상황에서는 이런 알고리즘이 전통적인 Blocking 적용([Lock](https://en.wikipedia.org/wiki/Lock_(computer_science)))의 유용한 대안이 된다.


#### 마치며
정확한 용어의 정의를 알아보려고 했다. 하지만 일반적으로는 아래의 대략적인 의미로 사용되는 듯하다. 나는 정확하게 쓰도록 노력해야겠다.
* Asynchronous Programming(비동기 프로그래밍): 하나의 요청을 시작한 후, 완료를 기다리지 않고 제어권을 다음 요청으로 넘기는 방식.(Non-Blocking의 의미를 포함)
* Synchronous Programming(동기 프로그래밍): 하나의 요청이 처리되는 동안 다른 요청이 처리되지 못하는 방식. 전 요청이 완료되어야 다음 요청 처리가 가능함.(Blocking의 의미를 포함) [참고](http://www.nextree.co.kr/p7292/)


#### References
1. https://en.wikipedia.org/wiki/Synchronization_(computer_science)#Thread_or_process_synchronization
1. https://en.wikipedia.org/wiki/Synchronous_circuit
1. https://en.wikipedia.org/wiki/Asynchronous_system
1. https://jins-dev.tistory.com/entry/동기Synchronous-작업과-비동기Asynchronous-작업-그리고-블락Blocking-과-넌블락NonBlocking-의-개념
1. https://medium.com/from-the-scratch/wtf-is-synchronous-and-asynchronous-1a75afd039df
1. https://homoefficio.github.io/2017/02/19/Blocking-NonBlocking-Synchronous-Asynchronous/
1. https://ozt88.tistory.com/20
1. http://www.masterraghu.com/subjects/np/introduction/unix_network_programming_v1.3/ch06lev1sec2.html
1. https://en.wikipedia.org/wiki/Blocking_(computing)
1. https://developer.ibm.com/articles/l-async/