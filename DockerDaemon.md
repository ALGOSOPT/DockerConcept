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

 

![그림2.68](https://github.com/ALGOSOPT/DockerConcept/blob/master/picture/2.68.jpg)

 

 

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

storage backend를 변경할 때 사용했던 --storage-opt도 있음. 

 

이 전 장에서 부록 A 'Docker 데몬 시작 옵션 변경하기'를 참고해 사용했던 옵션은 전부 Docker Daemon에 추가적인

옵션을 부여해 service로서 시작한 것.

 

물론 부록 A에서 설명한 방법을 사용하지 않고 옵션을 직접 추가해 Docker Daemon을 실행할 수도 있음.

다음과 같으 명령어는 Docker Private Registry 장에서 설명했던 --insecure-registry 옵션을 사용하도록 Docker Daemon을 직접

실행함.

 

```

# dockerd --insecure-registry=192.168.99.100:5000

```

 

그러나 이전 절의 마지막에서 설명했던 바와 같이 dockerd 명령어로 docker daemon을 직접 실행하는 것보다

Docker 설정 파일을 수정한 뒤 docker daemon이 설정 file을 읽어 서비스로 실행되게 하는 것이 일반적

 

이번 절에서는 Docker Daemon의 옵션을 설명하기 위해 dockerd 명령어오 예를 들지만 실제로 사용할 때는

option을 그대로 설정 파일의 DOCKER_OPTS에 입력하면 됨.

 

예를 들어, 다음의 두 방식은 동일하게 Docker Daemon을 설정하며, 직접 Docker Daemon을 실행하느냐 또는 

서비스로 실행하느냐의 차이만 있음.

 

다음 명령어는 dockerd로 직접 Docker Daemon을 실행함.

 

```

# dockerd -H tcp://0.0.0.0:2375 --insecure-registry=192.168.100.99:5000 --tls=false

```

 

아래 예제의 설정 파일인 /etc/default/docker는 ubuntu 14.04에서 Docker를 실행하는 경우의 설정 file.

 

docker 라는 이름의 서비스가 이 파일을 읽어 docker daemon 서비스로서 실행함

 

```

# vi /etc/default/docker

 

 

```

 

```

(참고)Docker Daemon의 설정 옵션은 매우 많으면서 여기서 모든 옵션을

설명하지는 않음. 전체 옵션의 설명을 확인하고 싶다면 Docker 공식 메뉴얼을 참조

하셈.

```

 

 

#### 2.5.3.1 Docker Daemon 제어 : -H

 

-H 옵션은 도커 Daemon의 API를 사용할 수 있는 방법을 추가함. 아무런 옵션을 설정하지 않고 Docker Daemon을 실행하면

Docker Client인 /usr/bin/docker를 위한 유닉스 socker인 /var/run/docker.sock을 사용함.

 

그러므로 단순히 dockerd를 입력해 docker daemon을 실행해도 docker client의 CLI을 사용할 수 있음.

 

즉, 다음의 두 명령어는 차이가 없음.

 

```

# dockerd

# dockerd -H unix:///var/run/docker.sock

```

 

-H에 IP주소와 port 번호를 입력하면 원격 API인 Docker remote API로 도커를 제어할 수 있음.

Remote API는 Docker Client와 다르게 Local에 있는 Docker Daemon이 아니더라도 제어할 수 있으며

RESTful API 형식을 띠고 있으므로 HTTP 요청으로 docker를 제어할 수 있음.

 

다음과 같이 Docker Daemon을 실행하면 host에 존재하는 모든 network interface의 IP주소와

2375번 port 를 바인딩해 입력을 받음.

 

```

# dockerd -H tcp://0.0.0.0:2375

```

 

-H 에 unix:///var/run/docker.sock을 지정하지 않고, 위와 같이 Remote API만을 위한 

Binding 주소를 입력했다면 unix socket은 비활성화 되므로 docker client를 사용할 수 

없게 되며, docker로 시작하는 명령어를 사용할 수 없음. 

 

따라서 docker client를 위한 unix 소켓과 remote API를 위한 binding 주소를 동시에

설정함.

 

 

다음은 host의 모든 network interface card에 할당된 IP 주소와 2375번 port로

docker daemon을 제어함과 동시에 docker client도 사용할 수 있는 예

 

```

# dockerd -H unix:///var/run/docker.sock -H tcp://0.0.0.0:2375

```

 

docker client가 Docker Daemon에게 명령어를 수행하도록 요청할 때도 내부적으로는 같은

API를 사용하므로 Remote API 또한 Docker Client에서 사용 가능한 모든

명령어를 사용할 수 있음.

 

-H로 Remote API를 사용하려면 cURL 같은 HTTP 요청 도구를 사용함.

 

예를 들어, IP주소가 192.168.99.100인 Docker Host에서 -H로 Remote API를 허용했다면

다른 host에서 다음과 같이 Remote API를 사용할 수 있음

 

```

root@:/# dockerd -H tcp://192.168.99.100:2375

```

 

```

root@:/# curl 192.168.99.100:2375/version

```

 

![그림2.69](https://github.com/ALGOSOPT/DockerConcept/blob/master/picture/2.69.jpg)

 

위 예에서는 -H tcp://192.168.99.100:2375라는 옵션을 사용해 192.168.99.100이라는 IP주소와

2375번 포트로 docker daemon을 바인딩했습니다. 그리고 다른 host에서 192.168.99.100:2375/version

이라는 URL로 http 요청을 보내 docker daemon의 버전을 확인했음.

즉, 192.168.99.100:2375/version으로 HTTP 요청을 전송하는 것은 docker version이라는 명령어와

같음.

 

 

Remote API의 종류는 docker 명령어의 개수만큼 있으며, API에 따라서 사용하는 방법이 docker 명령어와

조금씩 다른 부분도 있으므로 HTTP 도구로 직접 API 요청을 전송하기 보다는 특정 언어로 바인된 library

를 사용하는 것이 일반적. 뒤에서 자세히 설명하겠지만, Remote API를 특정 언어의 라이브러리로 사용하면

Remote API를 사용하기 위해 일일이 HTTP 요청을 사용할 필요도 없도,  docker를 활용할 수 있는 다양한

전략을 세울 수도 있습니다.

 

```

(참고) Remote API로 사용할 수 있는 docker 명령어를 HTTP 요청으로 사용ㅎ는 자세한 방법에 대해

알고 싶다면 Docker API 공식문서를 참고!

```

 

 

shell의 환경변수를 설정해 원격에 있는 docker를 제어할 수도 있음. Docker client는 shell의 

DOCKER_HOST 변수가 설정되어 있다면 해당 Docker Daemon에  API 요청을 전달함. 다음예시

는 192.168.99.100:2375의 주소에 연결애 Docker Daemon을 제어함

 

 

```

root@docker-remote-client :/# export DOCKER_HOST="tcp://192.168.99.100:2375"

root@docker-remote-client :/# docker version

```

 

또는 Docker Daemon Client에 -H 옵션을 설정해 제어할 원격 Docker Daemon을 설정할 수 있음.

-H 옵션을 사용하지 않으면 local에 있는 Docker Daemon을 제어함

 

```

# docker -H tcp://192.168.99.100:2375 version

```

 

 

---

---

### 2.5.3.2 Docker Daemon에 보안 적용 : --tlsverify

 

도커를 설치하면 기본적으로 보안 연결이 설정되어 읶ㅆ지 않음. 이는 Docker Client, Remote API를 사용할 때

별도의 보안이 적용되지 않음을 의미. 그러나 실제 운영 환경에서 Docker를 사용해야 한다면 보안을 적용하지

않는 것은 바람직하지 않음. 보안이 적용되어 있지 않으면 Remote API를 위해 바인딩된 IP주소와 포트 번호만 알면

도커를 제어할 수 있기 때문. 이번에는 Docker Daemon에 TLS보안을 적용하고, Docker Client와 Remote API 

Client가 인증되지 않으면 Docker Daemon을 제어할 수 없도록 설정하는 방법을 설명하고자 함.

 

 

![그림2.70](https://github.com/ALGOSOPT/DockerConcept/blob/master/picture/2.70.jpg)

 

보안을 적용할 때 사용될 파일은 총 5개로서, ca.pem, server-cert.pem, server-key.pem, cert.pem, key.pem

그림 2.64와 같이 Client측에서는 Docker Daemon에 접근하려면 ca.pem, cert.pem, key.pem 파일이 필요함

 

다음 과정을 하나씩 따라해 보안을 적용하는데 필요한 파일을 생성.

 

 

##### 1. 서버 측 파일 생성

1) 인증서에 사용될 key를 생성

```

# mkdir keys && cd keys

# openssl genrsa -aes256 -out ca-key.pem 4096

```

 

2) Public Key를 생성. 입력하는 모든 항목은 공백으로 둬도 상관 없음.

```

# openssl req -new -x509 -days 10000 -key ca-key.pem -sha256 -out ca.pem

```

 

3) 서버 측에서 사용 될 키를 생성함

```

# openssl genrsa -out server-key.pem 4096

```

 

4) 서버 측에서 사용될 인증서를 위한 인증 요청서 파일을 생성함 $HOST 부분에는 사용 중인 Docker host의 IP

주소 또는 Domain 이름을 입력하며, 이는 외부에서 접근 가능한 IP주소 또는 도메인 이름이어야 함

 

```

# openssl req -subj "/CN=$HOST" -sha256 -new -key server-key.pem -out server.csr

```

 

5) 접속에 사용될 IP 주소를 extfile.cnf 파일로 저장. 위와 마찬가지로 $HOST에는 docker host의 IP 주소

또는 domain이름을 입력함.

 

```

# echo subjectAltName = IP:$HOST, IP:127.0.0.1 > extfile.cnf

```

 

6) 다음 명령을 입력해 서버 측의 인증서 파일을 생성함. 다음 예에서는 192.168.99.100 으로 접속하는 연결에 사용되는 인증서 파일이 생성됨.

```

# openssl x509 -req -days 356 -sha256 -n server.csr 

-CA ca.pem -CAkey ca-key.pem -CAcreateserial -out server-cert.pem

-extfile.cnf

```

 

##### 2. Client 측에서 사용할 파일 생성

1) Client측의 키 파일과 인증 요청 파일을 생성하고, extfile.cnf 파일에 extendedKeyUsage 항목을 추가함.

```

# openssl genrsa -out key.pem 4096

 

# openssl req -subj '/CN=client' -new -key key.pem -out client.csr

# echo extendedKeyUsage = clientAuth > extfile.cnf

```

 

2) 다음 명령어는 입력해 Client 측의 인증서를 생성

```

# openssl x509 -req -days 30000 -sha256 -in client.csr 

-CA ca.pem -CAkey ca-key.pem -CAcreateserial out cert.pem -extfile.cnf

```

 

3) 여기까지 따라 했다면 다음과 같은 파일이 만들어 졌는지 확인

```

# ls

ca-key.pem ca.pem ca.srl cert.pem client.csr extfile.cnf key.pem server-cert.pem

server.csr server-key.pem

```

 

4) 생성된 파일의 쓰기 원한을 삭제해 읽기 전용 파일로 만듬.

```

# chdmod -v 0400 ca-key.pem server-key.pem ca.pem server-cert.pem cert.pem

```

 

5) Docker Daemon의 설정 파일이 존재하는 directory인 ~/.docker로 Docker Daemon 측에서 필요한 file을

옮김. 이 것은 필수적이지는 않지만 docker daemon에 필요한 file을 한 곳에 모아두면 관리하기 쉬움.

 

```

# cp {ca,server-cert,server-key,cert,key}.pem ~/.docker

```

보안 적용을 위한 파일을 모두 생성했으므로 다음 명령어를 입력해 암호화가 적용된 Docker daemon을 실행함.

TLS 보안을 적용을 활성화하기 위해 -tlsverify 옵션을 추가하고, --tlscacert, --tlscert, --tlskey에는

각각 보안을 적용하는 데 필요한 파일의 위치를 입력함

 

```

# dockerd --tlsverify \

--tlscacert=/root/.docker/ca.pem \

--tlscert=/root/.docker/server.pem \

--tlskey=/root/.docker/server-keypem \

-H=0.0.0.0:2376 \

-H unix:///var/run/docker.sock

```

 

이 전장에서도 언급했지만 위와 같이 --tlsverify 등의 모든 Docker Daemon 옵션을 부록 A에 설명한 대로

Docker service의 설정 파일을 변경해 사용할 수 있다는 점을 알아두기 바람.

 

이제 Docker Daemon에 TLS 보안이 정상적으로 적용되었음. 다른 Docker Host에서 Docker Client에

-H를 추가해 보안이 적용된 Docker를 제어해보자

 

```

# docker -H 192.168.99.100:2376 version

```

 

```

(참고) 도커의 Remote API를 사용하는 port는 보안이 적용되어 있지 않다면 2375번 포트를,

보안이 적용되어 있다면 2376번 포트를 사용하도록 설정. 이는 반드시 지켜야 할 규칙은 아니지

만 Docker 커뮤니티 내에서 약속한 관례

```

 

TLS 연결 설정을 하지 않앆다는 error가 출력 되는 것을 볼 수 있음. 보안이 적용된 Docker Daemon을 

사용하려면 ca.pem, key.pem, cert.pem 파일이 필요함. 이 파일을 docker 명령어의 옵션에 명시하고

다시 원격 제어를 시도함

 

```

# docker -H 192.168.99.100:2376 \

--tlscacert=/root/.docker/ca.pem \

--tlscert=/root/.docker/cert.pem \

--tlskey=/root/.docker/key.pem \

--tlsverify version

```

 

인증이 정상적으로 이뤄졌고, Docker Client로 원격 제어가 정상적으로 수행 된 것을 알 수 있음.

그러나 매번 Docker 명령어를 입력할 때마다 위와 같이 인증 관련 옵션을 입력하는 것은

귀찮은 일.

이 또한 shell의 DOCKER_HOST 환경변수와 마찬가지로 인증관련 환경변수를 설정해 매번

파일의 위치를 입력하지 않게 설정할 수 있음. DOCKER_CERT_PATH는 Docker Daemon 인증에 필요한 file의

위치를, DOCKER_TLS_VERIFY는 TLS 인증을 사용할지를 설정함

 

```

# export DOCKER_CERT_PATH="/root/.docker"

# export DOCKER_TLS_VERIFY=1

 

# docker -H 192.168.00.100:2376 version

```

 

```

(참고) Shell의 환경변수는 shell이 종료되면 초기화 되므로

~/.bashrc 등의 파일에 export를 추가해 shell을 사용할 때마다

환경변수의 값을 설정할 수 있음. 아래는 root 계정에서 ~/bashrc 파일을 수정하는

예시.

 

# vi ~/.bashrc

 

... 

export DOCKER_CERT_PATH="/root/.docker"

export DOCKER_TLS_VERIFY=1

...

```

 

curl로 보안이 적용된 docker daemon의 remote api를 사용하려면

다음과 같이 flag를 추가함. 사용되는 파일은 docker client에서 사용했던 것과 동일.

```

# curl https://192.168.99.100:2376/version \

--cert ~/.docker/cert.pem \

--key ~/.docker/key.pem \

--cacert ~/.docker/ca.pem \

```

 

 

---

#### 2.5.3.3 Docker Storage Driver 변경 : --storage-driver

 

도커는 특정 storage 백엔드 기술을 사용해 container와 image를 저장하고 관리함. 일부 운영체제는 docker를

설치할 때 기본적으로 사용하도록 설정된 storage driver가 있는데,

우분투 같은 데비안 계열 운영체제는 AUFS를, CentOS 같은 레드햇 운영체제는 OverlayFS를 사용하는 것이

대표적인 예. 이는 docker info 명령어로 확인할 수 있음.

 

```

# docker info | grep "Storage Driver"

```

 

docker를 사용하는 환경에 따라 storage driver는 자동으로 정해지지만 docker daemon 실행 옵션에서

storage driver를 변경할 수 도 있음. storage driver는 docker daemon 옵션 중 --storage-driver를

사용해 선택할 수 있으며 지원하는 driver로는 OverlayFS, AUFS, Btrfs, Devicemapper, VFS,

ZFS 등이 있음. 이 가운데 하나만 선택해 docker daemon에 적용할 수 있으며, 적용된 storage driver에 따라

Container와 image가 별도로 생성됨.

 

예를 들어, Docker가 AUFS를 기본적으로 사용하도록 설정된 ubuntu에서 다음과 같이 Docker Daemon을 실행하면

별도의 Devicemapper 컨테이너와 이미지를 사용하므로 AUFS에서 사용했던 image와 container를 사용할 수 없음.

별도로 생성된 Devicemapper 파일은 /var/lib/docker/devicemapper 디렉터리에 저장되며,

AUFS 드라이버 또한 /var/lib/docker/aufs 디렉터리에 저장됨.

```

# dockerd --storage-driver=devicemapper

```

어떤 storage driver를 사용할 지 말지는 개발하는 container application 및 개발 환경에 따라 다름.

레드 햇 계열 운영체제를 사용하고 있다면 OverlayFS를 선택하는 것이 옳은 선택이 될 수도 있고,안정성을

우선시 하는 container application을 개발하고 있다면 Btrfs가 좋은 선택이 될 수도 있음.

 

무조건 좋은 storage driver라는 것은 없기 땜누에 상황에 따라 각 드라이버의 장단점을 감안해 선택하는 것이

바람직함. 이러한 storage 선택 가이드에 대해서는 docker storage 공식 문서를 참고.

 

```

(참고) docker daemon이 사용하는 container와 image가 저장되는 directory를 별도로

지정하지 않았다면 driver 별로 사용되는 container와 이미지는 /var/lib/docker에 저장됨.

container와 image파일들이 저장될 directory를 임의로 지정하려면 docker daemon을 실행할 때,

--graph 또는 -g 옵션을 사용함. 단, 아무것도 없는 directory를 -g 옵션의 입력으로 설정하면

docker engine이  초기화된 상태로 docker daemon이 실행됨

 

# dockerd -g /DATA/docker

 

단, 여러 개의 device driver의 directory가 하나의 directory에 존재할 경우 --storage-driver

옵션으로 사용할 드라이버를 명시하지 않으면 docker daemon이 어느 드라이버를 사용할지 찾지 못하는

상황이 발생해 docker daemon이 시작되지 않음. 따라서 test 용도로 어러 개의 storage driver를

번갈아 사용하고 있다면 반드시 storage driver를 명시하는 것이 좋음.

```

 

 

##### storage driver 원리

 

2.3절 '이미지 다루기' 에서 container와 image가 어떻게 동작하는지 알아보았음.

image는 읽기 전용 파일로 사용되며 container는 이 image위에 얇은 container layer를 생성함으로써

container의 고유한 공간을 생성한다는 것. 그러나 이 것은 기본적인 개념이고, 실제로 container 내부에서 

읽기와 새로운 파일 쓰기, 기존의 파일 쓰기 작업이 일어 날 때는 Driver에 따라 Copy-on-Write(COW)

또는 Redirect-on-Write(RoW) 개념을 사용함. Storage 기술을 상세하게 다루는 것은 이 책의 범위를

벗어나므로 자세하게 설명하지는 않지만 Docker Storage Driver는 Cow, RoW를 지원해야 하므로 

이 개념에 대해 간단히 짚고 넘어 가겠음.

 

snap shot의 기본 개념은 '원본 파일은 읽기 전용으로 사용하되 이 file이 변경되면 새로운 공간을

할당한다' 임. storage를 snap shot으로 만들면 snap shot 안에 어느 파일이 어디에 저장되어 있는지가

목록으로 저장됨. 그리고 이 snapshot을 사용하다가 snapshot 안의 파일에 변화가 생기면 변경된 내역을 

따로 관리함으로써 snapshot을 사용함

 

![그림2.71](https://github.com/ALGOSOPT/DockerConcept/blob/master/picture/2.71.jpg)

 

예를 들어, 그림 2.65에서 A,B,C 파일이 snap shot으로 생성되었다면, 이 파일에 읽기 작업을 수행하는

App은 단순히 file system의 원본 파일에 접근해 file의 내용을 읽으면 됨.

그러나 app이 snapshot의 A 파일에 쓰기 작업을 수행해야 할 경우에는 조금 상황이 달라짐.

 

원본 파일을 유지하면서도 변경된 사항을 저장할 수 있어야 하기 때문.

 

이를 해결하는 방법에 따라 CoW, RoW로 나뉨.

 

![그림2.72](https://github.com/ALGOSOPT/DockerConcept/blob/master/picture/2.72.jpg)

 

 

CoW는 snapshot의 파일에 쓰기 작업을 수행할 때 snapshot 공간에 원본 파일을 복사 한 뒤 쓰기 요청을 반영함.

이 과정에서 복사하기 위해 파일을 읽는 작업 한 번, 파일을 스냅숏 공간에 쓰고 변경된 사아을 쓰는 작업으로 총 2번의 쓰기

작업이 일어나므로 overhead가 발생함

 

![그림2.73](https://github.com/ALGOSOPT/DockerConcept/blob/master/picture/2.73.jpg)

RoW는 CoW와 다르게 한 번의 쓰기 작업만 일어남. 이는 file의 snapshot 공간에 복사하는 것이 아니라

snapshot 에 기록된 원본 파일은 snapshot 파일로 묶은(Freeze) 뒤 변경된 사항을 새로운 장소에 할당받아

덮어 쓰는 형식. snapshot 파일은 그대로 사용하되, 새로운 블록은 변경 사항으로써 사용하는 것.

 

 

```

(참고) Docker를 사용하기 위해 CoW, RoW를 자세하게 알 필요는 없음. 

여기서는 snapshot이라는 개념으로 snapshot 파일을 불변(Immutable) 상태로 유지할 수

있다는 점만 알아두면 된당.

```

 

 

![그림2.74](https://github.com/ALGOSOPT/DockerConcept/blob/master/picture/2.74.jpg)

 

이를 Docker Container와 Image에 적용해 보자. Image Layer는 각 Snapshot에 해당하고 Container는

이 SnapShot을 사용하는 변경점. 

 

Container Layer에는 이전 image에서 변경된 사항이 저장되어 있으며,

Container를 image를 만들면 변경된 사항이 snapshot으로 생성되고 하나의 image layer로서 존재하게

됨.

 

각 Device Driver별로 snap shot과 layer를 지칭하는 용어는 조금씩 다르지만 기본적으로

이러한 개념을 따르고 있음

 

 

###### AUFS Driver 사용하기

 

 

AUFS Driver는 Ubuntu에서 Docker를 사용할 때 자동으로 설정되는 Driver이며, Docker에서

오랜기간 사용해왔고 많은 Community에서 지원을 받고 있기 때문에 안정성 측면에서 우수하다고

평가받고 있음. 그러나 AUFS Module은 기본적으로 kernel에 포함되어 있지 않으므로

일부 운영체제에서는 사용할 수 없으며, 사용할 수 없는 대표적인 OS로는 RHEL, CentOS 등.

 

AUFS Driver를 사용하려면 부록A를 참고해 다음과 같이 설정함. Ubuntu에서는 별도로 설정하지 

않아도 됨.

 

 

```

DOCKER_OPTS="--storage-driver=aufs"

```

 

AUFS 드라이버를 사용할 수 있는 Linux 배포판인지 확인하려면 다음 명령어를 입력함.

출력 결과는 AUFS Driver를 사용할 수 있음을 뜻함.

 

```

# grep aufs /proc/filesystems

nodev aufs

```

 

![그림2.75](https://github.com/ALGOSOPT/DockerConcept/blob/master/picture/2.75.jpg)

 

AUFS Driver는 지금까지 설명한 image의 구조와 유사함. 여러 개의 image layer를 유니언 마운트 지점

(Union Mount Point)으로 제공하며, Container Layer는 여기에 Mount해서 image를 read ony로 사용함.

Union Mount Point는 /var/lib/docker/aufs/mnt에, Container Layer는 /var/lib/docker/aufs/diff에

존재함.

 

 

AUFS Driver는 Container에서 Read Only로 사용하는 파일을 변경해야 한다면 Container Layer로 전체 파일을

복사하고, 이 file을 변경함으로써 변경사항을 반영함. 복사할 file을 찾기 위해 image의 가장 위의 layer로부터

시작해 아래 Layer까지 찾기 때문에 크기가 큰 파일이 image의 아래쪽 layer에 있다면 시간이 더 오래 걸릴 수 있음.

 

그러나 한 번 파일이 container layer로 복사되고 나면 그 뒤로는 이 file로 쓰기 작업을 수행함.

 

AUFS는 Container의 실행, 삭제 등의 Container 관련 수행 작업이 매우 빠르므로 PaaS에 적합한 Driver로

손꼽히곤 함. 또한 위에서 언급한 것과 같이 Docker에서 지원하는 대표적인 Storage Driver이기 때문에

많은 사람이 일반적으로 사용하는 Driver 중 하나.

 

 

##### Devicemapper Driver 사용하기

 

Devicemapper driver는 레드햇 계열의 리눅스 배포판을 위해 개발된 Storage Driver.

AUFS가 기본적으로 Linux Kernel에 포함되어 있지 않아 래드햇 계열에서 Docker를 사용할 수 

없었기 때문에 Linux kernel에 포함된 DeviceMapper framework를 사용하는 방법을 생각해 낸 것.

따라서 대부분의 Linux 배포판에서 Devicemapper driver를 사용할 수 있으며, 이는 레드햇 계열의

리눅스인 RHEL, CentOS, Fedora뿐 아니라 Devian 계열의 리눅스도 포함됨

 

```

(참고) 레드햇 계열의 Linux는 Docker Engine이 1.13.0 버전 이전에는

Devicemapper를, 1.13.0 버전 이후에서는 OverlayFS를 기본적으로 사용하도록 

설정되어 있음.

```

 

Devicemapper 드라이버를 사용하려면 부록 A를 참고해 다음과 같이 설정함.

```

DOCKER_OPTS="--storage-driver=devicemapper"

```

Devicemapper 드라이버를 사용하도록 설정한 뒤, /var/lib/docker/devicemapper/devicemapper

directory를 살펴모면 다음과 같이 2개의 파이릉ㄹ 볼 수 있음.

 

```

# ls -al /var/lob/docker/devicemapper/devicemapper

 

data, metadata

```

 

무려 100GB 크기의 data라는 파일이 생성되어 있음. 그러나 Docker가 이 파일 전체를 실제로 사용

하는 것은 아니며, 100GB 크기의 희소 파일(sparse file)에서 공간을 할당받아 image와 container

layer를 저장한다는 것. 이는 image와 container의 data가 분리된 directory로 저장되는 것이 아닌

data 파일로 이뤄진 pool에서 block 단위로 할당받는 구조.

 

Container와 image block의 정보는 metadata 파일에 저장됨.

 

 

```

(참고) ls 명령어에 -lsah를 추가해 실제 Docker Daemon이 차지하는 

파일의 크기를 확인할 수 있음.

# ls -lsah /var/lib/docker/devicemapper/devicemapper/

```

 

 

![그림2.76](https://github.com/ALGOSOPT/DockerConcept/blob/master/picture/2.76.jpg)

 

Devicemapper Driver를 사용하는 Docker는 위 그림과 같이 Devicemapper Storage pool에서 공간을

할당받고, image의 snapshot을 만들어 상위 layer를 생성함.image로부터 변경된 사항을 저장하는

Container Layer에는 이 Layer들을 묶은 마운트 포인트에서 새로운 snapshot을 생성해서 사용됨.

 

devicemapper driver를 사용하는 container는 allocate-on-demand라는 원리로 container 내부에서

새로운 파일을 기록함. 이는  devicemapper의 pool에서 필요한 만큼의 64KB 크기의 블록 개수를

할당해서 쓰기 작업을 수행하는 것. Container 내부에 이미 존재하는, 즉 image에 존재하던

data에 쓰기 작업을 수행할 때는 변경하려는 원본 파일의 block을 container에 복사하 뒤, container

내부에 복사된 block file을 수정함.

 

이는 AUFS driver와 달리 전체 file을 복사하지 않는 다는 점에서 성능상의 이점이 있지만

devicemapper는 container를 생성하고 삭제하는 등의 작업은 빠른 편이 아님

 

```

(참고) AUFS는 Devicemapper의 container 관련 성능을 비교해보고 싶다면 virtual

box나 AWS 등으로 Ubuntu instance와 CentOS instance를 생성해 Docker를

설치하고 Container가 생성되는 시간을 확인해보는 방법을 권장함.

적어도 Container에 관련된 작업에서는 두 Driver 사이에 분명한 속도 차이가 있음.

```

 

Devicemapper의 성능을 개선하기 위해 devicemapper가 기본적으로 사용하는 mode인 loop-lvm을 direct-lvm으로

변경할 수 있으며, 이에 대한 내용은 Docker 공식 문서를 참고.

 

현재 사용 중인 mode는 docker info 명령어로 확인할 수 있음.

 

```

# docker info | grep Data

```

 

Devicemapper driver는 devicemapper라는 data pool 안에 container와 image data를 저장하기 때문에

Multitenancy 환경을 위해 각 Container와 Image를 분리해 관리하는 것이 불가능함.

따라서 실제 운영 환경에서 PaaS와 같은 용도로 사용하는 것을 권장하지 않는 추세이며,

굳이 사용하고 싶다면 Devicemapper의 storage pool로 loop-lvm이 아닌 direct-lvm을 사용

하는 것이 좋음

 

 

 

##### OverlayFS Driver 사용하기

 

OverlayFS는 레드햇 계열의 운영체제와 Raspbian에서 Docker를 설치하면 자동으로 사용되도록 설정되는  Driver임. OverlayFS

는 AUFS와 비슷한 원리로 동작하지만 좀 더 간단한 구조로 사용되며 성능 또한 좀 더 좋음.

따라서 AUFS를 대체할 수 있는 차세대 file system으로 떠오르고 있음.

 

이전에는 운영 환경에서 사용하기에는 아직 안정적이지 않다는 평이 있었으나, 최근에는 OverlayFS 파일system이 

충분히 성숙되었다고 판단되어  Docker Engine 1.13.0 version update이후부터 CentOS의 기본

Storage Driver로 채택됨.

 

OverlayFS 드라이버는 overlay와 overlay2 Driver로 나뉨. overlay는 kernel 3.18버전 이상부터

기본적으로 내장되어 있으며 overlay2는 4.0버전 이상에서 사용할 수 있음. overlay2는 overlay에 비해

성능이 좀 더 우수하며, image를 구성하기 위해 여러 개의 layer 구조를 지원함.

 

뒤에서 다시 설명하겠지만, overlay는 다른 storage driver와는 달리 계층화된 image 구조를

사용하지 않으며 lowerdir이라는 단일화된 image layer를 사용함.

아래 설명은 overlay를 기준으로 함.

 

overlay Driver를 사용하기 위해 먼저 kernel version과 overlay kernel module이 사용가능한 

상태인지를 확인함. kernel version이 너무 낮지만 않으면 overlay driver를 사용할 수 있음.

 

```

#uname -r

```

 

```

# grep overlay /proc/filesystems

```

 

Docker Engine이 사용하고 있는 file system이 extfs(ext4)인지 확인하자.

이는 필수 적인 것은 아니지만 실제 운영 환경에서는 overlay storage driver와 ext4를 함께

사용하는 것을 권장함.

 

```

# docker info | grep Filesystem

```

 

부록 A를 참고해 Docker Daemon의 옵션을 추가함. 다음 예시예서는 overlay driver를 사용하도록

설정하짐나 kernel version이 충분하다면 overlay2 driver를 사용해도 됨.

 

```

DOCKER_OPTS="... --storage-driver overlay ..."

```

 

overlay driver를 사용하도록 설정하면 docker engine이 초기화 되므로 새로운 image를 내려 받은 뒤

container를 생성해보자. 생성된 container에서는 적당한 file을 새로 생성해 image로부터

변경사항을 만듦.

```

# docker pull ubuntu:14.04

 

# docker run -it --name container ubuntu:14.04

root@:/# echo my file! >> overlayfile

```

 

호스트로 빠져나와 /var/lib/docker/overlay directory의 내용을 살펴보면 다음과 같이 container의 image 파일을

담고 있는 directory가 존재하는 것을 확인할 수 있음. 이 간운데 이름 끝에 -init이 붙은 directort가

방금 생성한 container를 의미함. 이 directory의 이름에서 -init을 제외한 directory가 실제

container의 filesystem을 담고 있는 directory임.

 

```

# ls /var/lib/docker/overlay

```

 

-init을 제외한 이름의 directory를 확인해보면 다음과 같이 4개의 file이 존재함.

```

# ls a65....

```

 

```

(참고) mount 명령어를 사용하면 container의 mount

directory를 쉽게 확인할 수 있음.

 

# mount | grep overlay

```

 

![그림2.77](https://github.com/ALGOSOPT/DockerConcept/blob/master/picture/2.77.jpg)

 

overlay driver는 container를 사용하기 위해 docker를 merged, upperdir, lowerdir의 구조로 나눔.

lowerdir는 docker의 iamge layer에 해당하고 upperdir는 container layer에 해당함.

 

다른 storage driver와는 다르게 여러 계층의 image layer가 존재하는 것은 아니며, 

여러개의 image layer를 하나의 container mount지점에 통합해 사용함.

 

upperdir에는 container에서 발생한 변경 사항을 담고 있으며, 위에서 생성한 container는 overlay file이라는 

변경 사항을 가지고 있기 때문에 upperdir directory에는 overlayfile 파일이 존재함.

그리고 upperfile은 image layer에 해당하는 lowerdir과 함께 mount되어 최종적으로 container 내부에

보여지게 되고 이 것은 container mount 지점인 merged directory임

 

```

# ls /var/lib/docker/overlay /a65... /upper/

 

# ls /var/lib/docker/overlay /a65.../merged/

```

 

AUFS와 유사하게 overlay는 image에 존재하는 file에 쓰기 작업을 수행할 때 file을 container layer인

upperdir로 복사 (copy_up operation)해 사용함. 따라서 크기가 큰 file에 쓰기 작업을 수행할 때는

upperdir에 복사하는 시간으로 인해 작업 수행에 지연이 생길 수 있으나, AUFS와는 다르게 

계층화된 다중 layer 구조가 아니기 때문에 복사할 파일을 찾는 과정에서는 AUFS보다 빠름.

 

 

##### Btrfs Driver 사용하기

 

 

Btrfs는 Linux File system 중 하나로, SSD 최적화, data compression 등 다양한 기능을 제공함.

Btrfs Driver는 DeviceMapper나 AUFS, OverlayFS와는 다르게 File system을 별도로 

구성하지 않으면 Docker에서 사용할 수 없으며, /var/lib/docker 디렉터리가 btrfs file system을

사용하는 공간에 mount되어 있어야만 Docker는 Brtfs를 storage driver로 인식함.

Btrfs는 Linux Kernel에 포함되어 있으므로 linux 배포판에서 사용할 수 있음.

 

Btrfs를 설치하는 방법은 아래와 같음.

 

1. Btrfs를 설치하기 전에 Docker Daemon이 실행 중이라면 먼저 이를 정지한 뒤 진행함.

```

# service docker stop

```

 

2. Btrfs를 생성하기 위한 도구를 설치함. 

```

# apt-get install btrfs-tools

```

3. 다음 명령어를 입력해 Btrfs storage pool을 생성함. 

다음 명령어에서는 /dev/xvdb를 입력해 해당 device를 Btrfs로 만들었지만

사용하는 Device에 맞는 이름을 입력하길 바람

```

# mkfs.btrfs -f /dev/xvdb

```

 

Docker를 아직 설치 하지 않아 /var/lib/docker directory가 존재하지 않는다면 이를 생성함.

```

# mkdir /var/lib/docker

```

 

4. system이 재부팅될 때마다 Btrfs Device가 Mount되도록 설정하기 위해 /etc/fstab 파일에 

다음 내용을 추가함. 위와 마찬가지로 /dev/xvdb는 예시일 뿐이며, 사용하는 Device에 맞게 이름을 수정하길

바람

 

```

# vi /etc/fstab

 

/dev/vxdb /var/lib/docker btrfs defaults 0 0

```

 

```

(참고) /etc/fstab file은 Linux 부팅 시 partition을 자동으로 mount하도록 설정하는 file.

위 내용 외에는 다른 내용을 수정하지 않도록 주의하길 바람.

```

 

5. /etc/fstab file에 설정한 내용을 적용하기 위해 mount -a 명령어를 입력해 fstab file에

명시된 file system을 mount 함. mount 명령어를 사용하면 file system이 mount되었는지 확인

할 수 있음.

 

```

# mount -a

# mount

```

 

6. Btrfs mount가 완료되었으며, /var/lib/docker directory는 Btrfs filesystem을 사용하는 

Device와 Mount되었음.

Docker를 사용한 뒤 Docker info명령어로 Docker Storage Driver를 확인햅좌

 

```

# service docker start

 

# docker info | grep Storage

```

 

Docker가 정상적으로 Btrfs driver를 인식하고 사용하고 있음을 알 수 있음.

 

![그림2.78](https://github.com/ALGOSOPT/DockerConcept/blob/master/picture/2.78.jpg)

 

Btrfs Driver는 image와 Container를 Subvolume과 snapshot 단위로 관리함. 

구조 자체는 AUFS, devicemapper 등 다른 storage driver와 유사함. 

image에서 가장 아래에 있는 Base Layer가 sub volume이 되고 그 위에 쌓이는 

child layer가 base layer에 대한 snap shot으로 생성됨.

 

새로운 container가 생성되면 container layer가 image의 맨 위에 있는 layer의

snap shot으로서 생성됨. Btrfs는 Devicemapper와는 달리 블록 단위가 아닌

file 단위로 각 layer를 저장하기 때문에 /var/lib/docker/btrfs directory에서 이를

확인할 수 있음.

 

Btrfs driver를 사용하는 container 내부에서 새로운 file을 생성하는 것은 devicemapper와 마찬가지로

allocate-on-demand 작업에 의해 일어남. 이미 존재하던 file에 존재하던 file에 쓰기 작업을 수행할 때는

원본 파일을 보존하고 snapshot에 새로운 공간을 할당하는 RoW방식을 사용함.

 

Btrfs는 자체적으로 SSD에 최적화되어 있으며 대체적으로 우수한 성능을 보여줌. 또한 Linux의 filesystem이

제공하지 않는 여러 기능을 제공한다는 장점도 있음.

 

 

##### ZFS Driver 사용하기

 ZFS는 Sun Microsystems에서 개발 했으며 Btrfs 처럼 압축, 래플리카, data 중복 제거 등
 다양한 기능을 제공함. 그러나 ZFS는 라이센스 문제로 linux 커널에 기본적으로 탑재되어 있지 않기
 때문에 별도의 설치 과정이 필요함. 이를 위해 ZFS on Linux 라는 project가 ZFS를 독립적으로 설치할 수 있도록
 module을 제공함.
 
 설치 방법은 다음과 같다.
 
 1. 먼저 docker daemon이 실행 중이라면 이를 정지함.
 ```
 # service docker stop
 ```
 2. ubuntu 16.04에서는 다음 명령어로 zfs를 설치할 수 있음.
 
 ```
 # apt-get install zfs
 ```
 3. 모듈을 load한다.
 ```
 # modprobe zfs
 ```
 4. 다음 명령어를 입력해 새로운 zpool을 생성. 이 zpool의 이름은 zpool-docker로 설정되어 생성함.
 이 예에서는 device의 이름을 /dev/xvdb로 지정했지만 device에 맞게 이름을 변경하기 바람.
 
 ```
 # zpool create -f zpool-docker /dev/xcdb
 ```
 5. ZFS 파일 시스템을 생성하고 이를 /var/lib/docker에 mount함.
 ```
 # zfs create -o mountpoint=/var/lib/docker zpool-docker/docker
 ```
 6. ZFS 풀이 /var/lib/docker에 정상적으로 마운트되었는지 확인함. 
 ```
 # zfs list -t all
 ```
 
 7. docker를 재시작한 뒤 docker info 명령어로 storage driver가 zfs로 설정된 것을 확인
 ```
 # service dockoer start
 
 # docker info | grep Driver
 ```
 
 ```
 (참고) /var/lib/docker에 zpool이 mount됬지만 docker가 ZFS 드라이버를 사용하고 있지 않다면
 부록 A를 참고해 docker 시작 옵션에 --storage-driver를 추가하고 값을 zfs로 지정함.
 
  DOCKER_OPTS="--storage-driver=zfs"
 ```
 
![그림2.79](https://github.com/ALGOSOPT/DockerConcept/blob/master/picture/2.79.jpg)
 
 ZFS도 용어의 차이만 있을 뿐 다른 storage driver와 유사한 구조를 띔. ZFS driver를 사용하면 
 image와  container layer는 ZFS 파일 시스템, ZFS clone, ZFS snapshot으로 구분되어 
 관리함. image의 base layer가 ZFS file system이 되고, 이로부터 ZFS snapshot을 생성해
 하나의 layer를 구성함.
 이 ZFS snapshot에서 ZFS clone을 생성해 이를 container로서 생성되며, 이전과 동일한 과정을 거쳐 하나의
 image로 사용됨.
 
 ZFS의 쓰기와 읽기 작업도 Btrfs와 유사하게 RoW를 사용함. Container layer에서 새로운 data를 
 쓸 때는 allocate-on-demand 작업에 의해 zpool로부터 새로운 block을 할당받고 쓰기 작업을 수행함.
 이미 존재하던 file에 쓰기 작업을 수행할 때는 container layer인 ZFS clone에 zpool로부터 128KB
 크기의 새로운 블록을 여러 개 할당하고 이 block에 쓰기 작업을 수행함.
 
 
 ZFS는 성능뿐 아니라 안정선에 초점을 뒀으며, 앞서 언급한 것처럼 압축과 data 중복 제거 등 여러 기능을 제공함.
 또한 ZFS는 ARC(Adaptive Replacement Cache)라고 하는 memory 구조로 disk block을 cache하기 때문에 PaaS환경에도
 나쁘지 않는 storage.
 
 그러나 기억해야할 것은 ZFS는 결고 가벼운 file system이 아니라는 점. ZFS는 memory를 상당히 소모하는 file
 system이기 때문에 ZFS를 storage driver로 사용하는 docker에서 많은 수의 container를 동시에
 사용해야 한다면 host의 자원 사용량을 수시로 확인하는 것이 좋음
 
 ---
 ---
 ### 2.5.4 Docker Daemon Monitoring
 
 Docker Daemon을 Monitoring하는 데는 여러 가지 이유가 있을 수 있음. 많은 수의 docker server를 효율적으로
 관리하기 위해서 일 수도 있고, Docker로 Container Application을 개발하다가 문제가 생겼을 때 
 그 원인을 찾아내기 위해서일 수도 있으며 Docker를 Paas로써 제공하기 위해 실시간으로 Docker Daemon의 상태를
 check해야할 수도 있음. 이러한 목적에 부합하는 monitoring방법은 매우 ㅁ낳음.
 
 제한적으로나마 Docker Engine 자체가 지원하는 monitoring 기능도 있고, Docker project에서 지원하는
 상용 solution 및 각종 open source dash board도 있음.
 이번 절에서는 하나의 container 뿐만 아니라 Docker Daemon 자체를 모니터링 하는 방법을 알아 보장.
 
 
 #### 2.5.4.1 Docker Daemon Debug mode
 
 Docker Daemon에서 어떤 일이 일어나고 있는지 가장 확실하고 정확하게
 그리고 자세히 알아내는 방법은 Docker Daemin을 Debug option으로 실행하는 것.
 이렇게 하면 Remote API의 입출력뿐만 아니라 Local Docker Client에서 오가는
 모든 명령어를 log로 출력함. Debug mode는 Docker Daemon을 실행할 때 -D 옵션을
 추가해서 사용할 수 있음.
 
 ```
 # dockerd -D
 ```
 
 ```
 (참고) Docker를 service로서 구동했을 때는 위와 같은 출력 내용이 없기 때문에
 Log 파일에서 이를 확인할 수 있음. ubuntu 14.04에서는 /var/log/upstart/docker.log
 에서 systemd를 기반으로 하는 CentOS와 우분투 16.04에서는 journalctl -u docker
 명령어로 확인할 수 있음.
 ```
 
 debug mode는 Docker Daemon에 문제가 생겼을 때 무엇이 잘못되었는지 확인하는 가장 좋은 
 수단임.
 그러나 log에는 원하지 않는 정보까지 너무 많이 출력되며, host에 있는 file을 읽거나
 Docker Daemon을 foreground 상태로 실행해야한다는 단점이 있으므로 이 방법만으로는 
 뭔가 조금 부족해 보임. 다음 절에서는 Docker가 제공하는 명령어를 통해 Docker Daemon을 
 모니터링하는 방법을 알아보자.
 
 
 #### 2.5.4.2 events, stats, system df 명령어
 
 **events
 
 Docker를 사용하는 가장 쉬운 방법은 Docker 자체가 제공하는 기능을 사용하는 것이며,
 events 명령어도 Docker가 기본으로 제공하는 명령어. events는 Docker Daemon에 어떤 일
 이 일어나고 있는지를실시간 stream log로 보여짐. 사용법은 매우 간단함. 다음 명령어 중 하나를
 입력하면 Docker Daemon이 수행한 명령어의 결과를 실시간으로 볼 수 있음.
 
 ```
 # docker events
 # docker system events
 ```
 
 위 명령어를 입력한 직후에는 어떠한 event도 Docker Daemon에 발생하지 않았으므로 아무것도 출력되지 않음.
 새로운 terminal을 연 뒤 ubuntu:14.04 image를 pull 해보자
 ```
 # docker pull ubuntu:14.04
 ```
 
 image의 pull이 완료되면 docker events를 실행했던 terminal에서 다음과 같은 명령어가 출력되는 것을
 확인할 수 있음.
 
 ```
 # docker events
 ```
 
 이 처럼 Docker Daemon에서 실행되는 명령어의 결과를 log로 출력함. 그러나 docker client에서 입력하는 모든
 명령어가 출력되는 것은 아니며, attach, co,mit, copy, ceate 등의 container 관련 명령어,
 delete, import, load, pull, push등의 image관련 명령어, volume, network, plugin 등에 관련 명령어의
 수행 결과가 출력됨.
 
 
 
 events 명령어는 filter 옵션을 사용해 워하는 정보만 출력하도록 설정할 수 있음. 출력의 종류는
 container, image, volume, network, plugin, daemon으로 나뉘는데, 특정 항목에 대한 출력 결과만
 보고 싶다면 --filter 'type= ..' 처럼 옵션을 설정하면 됨. 다음 예는 image에 관한 log만 출력하도록
 설정했으므로 docker pull, push 등 image에 관련된 명령어만 출력될 것
 
 ```
 # docker events --filter 'type=image'
 ```
 
 type 뿐 아니라 특정 container와 특정 image로 생성된 image, label 등으로 filter를 설정할 수도 있으며,
 자세한 내용은 docker의 events 명령어 항목을 참고하기 바람.
 
 
 **stats
 
docker stats 명령어는 실핼 중인 모든 container의 자원 사용령을 stream으로 출력함.
stats 명령어도 다음과 같이 간단히 사용할 수 있음.

```
# docker stats
```
stats 명령어는 실행 중인 모든 container의 CPU, memory 제한 및 사용량, network 입출력
블록 입출력(hardware 입출력) 정보를 출력함.
기본적으로 stream 형태로 출력되며 stream 이 아닌 한 번만 출력하는 방식으로 사용하고 싶다면
--no-stream 옵션을 추가함.


```
# docker stats --no-stream
```

```
(참고) Remote API로 사용할 수 있는 stats 명령어는 docker client에서 확인할 수 있는
자원 사용량보다 더욱 자세한 정보를 반환함. docker client는 %와 대력적인 크기로만
출력되지만 Remote API는 모든 data를 byte 단위로 반환하며 훨씬 자세한 항목을 포함함.
이에 대한 자세한 설명은 RemoteAPI의 stats항목을 참고하기 바람.
```
 
** system df

system df 명령어는 docker에서 사용하고 있는 image, container, local volume의 총 개수 
및 사용 중인 개수, 크기, 삭제함으로써 확보 가능한 공간을 출력함. 다음 예시에서 총 image의
개수는 2개이고, 이 가운에 사용 중인 image는 1개이며, image가 차지하는 공간이 189MB임을
확인할 수 있음. RECLAIMABLE 항목은 사용 중이지 않은 image를 삭제함으로써 확보할 수 있는
공간을 의미함.

```
# docker system df
```

사용 중이지 않는  container와 volume은 각각 docker dontainer prune, docker volme prune으로
한꺼번에 삭제할 수 있음. docker image prune 명령어를 사용하면 사용 중이지 않은 
dangling image를 삭제함.

```
(참고) dangling image는 docker images 명령어의 출력결과에서
이름이 <none>:<none>인 image를 뜻함.

 # docker images
```

#### 2.5.4.3 CAdvisor

CAdvisor는 google이 만든 container monitoring 도구로, container로서 간단히 설치할 수 있고
container 별 실시간 자원 사용량 및 docker monitoring 정보 등을 시각화해서 보여줌.
CAdvisor는 open source로서 Github에서 source code로 사용할 수 있으며 
docker hub image로도 배포되고 있음.

CAdvisor를 사용하기 위해 다음 명령어를 입력함. CAdvisor는 container의 agent의 형태로
docker monitoring에 필요한 자료를 수집함.

```
# docker run \
  --volume=/:/rootfs:ro \
  --volume=/var/run/:/var/run:rw \
  --volume=/sys:/sys:ro \
  --volume=/var/lob/docker/:/var/lib/docker:ro \
  --publish=8080:8080 \
  --detach=true \
  --name=cadvisor \
  google/cadvisor:latest

```

image pull 및 container 생성이 완료되면 host의 8080번 port로 CAdvisor dash board에
접근할 수 있음. web브라우저에서 아래와 같은 화면을 확인할 수 있다면 
CAdvisor container가 정상적으로 설치 된 것.


![그림2.80](https://github.com/ALGOSOPT/DockerConcept/blob/master/picture/2.80.jpg)

CAdvisor에서는 생성된 모든 container의 자원 사용량을 확인할 수 있을 뿐만 아니라
docker daemon의 정보, 상태, host의 자원 사용량까지 한 번에 확인할 수 있음.

IP 주소와 8080번 port로 접속했을 때 확인할 수 있는 page는 host의 process, 자원의 사용량 등을 보여줌

[Subcontainer] 항목의 [/docker]를 클릭하면 docker daemon의 정보, container의 목록을 보여주는 page로
이동함 [Subcontainer] 항목의 container 이름을 click하면 container의 자원 사용률도 실시간으로 확인할 수 있음.

```
(참고) CAdvisor의 dashboard는 60초간 monitoring 정보만 보여주지만 influxDB나 Premetheus
등과 같이 사용하면 장기간의 monitoring 정보를 수집하고 분석할 수 있음.
```

![그림2.81](https://github.com/ALGOSOPT/DockerConcept/blob/master/picture/2.81.jpg)

여기서 짚고 넘어가야 할 부분은 CAdvisor를 생성할 때 option으로 설정한 host volume 공유(-v) 부분.
CAdvisor는 이렇게 많은 정보를 어떻게 사용자에게 보여줄 수 있는 걸까?

답은 docker daemon의 정보를 가져올 수 있는 host의 모든 directory를 CAdvisor container에 volume으로서
mount했기 때문임. /var/run에는 docker를 local에서 제어하기 위한 unix socket이 있고,
/sys에서는 docker container를 위한 cgroup 정보가 저장되어 있으며 /var/lib/docker에는 docker의
container, image 등이 file로 존재함.


```
(참고)
/sys/fs directort에는 docker container에게 격리된 자원을 제공하기 위한
cgroup directory사 존재.
이 directory에는 CPU, memory 등 container가 격리되어 할당받아야할 자원이 
directory로서 다시 존재하고, 그 하위 directory에는 각 container에게 
할당된 cgroup 정보가 존재함.

# ls /sys/fs/cgroup/(cgroup 할당 자원)/docker/(container ID)/...

다음 예에서 08d709...라는 ID를 가진 container의 memory cgroup 정보가 들어있는 
directory를 볼 수 있음.
관심이 있으면 한 번 cgroup directory를 자세히 살펴보는 것을 권장.

# ls /sys/fs/cgroup/memory/docker/
```

그러나 CAdvisor는 단인 docker host만을 monitoring할 수 있다는 한계가 있음.
여러 개의 host로 docker를 사용하고 있으며, 이를 기반으로 PaaS(Platform as a Sevice)
같은 docker cluster를 구축했다면 CAdvisor가 용도에 맞지 않을지도 모름.


#### 2.5.4.4 Docker Data Center(DDC)의 Univeral Control Plane(UCP)


여러개의 docker host로 이루어진 docker cluster를 monitoring하는 대부분의 
도구는 유료.
무료로 사용할 수 있는 도구도 있지만 실제 운영 환경을 위한 완벽한 지원과
기능을 기대하려면 상용 솔루션을 이용하는 편이 나을 수도 있음.

유료로 사용하는 솔루션에는 Universal Control Plane(UCP), Datadog,
New Relic, Docker-Scout 등이 있으며 무료로 사용할 수 있는 방법으로는
Shipyard, Graphite, Rancher Lab 등이 있음.

```
(참고) 이 책의 후반부에서 설명할 Rancher Lab은 cluster monitoring뿐
아니라 PaaS를 구성하기 위한 기능들을 제공하는 무료 clustering 솔루션임.
```

Univercal Control Plane(UCP)는 Docker가 제공하는 Docker host Clustering 
Monitoring 도구이며, 여러 개의 host를 등록하고 Container, image, resource 사용량 
등을 확인할 수 있음. 한마디로 여러개의 Docker host를 관리하기 위한 종합 solution이라고
생각하면 됨. Container, host 관리, 볼륨, 네트워크 생성과 삭제 등 docker를 제어할 수 있는
대부분의 기능을 사용할 수 있기 때문에 대규모 cluster를 세세하게 관리해야하는 조직에
적합한 monitoring 방법이라고 할 수 있음

그러나  가장 중요한 사실은 UCP는 유료라는 점. 또한 UCP는 EE(Enterprise Edition) docker
engine을 대상으로 하고 있음. 하지만 UCP를 간단히 test하는 용도라면 일반 Docker Engine도 
사용할 수 있으며, UCP는 30일 체험한을 먼저 사용해 볼 수 있으므로 조직에서
대규모 Docker Clustering을 고려하고 있다면 UCP를 무료로 사용해보고 난 뒤 결정해도 늦지 않음.

---

### 2.5.5 Remote API 라이브러리를 이용한 Docker 사용

이 전의 2.5.3.1절 Docker Daemon 제어하기 : -H 에서 Docker를 원격으로 제어하는 방법을
살펴 봄. 필요에 따라서는 -H 옵셔늘 원격의 Docker Daemon을 제어하기 위해 사용하는 것도
좋은 방법이 될 수 있지만 Application이 수행해야 할 작업이 많거나 Application 초기화 등에
복잡한 과정이 포함되어 있다면 Docker를 제어하는 Library를 사용해 이를 좀 더 쉽게
해결할 수 있음.

Docker를 제어하고 싶을 경우 일일이 Remote API에 대한 요청을 source code로 제작할 필요없이
이미 Remote API를 wrapping해서 사용하기 쉽게 만들어 놓은 Library를 이용할 수 있음.

이러한 Library 목록은 Docker SDK 페이지에서 확인할 수 있음. Docker에 기반 언어인 GO는 물론
C#, C++, Python, Dart, PHP, java 등 많은 Library를 Open source로 사용할 수 있음.

그림 2.83

이 가운데 친숙한 언어를 선택해 라이브러리를 사용하면 됨. 이 번절에서는 Java, Python Library를 사용하는 
방법에 대해 알아보자~

#### 2.5.5.2 Python Library

pip로 Docker Library를 설치
```
# pip install docker
```

라이브러리가 정상적으로 설치 되어는지 확인하기 위해 python shell을 실행해 
Docker Client를 실행함.
다음 예는 Unix Socker에 연결해 Docker Engine의 정보를 출력함. Remote API로 Docker Client
를 사용하려면 base_url에 https://192.168.0.100:2375와 같이 docker daemon에
접근할 수 있는 IP주소와 port번호를 입력함

```
# python 

>>> import docker
>>> client = docker.DockerClient(base_url='unix://var/run/docker.sock')
>>> clinet.info()
```

HTTPS를 사용하도록 TLS 보안이 적용된 Docker Daemon에 연결하려면 다음과 같이 TLSConfig 객체를 
생성해 사용함.

```
# vi tls_docker_connect.pu

import docker
tls_congif = docker.tls.TLSConfig(
  client_cert=('/root/.docker/cert.pem', '/root/.docker/key.pem')
)

client = docker.DockerClient(base_url='unix://var/run/docker.sock', tls = tls_config)
print(client.info());
```

```
# python tls_docker_connect.u
```

```
(참고) 스스로 발급한 증명서를 사용해서 InsecureRequestWarning에 관련된 경고가
출력된다면 TLSConfig 객체를 생성하기 전에 다음과 같은 내용을 추가해서 경고를
출력하지 않게 할 수 있음
import requests.packages.urllib3
requests.packages.urllib3.disable_warnings()
```

Docker Client 객체를 생성했다면 이를 이용해 Docker Engine을 제어할 수 있음.
다음 예시는 host의 80/tcp port를 container의 80번 포트에 연결하는 NginX
container를 Detach 상태로 생성하고 시작하는 예.
즉, 다음 내용은 Docker run -d -p 80:80 nginx와 동일
```
# vi run_nginx_container.py

import docker 

client = docker.DockerClient(base_url='unix://var/run/docker.sock')
container = client.containers.rn('nginx',
                                  detach=True,
                                  ports={'80/tcp':80})
print("Created container is : {}, {}".format(container.name, container.id))
```

```
# python run_nginx_container.py
```
