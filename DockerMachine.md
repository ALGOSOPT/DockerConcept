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

그림 4.1

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


그림 4.2

Quickstart Terminal을 실행하면 다음과 같은 출력 결과를 확인할 수 있는데, 
이 것은 기본적으로 사용하도록 설정된 default라는 이름의 가상 머신을 생성하는 과정

정상적으로 docker machine의 초기화 작업이 완료되었다면 다음과 같은 출력을 확인할 수 있음.

그림 4.2

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

