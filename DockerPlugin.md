# 6. Docker Plugin
---

## 6.1 Docker Plugin이란? 

Docker Engine은 그 자체로도 많은 기능을 제공함. Docker Engine에 원하는 기능이 없더라도
Docker swarm mode, compose 등 docker와 관련된 여러 프로젝트와 같이 사용함으로서
대부분의 문제를 해결할 수 있음.

그러나 docker container로 개발하는 영역이 더욱 커져 상업적 단계까지 고려해야한다면 docker
project 만으로는 해결할 수 없는 문제가 발생하기도 함. 이를 해결하기 위해서 특정
vendor나 클라우드 platform의 기능을 사용해야할 수도 있고, 여러 호스트에 생성된
container를 관리하기 위해 PaaS(Platform as a Sevice를 구축해야할 수도 있음.

Docker Project에서 원하는 기능을 찾을 수 없다면 docker plugin을 사용하는 것을 권장.

__docker plugin__ 은 docker에서 공식적으로 제공하지 않는 기능을 제공함. multihost networking
이나 자유로운 network topology를 구성하기 위한 network plugin, PaaS 환경에서 유동적으로 
생성되고 삭제되는 container에게 persistent storage 를 제공하기 위한 volume plugin 등이 그 예.
docker가 open source인 만큼, docker plugin 또한 open source로 누구나 자유롭게 사용할 수 있으므로
목적에 맞게 plugin을 선택해 사용하면 됨. 

단, docker plugin은 사용자가 직접 설치해야하며, plugin마다 사용하는 방법이 모두 다름

```
docker engine 1.12 버전 이후에서 플러그인을 관리하기 위한 docker plugin 이라는
명령어를 사용할 수 있지만, 모든 플로그인이 이 명령어를 지원하는 것은 아님, 이에 대한
자세한 정보는 docker plugin 공식 문서를 참고하길 바람.
```

Docker Plugin의 단점은 docker가 공식적으로 제공하고 개발하는 것이 아니므로 
개발 여부가 불투명 할 수 있다는 것. 크고 조직적인 단체가 개발하는 plugin은 빠르게 
개발되고 문서도 많이 작성되지만 작은 단체나 개인이 유지보수하는 플러그인은 
개발 문서가 없어 사용하기 어려울 수도 있을 뿐더라 언제든지 개발이 중단될 수도 있음.
따라서 opensource plugin을 선택할 때는 개발이 활발히 이루어지고 있는지, 
플로그인 사용자가 얼마나 많은 피드백을 하고 있는지, 기능 업데이트는 정기적으로
이뤄지는 지 등을 고려하는 것이 좋음

---
## 6.2 Docker Volume Plugin

Docker plugin 중 Volume에 관련된 plugin은 대부분 PaaS 환경을 위한 Container의 
persistant storage를 구성하는 것을 목표로 함. 이 것은 docker server cluster를 구축했을 때
반드시 직면하는 문제중 하나로서 container가 어느 host에 생성되도 같은 data를
가져야 한다는 것을 전제로 함.

하나의 docker server만 존재할 때의 data 저장방식은 docker volume 명령어로 docker volume을
생성하거나 run 명령어에서 -v 옵션을 사용해 host와 파일을 공휴하는 것. 
그러나 2.3.3.7절 스웜 모드 볼륨 사용하기 에서 설명했듯이 docker server cluster를 구축하면
container가 어느 서버에 생성될지 모르므로 이 방법을 사용하는 것은 문제가 있음.
모든 host에 같은 data를 준비할 수 없으며, 설령 그렇게 했더라도 data가 변경되었을 때 어떻게 
동기화할 것인가에 대한 문제가 남아 있음.
그러므로 멀티 호스트 환경에서 persistance storage를 구성하는 일반적인 방법은 외부의 저장공간을 
container에 mount하는 것.

그림 6.1

그렇다면 외부 저장 공간을 어떻게 확보할 것인가도 문제가 됨. 이를 해결하는 가장 간단한 방법은
각종 클라우드 공급자가 제공하는 Block Storage나 object storage를 사용하는 것.
일반적으로 많이 사용하는 cloud platform으로는 AWS, Azure, GCE(Google Compute Engine)이 있음.

Op-premise 환경에서 저장공간을 사용하려면 Openstack의 Cinder, software defined storage인
ceph, glusterFS 또는 NFS 를 사용할 수도 있음.
즉, container에 persistant storage를 추가하려면 어떤 플랫폼을 사용하느냐와 상관없이 Docker
Plugin을 사용해야함.


### 6.2.1 sshFS

모든 플러그인이 docker plugin 명령어를 지원하는 것은 아니지만 일부 plugin은 docker plugin 명령어로
쉽게 설치하고 사용할 수 있음. docker plugin을 사용하는 대표적인 플러그인으로 sshFS가 있음.

그림 6.2

#### 6.2.1.1 sshFS 사용

sshFS가 정상적으로 동작하는지 확인하려면 최소한 2대의 server가 필요함. 하나는 SSH 서버가 설치된
SSH로 접속할 수 있는 서버로, Volume의 파일을 저장하는 역할을 함. 달느 하나는 sshFS를 통해 SSH서버
에 접속해 Volume을 마운트하고 파일을 읽고 쓰는 작업을 수행하는 docker server.
즉 실제로 파일이 저장된 것은 docker server가 아닌 SSH 서버.

앞서 언급한 것처럼 sshFS는 docker plugin 명령어로 쉽게 설치할 수 있음. 볼륨을 사용하려는
docker server에서는 docker plugin install 명령어로 sshFS 플러그인을 설치함. 플로그인은
host의 각종 권한이 필요할 수도 있으므로 구한 부여 여부를 묻는 문구가 출력되면 y를 입력함


```
root@sshfs-client:~# docker plugin install vieux/sshfs

```

설치된 sshFS 플로그인 docker plugin ls 명령어로 확인할 수 있음.

```
# docker plugin ls
```

실제 파일이 저장될 서버인 SSH 서버에서 sshd가 동작 중인지 확인함.

```
# ps aux | grep sshd
```

sshFS에 mount할 서버에서 다음 명령어로 sshFS 볼륨을 생성함. -o는 볼륨의 옵션을
설정하는 옵션으로 sshcmd에는 해당 볼륨에 접속하기 위한 정보를, password에는 계정의
비밀번호를 port에는 SSH port를 입력함. 아래의 예제는 IP주소가 192.169.0.100인 host에
sshfs-server 계정의 mypassword 라는 비밀번호로 접근해 /home/sshfs-server 디렉터리를
persistent storage로 사용한다는 것을 의미. 이 옵션의 값은 사용환경에 맞게 적절하게 사용해야함

```
# docker volume create -d viex/sshfs \
-o sshcmd=sshfs-server@192.168.0.100:/home/sshfs-server \
-o password=mypassword \
-o port=22 \
sshvolume
```

volume을 생성했다면 sshFS에 마운트하려는 docker server에서 이 볼륨에 접근해 봅시다.
다음 명령어로 위에서 생성한 sshvolume을 사용하는 container를 생성함.

```
# docker run -it --name sshfs_container \
-v sshvolume:/test busybox
```

container의 /test 디렉터리에 sshFS 볼륨을 마운트 했으므로 /test에서 파일을 읽고 쓰면 IP주소가
192.169.0.100인 서버의 /home/sshfs-server 디렉터리에 파일을 읽고 쓰는 작업이 동일하게 반영됨

```
/ # touch /test/myfile
/ # ls /test/

```

```
root@sshfs-server:~# ls /home/sshfs-server

```


## 6.3 Docker Network Plugin

Docker Network Plugin은 대부분 멀티 호스트 환경의 PaaS를 위한 
VXLAN, network 분리 등을 목적으로 함. 물론 docker swarm 모드의 Ingress Network를 사용하거나
Zookeeper, Etcd 같은 외부 key-value 저장소를 사용해 Global network를 구축하는 등
docker 자체의 기능으로도 VXLAN을 구축할 수 있음. 그러나 이러한 docker의 기능이 사용 목적과 
일치하지 않거나 쿠버네티스 혹은 메소스와의 통합, 또는 확장된 network 기능을 고려하고 있다면
docker network plugin을 사용해서 network topology를 구성해보는 것도 좋음.


### 6.3.1 Weave

Weave는 Weaveworks 에서 개발한 network plugin으로서 멀티 호스트 환경에서의 
Docker Container의 network를 구성하는 기능을 제공함. weave는 외부에 분산 코디네이터를
설치할 필요가 없고 위브 host가 mesh topolgy를 구성해 data를 동기화함.
따라서 weave를 사용하는데 필요하는 것은 단지 여러 개의 docker server뿐.

다만 weave는 다른 플러그인과 다르게 host에서 직접 plugin agent가 실행되지 않고 container
의 형태로 agent가 구동되는 방식.

#### 6.3.1.1 Weave Cluster 구축

이번 절에서 사용할 cluster의 서버 IP는 다음과 같음. Weave를 test 하려면 적어도 3개의
host를 사용하는 것이 좋음. 물론 모든 host에는 docker engine이 설치 되어 있어야 하며,
VXLAN 통신을 위해 6783/tcp, 6783/udp, 6784/udp 포트는 개방되어 있어야 함.

```


```
