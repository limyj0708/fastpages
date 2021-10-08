---
title: <Docker> Mac에 Oracle DB 설치하기로 Docker 시작하기 
category: 학습_정리
layout: post
---

새로운 도구의 필요성은 언제나 갑자기 찾아온다.
"오너라, 오라클! 난 Docker 마스터다! 널 맥에 바로 설치해주마!"
같은 상황은 살면서 별로 일어나지 않는다.

그렇다고 구글링을 해서 나온 명령어들을 그저 복사&붙여넣기 하여 설치하기만 하면, 
응용도 안 되고 단지 시간을 쓴 것에 지나지 않게 된다.

단순한 따라하기를 넘어서, Docker로 Oracle DB를 Mac에 설치하는 과정을 통해 
Docker의 개념과 기본 명령어들을 공부해보자.
일단 설치부터 하고~ [Docker Desktop for mac](https://docs.docker.com/docker-for-mac/)


# Docker가 뭐요?
일단 [Nomad Coders의 영상](https://www.youtube.com/channel/UCUpJs89fSBXNolQGOYKn0YQ/about)을 본 다음에,
좋은 소개 자료들을 읽어보자. 
* [Docker 공식 문서 / Docker Overview](https://docs.docker.com/engine/docker-overview/)
* [Docker 공식 문서 / Container 개념 소개 페이지](https://www.docker.com/resources/what-Container)
* [초보를 위한 도커 안내서 - 도커란 무엇인가?](https://subicura.com/2017/01/19/docker-guide-for-beginners-1.html)

Docker Overview 페이지와 [Wikipedia](https://en.wikipedia.org/wiki/Docker_(software))의 설명을 요약하면, 아래와 같이 정리할 수 있지 않을까?

<span style="color:#AA0909"><b>'Container'라 불리는 단위로 어플리케이션을 묶고, 인프라로부터 분리하여, 인프라에 종속되지 않는 어플리케이션 실행과 빠른 배포를 가능하게 하는 플랫폼</b></span>

## Docker의 구조
### Docker Engine
다음은 Docker Engine의 구조이다. 서버-클라이언트 형식으로 되어 있다.<br>이 Docker Engine 위에서 Container가 돌아가고, 이런저런 관리를 하게 된다.

![Oops, Some network ants ate this pic!](https://drive.google.com/uc?id=1p6RtMxLnxHHcsjAGP3n8PFUKALgB7-Je "Docker Flow")

* [Daemon](#Daemon이-뭐지) 프로세스인 서버가 돌아가고 있다.

* [REST API](#Rest-api가-뭐지) : 프로그램들이 deamon과 통신하고 뭘 해야 할 지 명령할 때,
사용할 수 있는 인터페이스를 명시하는 REST API.

* CLI Client : [AWS CLI 처럼 HTTP API WRAPPER로서 기능함.](https://stackoverflow.com/questions/42641011/what-is-the-need-for-docker-daemon)
아래 설명을 잘 읽어 보자.
> When you use docker run command to start up a Container, your docker client will translate that command into http API call, sends it to docker daemon, Docker daemon then evaluates the request, talks to underlying os and provisions your Container.

### Docker Architecture
![Oops, Some network ants ate this pic!](https://drive.google.com/uc?id=10OeKD0B_zOS-Ivi5kvgsvjE18LWyCv3a "Docker Architecture")

<br>

* Client가 Daemon에게 요청을 보낸다.
    * Client와 Daemon은 같은 시스템에서 작동할 수도 있고, 
    * Client가 별도 서버의 Daemon에 접속할 수도 있다.
* Client와 Daemon은 [UNIX Socket](#Socket이-뭐지)(로컬에서 돌아갈 때)이나 네트워크 인터페이스를 통해, [REST API](#Rest-api가-뭐지)로 통신한다.

#### The Docker daemon
* Docker daemon(`dockerd`)는 Docker API의 요청을 받아서, Docker Object(image, Container, network, volume...)들을 관리한다. 
* Docker service들을 관리하기 위해 다른 daemon들과 통신할 수도 있음.   

#### The Docker client
* Docker client(`docker`) : Docker와 상호작용 하기 위한 가장 주요한 방법!
e.g : `docker run`명령어를 실행하면, client는 `dockerd`에 이 명령어를 보낸다. `docker`명령은 Docker API를 사용한다.
* 여러 개의 daemon과 통신할 수도 있다.

#### Docker registries
* Docker image들의 저장장소. Docker Hub라는 Public registy가 제공된다.
(image를 찾을 때, 찾는 장소는 Docker Hub가 기본값이다!) Private registry도 사용 가능.
* Docker Datacenter를 사용한다면, Docker Trusted Registy가 포함되어 있다.
(이 내용은 나중에 알아보자.)

## Docker objects : Container? image?
### Image
* Image는 Container를 만들기 위한 명령들이 들어 있는 읽기 전용 템플릿이다.
* Image를 만들기 위해서는 [Dockerfile](https://docs.docker.com/engine/reference/builder/)을 만들어야 한다.
    * Dockerfile은 image를 만들고 실행하는데 필요한 단계들을 정의하는 구문들을 담고 있는 파일이다.
    * Dockerfile의 각 명령들은 image의 각 ***layer***들을 생성한다. Dockerfile을 수정하고 다시 빌드하면, 바꾼 layer만 변경된다. 그래서 다른 가상화 기술에 비해 가볍고 작고 빠르다.
* Image는 상태(state)를 가지지 않으며, 변하지 않는다.(Container의 설계도가 상황에 따라 이리저리 변하면 곤란할 것이다.)
* Layer로 구성된 특성 덕분에, 특정 이미지를 약간 수정한 다른 이미지를 쉽게 만들 수 있다.
    * e.g. Ubuntu 이미지를 기반으로, Ubuntu에 Node.js를 설치한 다른 이미지 생성.

위의 설명을 시각화하면 아래처럼 된다.
![Oops, Some network ants ate this pic!](https://drive.google.com/uc?id=1TotuCQNuYkOj9e3_ITLi_R0DtrBWxCSk "Docker Layer")


### Container (is a runnable instance of image)
* 제목에 써 있는 대로, image의 인스턴스. image가 실행된 상태. 
* [Docker 공식 Container 소개 페이지에 따르면,](https://www.docker.com/resources/what-Container) 아래와 같은 컨셉으로 작동한다. VM은 비교를 위해 언급되어 있다.

![Oops, Some network ants ate this pic!](https://drive.google.com/uc?id=1iJFyDPDOoX7htyEYkazMAFMCVz_SHV7O "Docker Container")

> Containers
**Containers are an abstraction at the app layer that packages code and dependencies together.** Multiple Containers can run on the same machine and share the OS kernel with other Containers, each running as isolated processes in user space. Containers take up less space than VMs (Container images are typically tens of MBs in size), can handle more applications and require fewer VMs and Operating systems.

> VIRTUAL MACHINES
Virtual machines (VMs) are an abstraction of physical hardware turning one server into many servers. The hypervisor allows multiple VMs to run on a single machine. Each VM includes a full copy of an operating system, the application, necessary binaries and libraries - taking up tens of GBs. VMs can also be slow to boot.

**Container의 특징들**
* Docker API, CLI로 만들고(create) 시작하고(start) 멈추고(stop) 옮기고(move) 할 수 있다.
* 하나 혹은 여러 네트워크에 연결하고, 저장소를 추가하고, Container의 현재 상태를 기반으로 새 image도 만들 수 있다.
* 기본적으로는, Container는 다른 Container, 자신의 Host와 잘 분리되어 있으며, 이 격리 상태도 조정할 수 있다.
* Container는 (image) + (생성, 실행 시 받는 구성 옵션 값)에 의해 정의된다.(configuration option)
* Container가 삭제될 때에는, 영구 저장소에 저장되지 않은 상태 변경 값은 사라진다. 그렇다. 모든 변경 사항은 Container의 R/W Layer에 저장되며, 이 Layer는 Container가 삭제되면 같이 사라진다.

**`docker run` 명령어 예시로 알아보는 Container 생성과정**
`% docker run -i -t ubuntu /bin/bash`
위의 명령어를 실행하면, ubuntu Container를 만들고 실행해서 로컬 command-line 세션에 붙이고, Ubuntu의 Bash Shell을 실행한다. 어떤 일이 일어나는걸까?
1. `ubuntu` image가 로컬에 없으면, Docker가 image를 내가 설정해 둔 registry에서 pull한다. `docker pull ubuntu`를 직접 입력한 것처럼. (기본 registry는 Docker Hub)
1. Docker가 새 Container를 만든다. `docker Container create`를 직접 입력한 것처럼.
1. Docker가 읽고 쓰기가 가능한 파일시스템을 Container의 최종 레이어로써 할당한다.(R/W Layer) 이는 Container가 로컬 파일시스템에서 파일과 디렉토리를 만들고 수정할 수 있게 해 준다.
1. 네트워크 옵션을 하나도 주지 않았기 때문에, Docker가 Container를 기본 네트워크에 연결하기 위해 네트워크 인터페이스를 만든다. 이 과정은 IP 주소를 Container에 할당하는 과정이 포함된다. 자연스럽게, Container는 host의 네트워크 연결을 사용하여 외부 네트워크에 접속할 수 있게 된다.
1. Docker가 Container를 켠 후 `/bin/bash`를 실행한다. Container는 상호작용이 가능하게(interactively) 동작하고 있고, 터미널에 붙어 있기 때문에 (`-i`,`-t` 플래그를 넣어서), 키보드로 명령을 입력할 수 있고, 출력을 터미널로 받아볼 수 있다.
1. `/bin/bash`명령을 끄기 위해 `exit`를 입력하면, Container는 멈추지만 제거되지는 않는다. 다시 시작하거나 제거할 수 있다.

### Services
(이 내용은 당장 활용할 일이 없다. 이후 사용의 필요가 느껴지면 심도있게 알아보자.)
[(Swarm에 대해 소개하는 아주 좋은 글)](https://subicura.com/2017/02/25/container-orchestration-with-docker-swarm.html)
Service는 여러 개의 Docker daemon위에서 Container를 스케일링 할 수 있게 해준다.
이 여러 개의 Docker daemon들은 **Swarm**이라는, 여러 개의 *manager*, *worker*를 가지고 있는 단위로 뭉쳐서 작동한다.
Swarm의 각 멤버들은 Docker daemon이고, daemon들은 Docker API를 사용하여 통신한다.
Service는 원하는 상태를 정의할 수 있게 해 주는데, 특정 시간에 반드시 접근 가능해야 하는 Service 복제체의 숫자라던가 하는 것이다.
기본적으로, Service는 모든 worker node들에 부하 분산처리된다.(load-balanced) 사용자에게는 Docker service가 하나의 어플리케이션으로 보인다. Docker 1.12버전 이상부터 지원됨!

## 여기까지 읽었을 때의 궁금한 점들
### Daemon이 뭐지?
> https://en.wikipedia.org/wiki/Daemon_(computing)
> In multitasking computer operating systems, a daemon (/ˈdiːmən/ or /ˈdeɪmən/)[1] is a computer program that runs as a background process, rather than being under the direct control of an interactive user.
악마는 당신의 등 뒤에서 조용히 돌아가고 있다... :)
<br>

### Rest API가 뭐지?
**RE**presentational **S**tate **T**ransfer
이 영상 이상으로 잘 설명한 자료가 있을까?<br>내용이 방대하기 때문에, 추후 별도의 정리를 진행해야겠다.
<br>
> youtube: https://www.youtube.com/watch?v=RP_f5dMoHFc
<br>

### Socket이 뭐지?
**프로세스 간 데이터 교환을 위한, 소프트웨어로 작성된 통신 접속점**
아래는 Unix Socket(Unix Domain Socket)과 IP Socket에 대한 설명이다.
> https://serverfault.com/questions/124517/whats-the-difference-between-unix-socket-and-tcp-ip-socket
> A UNIX socket is an inter-process communication mechanism that allows bidirectional data exchange between processes running on the same machine.
> IP sockets (especially TCP/IP sockets) are a mechanism allowing communication between processes over the network. In some cases, you can use TCP/IP sockets to talk with processes running on the same computer (by using the loopback interface).
> UNIX domain sockets know that they’re executing on the same system, so they can avoid some checks and operations (like routing); which makes them faster and lighter than IP sockets. So if you plan to communicate with processes on the same host, this is a better option than IP sockets.

아래 글들도 읽어보도록 하자.
* [소켓과 포트](https://blog.naver.com/myca11/221389847130)
* [소켓 프로그래밍](https://recipes4dev.tistory.com/153)
* [소켓 프로그래밍 기초](https://12bme.tistory.com/228)
* [번외 : 서버,클라이언트,호스트](https://blog.naver.com/myca11/221369799273)
<br>

# 그래서 선생, 이 명령어가 도대체 뭐요?
## 어떤 명령어들을 사용하였는가
Oracle DB 설치와 실행을 위해 어떤 명령어들을 사용하였는가?
(클릭하면 각 명령어 설명 공식문서로 이동함)

* [`docker search oracle`](https://docs.docker.com/engine/reference/commandline/search/) : 기본 registry인 docker hub에서, oracle이라는 단어를 포함한 image를 찾는다. 추천수 순으로 정렬되어 나온다. 그런데 솔직히 [Docker Hub](https://hub.docker.com)에서 검색하는 것이 더 좋다. 

* [`docker pull oracleinanutshell/oracle-xe-11g`](https://docs.docker.com/engine/reference/commandline/pull/) : registry에서 image나 repository를 가져온다. 여기서는 추천수가 가장 높았던 *oracleinanutshell/oracle-xe-11g* 라는 image를 가져왔다. [Docker Hub : oracleinanutshell/oracle-xe-11g](https://hub.docker.com/r/oracleinanutshell/oracle-xe-11g) 설명을 보니 Ubuntu 18.04 LTS에 Oracle xe 11g를 올린 image다.

* [`docker run --name oracle-xe-11g -d -p 8080:8080 -p 1521:1521 oracleinanutshell/oracle-xe-11g`](https://docs.docker.com/engine/reference/commandline/run/) : `docker run [OPTIONS] image [COMMAND] [ARG...]` 이런 구조로 되어있다. Container를 **생성하여 실행한다.** 여기서 사용한 옵션값부터 살펴보자.
    * \-\-name : Container의 이름을 설정한다. 중복 이름은 허용하지 않음.
    * \-d : Container를 백그라운드에서 실행하고, Container ID를 출력한다.
    [Background와 Foreground의 차이](https://www.tecmint.com/run-docker-container-in-background-detached-mode/)
    * \-p : 특정 범위의 포트, 혹은 포트 하나를 host에 publish한다. 여기서는 hostPort:ContainerPort 구조로 사용했다. 즉, Container의 8080포트를 Host의 8080포트에 매핑하는 방화벽 규칙을 만든다. (하필이면 8080, 1521인 이유는, Oracle Listener가 사용하는 포트가 1521이고, XML DB가 8080포트를 사용하기 때문이다.)<br> [`docker port oracle-xe-11g`](https://docs.docker.com/engine/reference/commandline/port/)로 연결된 포트를 확인해보면, 다음과 같이 뜬다.
    ```
    1521/tcp -> 0.0.0.0:1521
    8080/tcp -> 0.0.0.0:8080
    ```
    포트들이 [0.0.0.0 (all IPv4 addresses on the local machine)](https://superuser.com/questions/949428/whats-the-difference-between-127-0-0-1-and-0-0-0-0)의 동일 포트에 매핑되었다. 이제 Container가 **생성**된 후 **실행**되었는데, 그럼 다음엔 뭘 해야 할까?
        * [참고 공식 문서 : Container networking](https://docs.docker.com/config/Containers/Container-networking/#published-ports)
        * [참고 공식 문서 : Docker run reference](https://docs.docker.com/engine/reference/run/#expose-incoming-ports)

* [`docker exec -it oracle-xe-11g bash`](https://docs.docker.com/engine/reference/commandline/exec/) : 외부에서, 실행 중인 Container 안의 명령을 실행한다. oracle-xe-11g Container는 Ubuntu 18.04위에서 Oracle DB를 구동하는 구조이기 때문에, bash shell을 열라는 명령어를 보내 보았다.(bash) 옵션 -it는 뭘까?
    * \-i : 키보드, 화면을 통해 STDIN, STDOUT(표준입력, 표준출력)[[설명]](https://github.com/kennyyu/bootcamp-unix/wiki/stdin,-stdout,-stderr,-and-pipes)[[설명의 번역]](https://velog.io/@jakeseo_me/유닉스의-stdin-stdout-stderr-그리고-pipes에-대해-알아보자)을 열고 유지한다. 명령어 입력, 결과 출력을 위해서 넣어주어야 하는 값이다.
    * \-t : pseudo-TTY를 할당한다. 터미널 환경을 에뮬레이션 해 주는데, 이 옵션을 입력하지 않으면 터미널 환경이 보이지 않는다.<br>(i와 t값을 넣지 않으면 각각 어떻게 되는가? 는 [이 블로그](https://m.blog.naver.com/alice_k106/220340499760)를 참고하자.)
    보기만 해서는 잘 기억나지 않을테니, 직접 해보자.
    ```
    % docker exec oracle-xe-11g bash
    %
    ```
    하나도 안 쓰면 바로 종료된다. 입출력도, tty도 활성화되지 않았으니 당연한 결과.
    ```
    % docker exec -t oracle-xe-11g bash
    root@a702ae5d7f10:/# sqlplus

    ```
    \-t만 쓰면, 첫 번째 명령어 입력까지는 가능하나 그 후의 결과가 출력되지 않는다. STDIN을 열지 않았으니 당연한 결과.
    ```
    % docker exec -i oracle-xe-11g bash
    sqlplus
    bash: line 1: sqlplus: command not found
    ls
    bin
    boot
    dev
    ...(이하생략)
    ```
    \-i만 쓰면, 터미널 환경이 조성되지 않는다. 그냥 명령어를 입력하면 sqlplus는 command not found 에러가 뜨고, ls는 제대로 출력이 되긴 한다. 응용 프로그램(sqlplus) 실행은 안 되고, 기본 shell command는 제대로 실행되는 건 terminal의 부재 때문이 아닌가 추측해본다. 자세한 이유는 다음에 알아보기로 하자.<br>
    ```
    % docker exec -it oracle-xe-11g bash

    root@a702ae5d7f10:/# sqlplus

    SQL*Plus: Release 11.2.0.2.0 Production on Sun Oct 27 02:33:24 2019

    Copyright (c) 1982, 2011, Oracle.  All rights reserved.

    Enter user-name: system
    Enter password:

    Connected to:
    Oracle Database 11g Express Edition Release 11.2.0.2.0 - 64bit Production

    SQL>
    ```
    제대로 모두 입력하여 sqlplus에 로그인까지 진행하면 이렇게 된다.

도대체 pseudo-TTY가 뭐지? 에 대한 글은 다음 링크를 참고하자.
* [Bash Shell에 대한 엄청난 gitbook : TTY](https://mug896.github.io/bash-shell/tty.html)
* [콘솔? 터미널? 쉘?](https://kldp.org/node/134965)

\- oracleinanutshell/oracle-xe-11g image로부터 Container를 실행하였고, Container에서 sqlplus를 실행해서 로그인도 해 봤다.
\- 설정을 다 했다. 그런데 컴퓨터 재부팅을 하거나, Docker를 종료했다가 Container를 또 실행하고 싶으면 어떻게 해야 할까?

\-[`docker start oracle-xe-11g`](https://docs.docker.com/engine/reference/commandline/start/) : 하나, 혹은 여러 개의 멈춘 Container를 실행한다. 여기서는 `docker run`으로 생성한 'oracle-xe-11g' 라는 이름을 붙인 Container를 실행한다.

## 뭐 하나 잊어버린 것 같은데..? : Volume
그런데 여기서 빼먹은 것이 하나 있다. 위에 위험한 설명이 하나 있었던 것 같은데?
> Container가 삭제될 때에는, 영구 저장소에 저장되지 않은 상태 변경 값은 사라진다. 그렇다. 모든 변경 사항은 Container의 R/W Layer에 저장되며, 이 Layer는 Container가 삭제되면 같이 사라진다.

Container를 영영 없애지 않을 생각이거나, 한 Container의 데이터를 다른 Container와 공유하지 않을 생각이라면 지금까지 입력한 명령어들만으로도 충분하다. 하지만 아니라면? Volume을 사용, 데이터를 Host에 저장하여 안전하게 유지, 공유해보자. 아래 두 링크를 꼭 읽어보자. 초반 부분만 읽어봐도 된다.
* [Docker 공식 페이지 / Use volumes](https://docs.docker.com/storage/volumes/)
* [Docker 공식 페이지 / About storage drivers](https://docs.docker.com/storage/storagedriver/)
> https://docs.docker.com/storage/volumes/
> Volumes are the preferred mechanism for persisting data generated by and used by Docker Containers.

> https://docs.docker.com/storage/storagedriver/#Container-and-layers
> The major difference between a Container and an image is the top writable layer. All writes to the Container that add new or modify existing data are stored in this writable layer. When the Container is deleted, the writable layer is also deleted. The underlying image remains unchanged.
> Because each Container has its own writable Container layer, and all changes are stored in this Container layer, multiple Containers can share access to the same underlying image and yet have their own data state.

그럼, oracle-xe-11g Container 안의 어떤 폴더를 Host의 폴더와 연결해 주어야 할까?
Container 안에서 oracle이란 이름을 가진 폴더를 검색해보자.
```
$ find / -name oracle -type d  # 전체 폴더에서 oracle 이름을 가진 폴더 검색
/u01/app/oracle
```
오호라, `/u01/app/oracle`를 연결하면 될 것 같다.

[`docker run`]((https://docs.docker.com/engine/reference/commandline/run/))문서에 따르면, Volume을 할당하기 위해 다음과 같은 옵션이 필요하다.
`-v [Host directory]:[Container directory]`
이 옵션을 추가하여, Container를 다시 생성해 보자.

`docker run --name oracle-xe-11g -d -p 8080:8080 -p 1521:1521 -v /Users/youngjinlim/Coding/BigData_Study/SQL/Docker_volume:/u01/app/oracle oracleinanutshell/oracle-xe-11g`

과연 Volume이 잘 Mount 되었을까?
```
% docker inspect --format='{{.Mounts}}' oracle-xe-11g
[{bind  /Users/youngjinlim/Coding/BigData_Study/SQL/Docker_volume /u01/app/oracle   true rprivate}]
```
오, 잘 연결된 것 같다. 그런데... DB에 연결할 수가 없었다! 왜지?
```
root@141d12eb18ac:/u01/app/oracle# ls -l
total 0
```
어엉? Container 쪽 폴더가 텅 비어버렸다! 아무래도 Host쪽의 폴더로 덮어씌워진 것 같다.
아.. 너무 고통스럽다..
이에 대한 원인과 해결방법은 이 블로그에서 찾을 수 있었다.
* [docker volume의 사용방법과 차이점](https://darkrasid.github.io/docker/container/volume/2017/05/10/docker-volumes.html) :
<span style="color:red"><b>!!주의!!<b></span> 이 블로그 포스팅에선 `docker create volume volume_name` 라고 썼는데,<br>이러면 안된다. 왜냐면..

> https://docs.docker.com/engine/reference/commandline/create/
> Description : Create a new Container
> Usage : docker create [OPTIONS] image [COMMAND] [ARG...]
docker create는 Container를 만들 때 쓰는 명령어이기 때문이다.
[docker volume](https://docs.docker.com/engine/reference/commandline/volume/)을 써야 한다.

Host쪽 폴더가 텅 비어있을 때, Container쪽 폴더를 남기려면 아래와 같은 방법을 사용해야 한다.
```
docker volume create volume_name
docker run --name oracle-xe-11g -d -v volume_name:/Container/some/where ...(이하생략)
```
그럼 실행해 보자.
```
% docker volume create oracle-xe-11g_study    # Volume 생성
% docker volume ls    # 잘 생성되었는지 확인
DRIVER              VOLUME NAME
local               oracle-xe-11g_study
% docker run --name oracle-xe-11g_study -d -v oracle-xe-11g_study:/u01/app/oracle -p 8080:8080 -p 1521:1521 oracleinanutshell/oracle-xe-11g
6a7d5728c9084c9f2c25931a8a7bf1120594776380d5cd8e17d6d15b48604eb6    # 새 Container 생성
% docker exec -it oracle-xe-11g_study bash    # /u01/app/oracle 폴더 무사한지 확인하러 들어감
root@6a7d5728c908:/# cd /u01/app/oracle
root@6a7d5728c908:/u01/app/oracle# ls -l    # 결과를 보면 무사함을 알 수 있음
total 24
drwxr-x--- 4 oracle dba  4096 Oct 27 03:39 admin
drwxrwxr-x 4 oracle dba  4096 Oct 27 03:39 diag
drwxr-x--- 3 oracle dba  4096 Oct 27 03:39 fast_recovery_area
drwxr-x--- 3 oracle dba  4096 Oct 27 03:39 oradata
drwxr-xr-x 3 oracle dba  4096 Oct 27 03:39 oradiag_oracle
drwxr-xr-x 3 root   root 4096 Oct 27 03:39 product
root@6a7d5728c908:/u01/app/oracle# exit
exit
% docker inspect --format='{{.Mounts}}' oracle-xe-11g_study    # Volume 잘 연결 되었는지 확인
[{volume oracle-xe-11g_study /var/lib/docker/volumes/oracle-xe-11g_study/_data /u01/app/oracle local z true }]
```
연결도 잘 됐고, Container 쪽 폴더도 무사하다.
'CUSTOMERS' 라는 이름의 테이블을 추가한 후, Container를 새로 생성해서 같은 Volume에 연결했을 경우, 새로 생성한 Container에서도 CUSTOMERS 테이블이 잘 보이는지 확인해 보자.

![Oops, Some network ants ate this pic!](https://drive.google.com/uc?id=1P4k-ysg5DS9Rnb69GKno9BlBpLEdzoeq "Add one table")

```
% docker run --name oracle-xe-11g_volumetest -d -v oracle-xe-11g_study:/u01/app/oracle -p 8080:8080 -p 1521:1521 oracleinanutshell/oracle-xe-11g
08ba1459d196d6094b7adc2232d843e430574551cb1ba4c823fae7f85aa8fe36
youngjinlim@Youngui-MacBookPro ~ % docker ps
...중략  NAMES
        oracle-xe-11g_volumetest
```
새로 생성한 Container 하나만 실행 중이다.(oracle-xe-11g_volumetest)
이제 추가했던 테이블이 그대로 있는지 확인해 보자.

![Oops, Some network ants ate this pic!](https://drive.google.com/uc?id=11goqVWONxi2uduKTx1mUzgBK2MuXuHLc "table check in new container")

잘 있다.

일단 Mac에서 Oracle DB를 사용하기 위한 여정은 여기서 끝이다.
Docker 마스터의 길은 멀고도 험하느니.

* * *

# 이번에 알아본 것
* Docker의 기본적인 개념
* Docker를 개념을 알아보다가 궁금해진 것들
    * 궁금해진 것들 중 너무 큰 주제들이 많았는데, 별도로 정리를 할 필요가 있어 보인다.
* Oracle Database 11g 설치 중 사용한 명령어들의 의미
    * Docker 공식문서가 최고다. 공식문서를 보시오