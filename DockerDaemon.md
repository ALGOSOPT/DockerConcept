# Docker daemon

지금까지 docker를 사용하는 방법을 설명. 사장 먼저 알아야할 container부터 시작해서 container의 밑바탕이 되는 image, 그리고
그 image를 생성할 수 있는 docker file을 알아봄. 그렇자면 이제는 docker 자체를 다뤄볼 차례. 
docker 자체에 사용할 수 있는 여러 옵션을 익히면 container와 image를 좀 더 쉽게 사용할 수 있을 뿐더러
docker를 이용한 개발이 더욱 수월해짐


---

### Docker의 구조

docker를 사용할 때 docker라는 명령어를 맨 앞에 붙여서 사용해왔음. 그렇다면 docker는 실제로 어디에 있는걸까?
which 명령어로 docker 명령어의 위치를 확인 할 수 있음.

```
# which docker
```
보다 시피 docker 명령어는 /usr/bin/docker에 위치한 file로 docker  명령어를 실행되고 사용하고 있음.
이번에는 실행 중인 docker process를 확인해보자

```
# ps aux | grep docker
```

```
(참고) 리눅스의 which 명령어는 명령어의 파일이 위치한 경로를 출력
ps aux 명령어는 실행 중인 프로세스의 목록을 출력
```

container나 image를 다루는 명령어는 /usr/bin/docker에서 실행되지만 docker engine의 process는
/usr/bin/dockerd 파일로 실행되고 있음. 이는 docker 명령어가 실제  docker engine이 아닌
클라이언트로서의 docker이기 때문

docker의 구조는 크게 2가지고 나뉨. 하나는 client로서의 docker이고, 다른 하나는 server로서의 docker
실제로 container를 생성하고 실행하며 image를 관리하는 주체는 docker server이고,
이는 dockerd process로서 동작함. docker engine은 외부에서 api를 입력받아 
docker engine의 기능을 수행하는데, docker 프로세스가 실행되어 server로서 입력을 받을 준비가 된 상태를
**docker daemon** 이라고 함.


다른 하나는 docker client. Docker Daemon은 API를 입력 받아 docker engine의 기능을 수행하는데, 
이 API를 사용할 수 있도록 CLI(Command Line Interface)를 제공하는 것이 docker client.
사용자가 docker로 시작하는 명령어를 입력하면 docker client 를 사용하는 것미여, 
docker client는 입력된 명령어를 local에 존재하는 docker daemon에게 API로서 전달.

이때 Docker Client는 /var/run/docker.sock에 위치한 unix socket을 통해 docker daemon의 API를
호출함. docker client가 사용하는 unix socket 같은 host 내에 있는 docker daemon에게 명령을
전달할 때 사용됨.

tcp로 워격에 있는 docker daemon을 제어하는 방법도 있지만 이는 뒤에서 자세히 설명함.

그림 2.68


즉, 터미널이나, PuTTY 등으로 docker 가 설치된 host에 접속해 docker 명령어를 입력하면
아래와 같은 과정으로 docker가 제어됨.

1. 사용자가 docker ps 같은 docker 명령어를 입력함.

2. /usr/bin/docker는 /var/run/docker.sock 유닉스 소켓을 사용해 docker daemon에게
명령어를 전달함.

3. docker daemon은 이 명령어를 parsing하고 명령어에 해당하는 작업을 수행함.

4. 수행 결과를 docker client에게 반환하고 사용자에게 결과를 출력함.


이 것은 아무런 설정을 하지 않았을 때 일반적으로 docker daemon을 제어하는 순서.
docker daemon에 각종 옵션을 추가해 실행한다면 위 순서에 별도의 과정이 포함될 수 있음.


---

### 2.5.2 Docker Daemon Execute

docker daemon은 일반적으로 아래와 같으 명령어로 시작, 정지할 수 있음. 
ubuntu에서는 docker가 설치되면 자동으로 service로 등록되므로 host가 
재시작되더라도 자동으로 실행.


```
# service docker start
# service docker stop
```

레드햇 계열의 운영체제는 docker를 설치해도 자동으로 실행되도록 설정되지는 않음.
docker 를 자동으로 실행하도록 설정하려면 아래의 명령어로 docker service를 활성화함.

```
# systemc enable docker
```

앞에서 설명했듯이 docker service는 dockerd로 docker daemon을 실행함. 

그러나 서비스를 사용하지 않고 직접 docker daemon을 실행할 수도 있음.
docker service를 정지한 뒤 명령어로 docker를 직접 실행해보자

dockerd 명령어 또한 /usr/bin/dockerd로서 존재하지 때문에 docker 명령어와 같이 
바로 사용할 수 있음.

```
# service docker stop
# dockerd
```


dockerd르 입력하면 docker daemon이 실행됨.

그러면 docker daemon에 대한 각종 정보가 출력되는데, 마지막에
/var/run/docker.sock에 입력(listen)을 받을 수 있는 상태라는 message가 출력됨.
terminal을 하나 더 연 다음 docker 명령어를 입력하면 docker를 사용할 수 있음.

termina,에 실행된 docker daemon을 종료하려면 Ctrl + C를 입력하면 됨.

```
(참고) 직접 docker daemon을 실행하면 하나의 terminal을 차지하는 foreground 상태로 
실행될 뿐더러 보안 측면에서도 바람직하지 않으므로 보통의 service의 형태로 실행함.
실제 운영 환경에서는 위와 같이 docker daemon을 직접 실행하지는 않으며, debugging 용도로
쓰일 때만 직접 docker daemon을 실행하곤 합니다.
```

---

### 2.5.3 Docker Daemon 설정

dockerd 명령어는 docker daemon이라는 명령어로 사용될 수 있으며, docker damon 명령어는 dockerd를 간접적으로
실행하므로 두 명령어는 동일함. 단, docker daemon 명령어는 1.13.0 버전부터 폐기 예정(Deprecated)이며 추후 
update에서 삭제될 예정이므로 dockerd를 사용하는 것이 좋음.
```
# docker daemon
```

그럼 docker daemon에 입력할 수 있는 명령어로는 무엇이 있는지 --help옵션을 추가해 확인해보자.

```
# dockerd --help
```

docker daemon에 적용할 수 있는 option은 매우 많음. --help의 출력결과를 자세히 살펴보면 registry container
를 구축할 때 사용했던 --insecure-registry도 있고 container의 logging을 설정할 때 사용했던 --log-driver,
storage 


