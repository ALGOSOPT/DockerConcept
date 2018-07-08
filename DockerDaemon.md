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

2.3절 '이미지 다루기' 에서 container와 image가 어떻게 동작하는지 알아봄. 
