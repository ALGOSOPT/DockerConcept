# Docker Machine
---
---

## 4.1 Docker machine을 사용하는 이유

Docker Container 환경에서 application을 개발해야 하는 개발자가 가장 먼저 마추지는
난관은 Docker 환경을 어디서 마련하지? 임.

조직에서 On-premise 환경의 server를 제공하거나 개인 서버를 소유하고 있거나
AWS와 같은 cloud service를 이용할 수 있다면 운이 좋은 편.

그러나 앞서 언급한 것조차 사용할 수 없다면 일반적으로 Virtual Box나 VMware 같은
가상 머신을 이용해 개인 PC에 linux를 설치 한 후 Docker로 개발함.


그러나 PaaS처럼 Multi host 환경의 Application을 개발한다면 virtual box같은 가상
환경을 사용하는 데도 한계가 있음. 일반적으로 적어도 3개 이상의 docker engine을 
사용해야 multi host환경에서 제대로 개발할 수 있는데, 가상 머신을
여러개 생성하는 것은 번거로울 뿐만 아니라 PC에도 많은 부담을 줌.


```
(참고) virtual box 같은 가상화 도구에서 가상 machine을 복제해서 사용하는
것은 바람직하지 않은. 게다가 일부 PaaS 솔우션은 MAC주소가 겹치면
오류가 발생하기도 함.
```

![그림4.1](https://github.com/ALGOSOPT/DockerConcept/blob/master/picture/4.1.jpg)

이러한 불편함을 해소하기 위해 Docker Machine은 window나 mac os에서 손쉽게
가상 machine을 생성하고 삭제하는 기능을 제공함. 가상화 도구로 직접 가장 머신을 생성해
리눅스를 설치할 필요도 없을뿐더러 Docker를 일일이 설치하지 않아도 됨.
Docker Machine을 이용하면 swarm이나 swarm clurster 같은 multi host의 개발환경도
손쉽게 생성하고 삭제할 수 있음.

Docker Machine은 가상 머신을 관리하는 기능 뿐 아니라 각종 cloud service를 이용해
새로운 docker server를 제공하는 기능도 제공함.
AWS, Digital Ocean 등의 cloud 환경에서 docker를 사용하고 있으며,
Docker server를 동적으로 늘려야 하는 개발 환경이 필요하다면 docker machine이 좋은
해결책이 될 수 있음. Docker Machine에 Cloud service 계정을 인증한 뒤 Docker Server
를 생성하면 Docker Machine은 Cloud에 instance를 자동으로 생성해 Docker를 설치함.
가상 머신이나 클라우드 서비스와 별개로 Onpremise 환경의 개인 서버도 Docker Machine에 
등록해 사용할 수 있음.


이렇게 각종 방법으로 등록된 Docker Server는 Docker Machine에서 관리하고 사용할 수 있음.
이전의 2.5.3.1 절 Docker Daemon 제어하기에서 Docker Daemon에 Remote API를 위한 port를
추가하고 DOCKER_HOST 변수를 설정해 Docker Client에서 원격으로 Docker Daemon에 접근하는
방법을 설명했음. 이와 비슷하게 Docker Machine도 등록된 서버를 하나의 termina에서 제어
하며 docker ps, run과 같은 일반적인 명령어도 사용할 수 있음. 즉, Docker Machine은 각종
환경에서 Docker server를 생성, 관리하고 사용하는 도구임.


```
docker machine은 window, MAC OS에서 설치한 Docker ToolBOX 및
Docker for windows, Docker for MAC에 포함되어 있으므로 이 중 하나를
설치했다면 별도로 docker machine을 설치할 필요가 없음.
이어지는 예제에서는 Docker ToolBox를 사용해 docker machine을 사용하는
방법을 설명.
```

---
## 4.2 Docker Machine 사용

Docker Machine은 윈도우의 git bash terminal, 명령 프롬프트 등과 맥 OS의 터미널에서
사용할 수 있음. 그러나 PowerShell이나 명령 프롬프트, terminal 등에서는
Docker Machine의 명령어가 의도치 않게 동작하는 경우가 있으므로 가능하면
Docker Quickstart Terminal이라는 shell 환경을 사용하는 것을 권장
Docker Quickstart Terminal은 Docker Toolbox를 설치하면 자동으로 생성됨.


![그림4.2](https://github.com/ALGOSOPT/DockerConcept/blob/master/picture/4.2.jpg)

Quickstart Terminal을 실행하면 다음과 같은 출력 결과를 확인할 수 있는데, 
이 것은 기본적으로 사용하도록 설정된 default라는 이름의 가상 머신을 생성하는 과정

정상적으로 docker machine의 초기화 작업이 완료되었다면 다음과 같은 출력을 확인할 수 있음.

![그림4.3](https://github.com/ALGOSOPT/DockerConcept/blob/master/picture/4.3.jpg)

docker engine과 마찬가지로 version 명령어로 docker machine이 정상적으로 설치되었는지 확인함.
docker machine의 명령어는 docker-machine으로 시작함.

```
# docker-machine version
```

---

### 4.2.1 Docker Machine 시작

가장 먼저 할 일은 docker machine에 등록된 docker server의 목록을 확인하는 것.
docker-machine ls 명령어는 docker machine에 등록된 docker server를 출력함.
docker machine을 처음 사용하려면 다음과 같이 default docker server 하나만 출력됨.

```
$ docker-machine ls
```

default라는 가상 머신이 생성되어 있음을 알 수 있음. 생성된 가상 machine의 driver는 
virtual box이고, docker server에 할당된 내부 IP가 192.18.99.100이므로
tcp://192.168.99.100:2376으로 해당 docker server에 Remote API로 접근할 수 있음.
Docker Machine은 사용 가능한 최신 버전의 Docker Engine을 가상 machine에 자동으로
설치함

```
Remote API를 사용할 때 Docker Engine에 보안을 적용하지 않으면 2375번 port를,
HTTPS로 보안을 적용했으면 2376번 port를 일반적으로 이용함.
이 port 번호는 반드시 지켜야 하는 것은 아니지만 대부분의 docker community에서
암묵적으로 지키는 규칙
```

docker machine은 등록된 docker server의 하나를 선택해 제어하는 방식으로 동작.
예를 들어, docker ps 같은 명령어를 docker client와 동일하게 사용할 수는 있지만
이 것은 docker machine이 선택한 docker server에서 실행됨.
선택된 docker server는 docker-machine ls 명령어오 출력된 ACTIVE 항목에서
값이 별표(* )로 표시됨. 위 예제에서는 default라는 가상 머신이 기본적으로
선택되어 있으므로 docker로 시작하는 각종 명령어는 default 가상 머신을 제어해야함.

현재 선택된 docker를 확인하려면 docker-machine activec 명령어를 사용함

```
# docker-machine active
```

선택한 docker server를 사용하는 방법은 docker client와 같음. 
예를 들어, 다음과 같은 docker run 명령어오 nginx server container를 생성할 수 있음.

```
# docker run -d -p 8080:80 --name nginx nginx
```
선택된 docker server가  default 가상 머신이므로 브라우저로 접속해 확인할 수 있음.
브라우저에서 192.168.99.100:8080으로 접근하면 nginx 서버가 작동 중인 것을 확인할 수 있음.

![그림4.4](https://github.com/ALGOSOPT/DockerConcept/blob/master/picture/4.4.jpg)

docker-machine ssh 명령어로 특정 서버에 접속할 수 있음. 보통 docker machine에서 docker 
server를 관리하므로 직접 SSH로 접속할 경우가 많지 않지만 docker server에 error가 발생하는 등 
직접 server에 접속할 일이 생길 때 주로 사용함. docker server에서 docker machine으로 돌아오려면 
exit을 입력함.

![그림4.5](https://github.com/ALGOSOPT/DockerConcept/blob/master/picture/4.5.jpg)

---
### 4.2.2 virtual box를 이용한 local virtual machine 생성

docker machine을 사용하는 가장 기본적인 방법은 가상 머신을 생성해서
사용하는 것. docker toolbox가 설치될 때 virtual box도 함께 설치되므로
별도로 설치할 필요는 없음.

새로운 가상 머신을 생성하는 명령어는 docker-machine create이며 --driver 옵션으로
생성될 docker server의 환경을 명시할 수 있음. 다음 예제에서는 --driver 옵션에
virtual Box를 설정했으므로 virtualbox 환경에서 가상 머신을 생성함.
create 명령어의 마지막에 입력한 mydocker는 생성될 가상 머신의 이름.

```
docker machine은 VMware도 지원하지만 docker toolbox에 포함된 가상화 툴이
virtualbox이므로 virtualbox를 기준으로 설명
```

```
# docker-machine create \
--driver virtualbox \
mydocker
```

간단하게 새로운 docker server를 생성했음. docker machind은 boot2docker.iso 라는 
경량화된 linux 배포판 이미지를 이용해 가상 머신을 생성함. 
따라서 virtualbox 같은 가상화 도구에서 CentOS나 우분투 같은 리눅스 배포판을
직접 설치하는 것보다 간편하게 설치할 수 있을 뿐더러 노트북과 같이 
하드웨어 자원이 부족한 상황에서도 비교적 부담 없이 linux 가상 환경을 사용할 수 있음.

생성된 docker server를 확인ㅇ해보자

```
# docker-machine ls
```

IP주소가 192.168.99.101인 mydocker 가상 머신이 생성되었지만 ACTIVE 항목에서
별표가 default에 표시되어 있으므로 선택된 docker server는 여전히 default.
eval 명령어와 docker-machine env 명령어로 다른 docker server를 선택할 수 있음.

```
# eval $(docker-machine env mydocker)
```

다시 docker server의 목록을 확인하면 생성한 가상 머신을 사용하도록 설정되어 있음. 
mydocker 의 ACTIVE 항목에 별표가 표시되어 있으므로 앞으로 docker 명령어를 사용하면
mydocker의 가상 머신을 제어하게 됨.

```
# docker-machine ls
```
container와 비슷하게 가상 머신은 정지할 수 있고 다시 시작할 수도 있음.
docker-machine rm 명령어로 가상 머신을 삭제할 수도 있지만 이 경우 가상 머신에 
존재하는 각종 container와 image도 함께 삭제되므로 주의해야함.

```
# docker-machine stop mydocker


# docker-machine start mydocker
```

docker-machine으로 시작하는 명령어는 사용할 docker server의 이름을 명시하지 않으면
기본적으로 default 가상 머신을 대상으로 함. 이는 start, stop과 같은 명령어뿐 아니라
ssh, run 등 docker server를 제어하는 명령어에도 적용됨

```
# docker-machine stop
```

따라서 default 가상 머신을 삭제하면 docker machine의 명령어가 제대로 동작하지 않을 수 
있음. 그리고 default 가상 머신을 삭제해도 Quickstart Terminal을 실행하면 자동으로
default 가상 머신이 생성됨.

---
### 4.2.3 cloud에 docker server 생성

docker-machine create 명령어로 가상 머신 뿐 아니라 cloud 공급자로부터 server를
할당받아 docker server를 생성할 수 있음. 사용 가능한 대표적인 cloud service로는
AWS, 디지털 오션, Azure, Openstack 등이 있음. cloud에 생성된 docker server를 사용하는
방법은 가상 머신에 생성된 docker server를 사용하는 방법과 같음.

이번 절에는 개발자들이 주로 이용하는 cloud service인 AWS, Azure에 연결해 docker 
server를 생성하는 방법을 설명.
docker machine에서 사용 가능한 모든 cloud 서비스를 확인하고 싶다면 docker 공식 문서의
docker machine driver 항목을 참고하길 바람.

---

### 4.2.4 On Premise 환경에 연결

Docker Machine은 각종 cloud 서비스에 연결해 docker server를 생성할 수 있지만
On premise환경의 서버에 연결해 사용할 수도 있음. 기존의 개발 server에 docker engine을
설치하고 싶다면 docker machine에 등록한 뒤 가상 machine이나 cloud의 docker server와 같이
docker machine에서 사용하면 됨.

on premise  서버에 연결하는 것은 cloud service에 연결했던 것과는 다른 방식.

그림 4.12

[1] 연결하려는 server에 docker 명령어를 사용할 수 있는 즉, root나 sudo 권한을 가진 계정이고
server에는 docker engine이 설치 되지 않은 상태여야 함.

[2] 계정의 ~/.ssh directory에 ssh key 쌍 파일이 있어야 하며, 이 키 쌍 파일은 docker machine을 사용
하는 host에도 있어야 함.

[3] docker machine 은 해당 SSH 키와 계정으로 server에 접속해 docker engine을 설치한 뒤 제어 함.

```
(참고) 여기서 설명하는 예제는 on premise 서버의 root 계정에 접근해 docker 를 제어하며
이 root 계정은 비밀번호를 사용하고 있다고 가정. 그러나 실제로는 root 계정을 사용하기보다는
별도의 계정에 sudo 권한을 부여해 사용하는 것이 바람직함. docker server에 존재하는 다른 
계정을 사용해 docker machine에서 접근하려면 이번 절의 예제에서 사용하는 root를 다른 계정으로
변경해 사용하길 바람. 단 해당 계정에는 비밀번호를 입력하지 않아도 되는 sudo 권한을 부여되어 있어야
함. 특정 계정에 이 권한을 부여하려면 visudo 명령어를 입력한 뒤 아래와 같은 내용을 추가 함

# visudo
...
myusername ALL=(ALL) NOPASSWD: ALL
...
```

docker machine을 사용하는 Quickstart Terminal에서 ssh-keygen을 입력해 새로운 key 쌍을 생성함.
key를 저장할 위치와 key의 비밀번호를 설정할 수 있는데 이 예제에서는 어떠한 설정도 하지 않고
기본 설정을 따름.
```
# ssh-keygen

```
docker machine이 존재하는 host의 ~/.ssh/ 디렉터리에 id_rsa와 id_rsa.pub 파일이 생성됨.
이 file들을 docker machine에 등록하려는 docker server에 복사해야 함. 다음 명령어를 입력하면
SSH 키 파일들이 docker server root계정의 ~/.shh/ directory에 복사됨. 진행 과정에서 연결 확인
여부를 물어보면 'yes' 를 입력하고 root계정의 비밀번호를 입력함

```
$ ssh-copy-id root@(docker server IP)
```

key file 복사가 완료도면 다음 명령어를 입력해 docker server를 등록함. --driver 옵션에 generic을
설저하고 --generic-ip-address에 docker server의 IP 주소를 설정함.
```
# docker-machine create --driver generic --generic-ip-address (IP) myserver
```

--driver generic으로 docker server를 등록할 경우, docker machine은 기본적으로 root계정으로 접근함.
docker server로 접근할 때 root 계정이 아닌 다른 계정으로 변경하려면 --generic-ssh-user 옵션을 사용하면
됨. 또한 docker machine이 존재하는 host에서 docker server에 접근할 SSH 키 파일의 기본 위치는 ~/.ssh 이지만
--generic-ssh-key 옵션으로 key file이 존재하는 directory 위치를 지정할 수 있음.

```
# docker-machine create --driver generic \
--generic-ip-address (IP) \
--generic-ssh-user myuser \
--generic-ssh-key ~/.Download/my_key.pub \
myserver
```

정상적으로 server가 등록되면 해당 서버에 자동으로 docker engine을 설치한 뒤 TLS연결을 설정함.
등록된 server는 다른 docker server와 같이 docker machine의 명령어로 제어할 수 있음.

```
# docker-machine create --driver generic \
--generic-ip-address (IP) \
--generic-ssh-user myusername \
myserver
```

