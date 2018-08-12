# 7. Docker Solution

지금까지 docker와 관련된 여러 project와 plugin을 살펴보았당
가장 기초가 되는 container를 다루기 위해 docker engine을 먼저 알아보았고
docker engine을 쉽고 편리하게 사용할 수 있는 docker compose, docker swarm,지만, 
docker swarm mode, docker machine과 몇몇 plugin을 알아 보았다.
이러한 project를 먼저 소개한 이유는 docker plugin을 제외하면 모두 docker
project라는 큰 틀안에서 공식저으로 개발하고 제공되는 project들이기 때문.



그러나 docker container를 다루는 방법이 docker에서 공식적으로 제공하는 방법만 있는
것은 아님.
현재는 docker project가 점점 더 성장함에 따라 docker가 공식적으로 제공하는 container
오케스트레이션 방법이 많아졌지만, 예전에 docker container로 PaaS 환경을 직접 구성하기 어려웠을 때는
docker 외의 project로 PaaS환경을 구성하기도 함. 그리고 이러한 project는 현재에 이르러 완전한 
container ochestration 생태계로 자리 잡고, docker project와는 다른 길을 걷고 있음.


PaaS 환경으로서 Docker Container를 사용하기 위한 project는 여러 가지가 있음.
PaaS 솔루션으로서 완벽에 가까운 container 배포 환경을 제공하는 redhat의 OpenShift, 
PaaS 환경을 구축하기 위한 여러 기능을 제공하는 구글의 Kubernetes, 다양한 종유의 resource를 사용할 수 있는
범용 PaaS 플랫폼인 Mesosphere, Container에 최적화된 Infra를 제공하는 운영체제인 CoreOS,
여러 솔루션을 쉽게 사용할 수 있는 platform을 제공하는 Rancher 등 많은 project가 더 나은 PaaS 환경을
만들 수 있도록 노력하고 있음.

PaaS와 Container Ochestration에 관심이 많다면 이러한 solution을 살펴보는 것을 궈장.


이번 장에서는 일부 solution의 개념과 간단한 사용법을 다룸. 이를 통해 docker의 공식 project외에도
docker container를 사용할 수 있는 방법이 많다는 것과 그 것들이 지금도 계속 발전해 나가고 있다는 것을 
알 수 있음.

---

## 7.1 Kubernetes.

그리스어로 "조타수" 라는 뜻의 Kubernetes는 Google에서 open source project로 내놓은 container 
ochestration 도구 .

오랫 동안 Google이 사용해온 Container 운용 경험을 녹인 도구라고 일컬어지고 있으므로 많은 사람에게
성능과 안정성 면에서 신뢰받고 있음. Kubernetes는 일반적으로 Web Service를 Container로서 제공하기 
위해 사용되며, 다른 Ochestration 도구와 마찬가지로 여러 개의 docker host를 하나의 resource pool로
사용할 수 있게 cluster를 구성함.

여기서 한 가지 짚고 넘어가야할 점은 kuberetes는 그 자체로는 PaaS 환경을 제공하지 않으며,
PaaS 환경을 구성할 수 있는 안정적인 환경을 만들어준다는 것. 그래서 일반적으로 Kubernetes를 기반으로 플랫폼
을 구성하는데  대표적인 예가 redhat의 openshift.

![그림7.1](https://github.com/ALGOSOPT/DockerConcept/blob/master/picture/7.1.jpg)


openshift는 container 기반의 web service를 제공하는 PaaS로서 PHP, JBoss 등 각종 Web service의
docker image를 선택하고 S2I(Source To Image) 기능을 이용해 사용자의 source code에서 Web service를
생성하는 기능 등을 제공함. 즉,  Kubernetes를 기반으로 하되 추가적인 플랫폼 기능을 구현해 PaaS로 사용하는
셈.


Kubernetes의 기능은 Web service 초점을 맞추고 있으므로 Kubernetes는 Docker Engine의 swarm mode와
성능 및 기능 측면과 자주 비교되곤 함. 뒤에서 다시 살펴보겠지만 ingress, NodePort, Container 레플리카
등 Kubernetes의 기능은 swarm  mode와 유사한 것이 많음. 그러나 Kubernetes는 swarm mode가 출시되기 
이전부터 있었으며, 사용자가 customizing할 수 있는 요소가 swarm mode보다 많은.
물론 swarm mode도 앞으로 많은 기능이 업데이트 되겠지만 어느쪽을 선택할지는 사용 목적과 각 tool의 장단점을 꼼꼼히
살펴본 뒤에 결정하는 것이 좋음. 


Docker Engine만으로 간편하게 Web Service Container를 분사해서 사용하고 싶다면 Swarm mode를,
시간과 비용이 충분하고 좀 더 많은 기능이 필요하다면 Kubernetes를 선택하는 것도 하나의 전략이 될 수 있음.

---

### 7.1.1 Kubernetes 설치

Kubernetes를 사용하는 간단한 방법은 각종 Cloud service에서 Kubernetes가 설치된 Cluster를
받는 것. Kubernetes cluster는 Google Compute Engine(GCE), Azure에서 간단히 생성해
사용할 수 있으며, AWS에서도 script를 사용해 쉽게 설치할 수 있음. 그 뿐만 아니라
여러 개의 server나 가상 머신이 있다면 직접 설치할 수도 있음.

이번 절에서는 Minikube와 Google Compute Engine의 Kubernetes Cluster를 생성하는 방법을
설명하지만 다른 설치 방법에 대해서는 Kubernetes 설치 문서를 참고하기 바람.

```
(참고) Kubernetes는 Cloud platform에서만 사용할 수 있는 기능이 일부 포함되어 있으므로
가급적 Cloud Platform에서 사용하는 것을 권장함.
```

#### 7.1.1.1 Minikube

Minikube는 local에서 가상 머신 1개를 생성해 자동으로 Kubernetes를 사용할 수 있는 환경을 제공함.
이 경우 Kubernetes의 기능을 간단히 test할 수 있다는 장점은 있지만, 실제 운영 환경에
Minikube를 적용하기가 힘들뿐더러 Kubernetes의 기능을 출분히 활용할 수 없으므로
가능한 한 여러 대의 server로 kubernetes를 구성하는 것이 좋음.

Minikube는 맥 OS X, window, linux에서 가상 머신을 생성해 사용하기 때문에 virtual box나
KVM 같은 가상화 환경이 준비되어 있으여 함. 이번 절에서는 linux에서  virtual box를 이용해 
Minikube를 설치하는 방법을 설명함.

__1.virtual box 설치
가상 머신을 사용하려면 먼저 virtual box를 설치해야함. ubuntu와 debian 계열 리눅스 에서는
apt-get install로 쉽게 설치할 수 있음.

```
root@ubuntu:~# apt-get install virtualbox
```

__2.minikube, kubectl 다운로드
운영체제와 상관없이 다음 명령어로 kubectl, minikube를 내려 받음. minikube, kubectl은
kubernetes가 지속해서 개발됨에 따라 version이 올라갈 수 있으며, 2017년 2월 기준으로 minikube는
0.15.0 버전을, kubectl은 1.5.1 version을 사용할 수 있음.

```
이건 책보고 하자 316p
```

__3. minikube 가상 머신 설치
virtualbox가 설치되면 다음 명령어로 minikube 가상 머신을 생성할 수 있음.

```
# minikube start
```

kubectl get nodes 명령어로 가상 머신에서 kubernetes가 작동 중인지 확인함.

```
# kubectl get nodes
```

#### 7.1.1.2

GCE(Google Compute Engine)은 Container 기반의 Web Application service를 쉽게
관리하고 배포할 수 있는 Google Container Engine을 제공함.
google container engine은 kubernetes와 docker engine이 설치된 일정 개수의 가상 머신을 생성하고
kubernetes를 이용해 각 가상 머신에 container를 생성해 사용하는 방식으로 동작.
즉, 자동으로 kubernetes가 설치되어 있므으로 kubernetes를 일일이 설치할 필요가 없음.

```
(참고) 대부분의 cloud service는 최초 사용에 한해 무료 평가판을 제공함.GCE도 60일 동안
300달러를 사용할 수 있는 credit을 제공하고 있음.
```


1. Virtual Machine Cluster 생성
Google Container Engine Site에 접속해 login 하면 다음과 같은 화면을 볼 수 있음.
Container Cluster 만들기를 클릭해 새로운 cluster를 생성하는 화면으로 이동함.

![그림7.2](https://github.com/ALGOSOPT/DockerConcept/blob/master/picture/7.2.jpg)

cluster를 생성하는 화면에서 [크기]항목에 가상 머신의 개수를 입력해
kubernetes  cluster의 크기를 설정할 수 있음. 이 예제에서는 그림 7.3과 같이 [크기]항목에
3을 입력한 뒤 하단의 [만들기]를 클릭해 kubernetes cluster를 생성함.

![그림7.3](https://github.com/ALGOSOPT/DockerConcept/blob/master/picture/7.3.jpg)



2. Cluster 접속 정보 확인
Cluster 생성이 완료되면 그림 7.4와 같이 생성 완료 표시가 나타남. Cluster 생성이 완료된 후 
해당 cluster의 [연결]버튼을 클릭함.

![그림7.4](https://github.com/ALGOSOPT/DockerConcept/blob/master/picture/7.4.jpg)


gcloud container cluster ... 명령어를 복사함. 이 명령어는 생성한 kubernetes cluster에 
접근하는 데 사용함.

![그림7.5](https://github.com/ALGOSOPT/DockerConcept/blob/master/picture/7.5.jpg)



3. cluster에 접속

오른쪽 위의 command 라인 아이콘을 click해 kubectl을 사용할 수 있는 google cloud shell을 활성화함.

![그림7.6](https://github.com/ALGOSOPT/DockerConcept/blob/master/picture/7.6.jpg)

브라우저 하단에 terminal 형태의 shell이 활성화되면 위에서 복사한 명령어를 shell에 입력함


![그림7.8](https://github.com/ALGOSOPT/DockerConcept/blob/master/picture/7.8.jpg)


kubectl get nodes 명령어로 kubernetes cluster에서 사용 가능한 server를 확인함.

그림 7.8


----

#### 7.1.1.3 kubeadm

Kubernetes는 On premise 환경에서도 Kubernetes를 쉽게 설치할 수 있는 kubeadm이라는 관리 module을
제공함. kubernetes의 설정이나 각종 component를 일일이 다룰 필요 없이 간편하게 kubernetes cluster를 사용할 수 
있다는 장점은 있지만, master와 분산 코디네이터의 다중화 등을 비롯란 여러 세부 기능들이 아직 개발 중이라서
실제 운영 환경에 적용하기에는 적합하지 않을 수 있음.


이번 예제에서는 우분투 16.04를 기준으로 설치 과정을 설명함. kubernetes의 기능을 제대로 사용하려면
최소 3개의 가상 또는 물리 머신이 필요함. 이번 예제에서 사용할 machine의 host명과 IP는 다음과 같당


```
kube-master01 192.168.0.100
kube-agent01 192.168.0.101
kube-agent02 192.168.0.102
```

```
(참고) 도중에 설치가 제대로 진행되지 않아 다시 설치해야 할 때는 다음 명령어로
kubernetes를 초기화 한 뒤 다시 진행할 수 있음. 이 며령어는 설치가 완료된 뒤에도
kubernetes를 삭제하는 용도로 사용할 수 있음.

# kubeadm reset
```


1. kubernetes 저장소 추가
kubernetes를 설치할 모든 node에서 다음 명령어를 차례대로 입력해 kubernetes 저장소를 추가함.

```
하.. 이 것도 책보디 320p
```

2. kubeadm 설치

kubeadm은 docker container를 이용해 kubernetes를 빌드하는 과정이 포함되어 있으므로 docker
engine이 미리 설치되어 있어야 함. 따라서 모든 node에서 docker engine을 먼저 설치함.

```
# wget -q0- get.docker.com | sh
```

3. kubernetes cluster 초기화
kubernetes는 cluster를 제어하는 master node와 container를 구동하는 kubernetes node로 나뉨.
master node로 사용할 host에서 다음 명령어로 cluster를 초기화 함.

--api-advertise-address 옵션의 인자에 다른 node가 master에게 접근할 수 있는 IP주소를 환경에 맞게
입력함. 다음 예제는 kube-master01 host에 접근할 수 있는 IP주소가 192.168.0.100 인 경우

```
root@kube-master01:/# kubeadm init --api-advertise-address 192.168.0.100
```

```
(참고) 최선 버전의 docker는 kubeadm과 호환되지 않을 수 있음. 다음과 같은 오류가 발생하면
부록B를 참조해 버전을 낮춰보길 바람.
```

빌드가 완료도면 다음과 같은 출력 결과를 확인할 수 있음. 맨 마지막에 출력된 명령어인 kubeadm join ...을 복사해 master
를 제외한 노드인 kube-agent01, kube-agent02에서 실행함. kubernetes node에 연결하는 과정에서 9898, 6443 번 포트를
사용하므로 master node에서 이 port들을 미리 열어둬야 함.

```
321p 밑.
```

```
만약 x509 에러가 출력되면 ntp를 설치
# apt-get install ntp
```


4. container network 에드온 설치

container 간 통신을 위해 network 애드온으로서 flannel, weave 등 여러 가지를 사용할 수 있지만
이번 예제에서는 weave를 사용함. master node에서 다음 명령어로 weave를 설치할 수 있음.
weave의 통신에는 6783/tcp, 6785/udp, 6784/udp port를 사용하므로 모든 node에서 이
port들을 미리 열어둬야 함

```
:/# kubectl apply -f https://git.io/weave-kube
```

설치가 정상적으로 완료되었는지 확인하기 위해 다음 명령어로 master node에서 연결된 node를 확인함.

```
:/# kubectl get nodes

```

kube-dns, weave가 정상적으로 구동하고 있는지 확인하기 위해 다음 명령어로 agent 목록을 확인함.


```
:/# kubectl get pods --namespace kube-system
```


---

### 7.1.2 Kubernetes 구조

Kubernetes cluster의 서버를 전체 cluster를 관리하기 위한 master node와 실제로
container가 생성되는 kubernetes node로 나뉘며, kubectl이라는 command 라인 interface를 
master node에서 사용함으로써 kubernetes cluster를 제어함.

kubernetes cluster를 동작시키기 위한 정보 저장, data 동기화 등을 담당하는 분산
코디네이터인 etcd가 구동되고, kubelet 이라는 이름의 process가 각 kubernetes node를
관리하기 위한 agent로서 동작함.


![그림7.9](https://github.com/ALGOSOPT/DockerConcept/blob/master/picture/7.9.jpg)

각 Kubernetes 노드는 1개 이상의 container를 Pods라는 단위로 생성함.

각 pods의 container는 pods의 IP주소를 가짐으로써 같은 network를 사용함.

이러한 pods는 Replica Sets에 의해 생성되고 관리됨. Kubernetes의 Pods는
Replica set에 의해 자동으로 생성되고 장애가 생기면 자동으로 재생성되며, 롤링 업데이트를 수행함.

즉, Replica Set은 Pods를 종합적으로 관리하기 위한 단위라고 볼 수 있음.

```
Kubernetes는 Replica Set 뿐만아니라 Replication Controller 라는 단위로 Pods를 관리할 수도
있음. 하지만 Replica set은 Replication Controller의 상위 개념이며 더 많은 기능을 지원하므로
이번 예제에서는 Replica Set을 기준으로 설명함.
```


Deployment는 Replica Set과 Pods를 선언적으로 정의하는 개며으로서, 일반적으로 Container(Pods)를 생성
할 때 Deployment를 사용함.


Deployment를 생성해 Pods가 생성되었다고 해도 바로 외부에 노출되어 서비스 되는 것은 아님.
Pods를 어떻게 외부에 노출할 것인지, 어느 port를 노출할 것인지 등을 정의해야함. Service는
Deployment에서 생성된 Pods를 외부에 노출하는 역할을 담당함. Pods를 외부에 노출하면 별도의
설정을 하지 않아도 사용자가 Pods에 접근할 때 Load Balancing된 서비스를 받게 됨.

---

### 7.1.3 Kubernetes 사용

#### 7.1.3.1 Deployment 생성

앞에서도 사용했지만 먼저 cluster에 사용 가능한 node를 확인하기 위해 kubectl get nodes 명령어를
실행합니다.

```
# kubectl get nodes
```

Minikube로 Kubernetes를 설치했다면 1개의 node를, 각종 cloud platform에서 
kubernetes를 설치 했다면 여러 개의 node를 사용할 수 있을 것.
이번 예제에서는 3개의 node로 구성된 cluster를 기준으로 설명함.


가장 먼저 deployment를 생성해 봅시다. docker와 비슷하게 run명령어로 deployment를 생성할 수 있음.
kubectl run 뒤에 deployment의 이름을 입력하고 --image를 설정한 뒤 --replicas로 복제될 
Pods의 개수를 설정함. 외부로 노출할 Container의 Port는 --port로 설정할 수 있음.


```
# kubectl run mydeployment --image=alicek106/composetest:balanced_web \
--replicas=3 \
--port=80
```

deployment를 포함한 kubernetes의 resource는 kubectl get [resouce 종류] [이름]의 형식으로
정보를 출력할 수 있음. 앞에서 생성한 deployment의 정보를 출렷함.

```
# kubectl get deployment mydeployment
```

deployment는 기본적으로 pods(container)를 생성하기 위한 단위이므로 생성된 Pods의 개수를 
출려함. 위 출력 결과에서 총 3개의 복제된 Pods가 생성된 것을 알 수 있음.


이번에는 Pods와 Replica Set을 출력해보자. 앞에서 설명한 것처럼 Replica Set은 Pods를 관리하기 위한
단위임. kubectl get [resource 종류]와 같이 입력해 모든 resource의 정보를 출력할 수 있음.
STATUS가 'ContainerCreating'이면 image를 받고 있는 중이므로 잠시 기다려야 함.

```
# kubectl get pods
```

```
# kubectl get replicasets
```

```
pods는 po로 줄여 쓸 수 있고, replicaset도 rs로 줄여 쓸 수 있음.
즉, 위의 두 명령어는 다음과 동일함.

# kubectl get po
# kubectl get rs
```

생성된 리소스는 'kubectl delete [리소스 종류] [이름]' 형식으로 삭제할 수 있음.
'kubectl delete deployment [이름]' 을 입력하면 pods와 replicaset에 한번에 삭제됨.

```
# kubectl delete deployment mydeploment
```


---

#### 7.1.3.2 service create

이번 절에서는 다른 방법으로 deployment를 생성해 보자. kubectl run 명령어로 deployment
를 생성할 수도 있지만, yaml 파일에 각 pods의 설정 정보를 명시해 deployment를 생성하는 것이
일반적. 다음 내용을 my-deploy.yaml 파일로 저장함

```
예제 7.1 my-deploy.yaml 파일

# vi my-deploy.yaml

apiVersion: extension/v1beta1
kind: Deployment
metadata:
  name: my-deploy-name
spec:
  replicas: 3
  template:
    metadata:
      labels:
        app: my-deploy
    spec:
      containers:
      - name: mycontainer
        image: alicek106/composetest:balanced_web
        ports:
        - containerPort: 80
```

```
(참고) kubernetes의 yaml 파일은 docker compose에서 사용한 yml 파일과 같은 언어
따라서 파일 확장자는 yaml, yml 둘 중 어떤 것을 사용해도 상관 없음.
```

위 예제는 7.1.3.1절 Deployment 생성에서 Deployment를 생성할 때 사용한 run 명령어를
변환한 것임. kind 항목에서 resource의 종류를 Deployment로 정의하고 metadata에 이름을 지정했습니다.

spec 항목은 생성될 포드의 설정을 정의하는데, 여기서 눈여겨봐야 할 항목은 labels의 app:my-deploy임.

라벨은 키-값의 쌍으로 이루여 있으며, 생성된 포드의 라벨에 app의 data를 my-deploy로 설정.
라벨의 키와 값은 아래에서 생설할 서비스에서 다시 사용되므로 기억해두자.

그 밖의 설정은 이전과 같음.


위 파일을 deployment로 생성하려면 kubectl create -f [파일이름]을 입력함.


```
# kubectl create -f my-deploy.yaml
```

그러나 아직은 deployment를 통해 포드를 생성만 했을 뿐 외부에 노출하지 않았음.
포드를 외부에 노출하려면 서비스를 생성해야함. 마찬가지로 서비스도 yaml 파일로 작성해
생성할 것이므로 다음 내용을 my-service.yaml 파일로 저장ㅎ.ㅁ

```
예제 7.2 my-service.yaml
$ vi my-service.yaml

apiVersion: v1
kind: Service
metadata:
  name: my-service-name
spec:
  ports:
    - name: my-deploy-svc
      port: 8080
      targetPort: 80
  tpye: NodePort
  selector:
    app: my-deploy
```

예제 7.1과 다르게 kind 항목에 resource의 종류를 Service로 명시했고, metadata에 서비스의
이름을 설정함. 여기서 맨 마지마게 있는 selector 항목의 app: my-deploy 는 어떤 포드를 선택해
이 service의 외부에 노출할 것인지를 선택함. 예제 7.1 의 my-deploy.yaml 파일에서 
deployment의 라벨을 app:my-deploy로 설정했으므로 이 deployment에서 생성된 포드는 위에서
생성한 서비스에 의해 외부에 노출될 대상이 됨. 

즉, 라벨이 외부로 노출할 선택자로 작용하는 것임.

 
 
 type: NodePort는 모든 node에서 해당 서비스에 접근할 수 있게 설정함. 
 이 것은 스웜모드의 ingress 네트워크와 유사하게 어느 노드에 접근해도 같은 서비스를 받을 수 있도록
 설정함.
 targetPort:80은 container 외부에 노출할 port를 의미함.
 
 
 마찬가지로 위 파일도 kubectl create 명령어로 서비스를 생성함.
 
 ```
 # kubectl create -f my-service.yaml
 ```
 
 
 서비스를 생성한 뒤 서비스의 목록을 확인하면 각 노드에서 접근할 수 있는 포트를
 확인할 수 있음. 서비스의 다음 예제에서는 kubernetes의 각 node의 IP 주소와
 30091 포트로 포드에 접근할 수 있음.
 
 ```
 # kubectl get service
 ```
 
 
 ```
 각 node의 IP 주소는 kubectl describe 명령어오 확인할 수 있음.
 describe는 특정 resource의 자세한 정보를 출력함.
 
 # kubectl describe nodes | grep Addr
 ```
 
 외부에서 각 node의 공인 IP와 서비스의 port로 접근하려면 다음과 같이 container의 host 이름을
 확인할 수 있음. 
 
 service는 자동으로 pod 사이에 round robin 방식의 load balancing을 적용하므로 
 브라우저로 접근할 때 마다 다른 host 이름이 출력되는 것을 알 수 있음.
 
 
![그림7.10](https://github.com/ALGOSOPT/DockerConcept/blob/master/picture/7.10.jpg)
 
 
 
 ```
 GCE에서 kubernetes를 사용하고 있다면 service의 port로 접근하기 위해 방화벽
 규칙에서 특정 port를 개방해야함. page의 하단의 googlle cloud shell terminal에서
 다음 명령어를 입력하면 port를 개방할 수 있으며, 개방하려는 port의 번호를
 --allow 옵션에 적절히 입력함
 
 # gcloud compute firewall-rules create my-rule-name --allow tcp:32697
 ```
 
 service로 접근하기 위한 port를 서비스의 yml 파일에 지정하지 않으면
 kubernetes는 server에 30000번부터 32767번까지의 port중 하나를 선택해
 임의로 service에 할당하지만, nodePort 항목을 명시하면 serivce가 외부로
 노출될 port를 직접 지정할 수 있음.
 
 
 다음 예시에서는 service에 접근하기 위한 port를 31000번으로 지정함.
 
 단, 이 port 또한 30000번부터 32767번 port 중 하나를 선택해야함.
 
 ```
 # vi my-service.yaml
 
 ...
 spec:
  ports:
    - name : my-deploy-svc
      port:8080
      targetPort: 80
        nodePort:31000
  type: NodePort
  
 ```
 
 cloud platform에서 kubernetes를 사용하고 있다면 service의 type을 LoadBalancer
 로 설정해 사용할 수 있음.
 
 LoadBalancer의  Type의 서비스는 각 포드에 접근하기 위한 외부 IP를 별도로 할당하고
 서비스에 접근하는 연결을 분배함.
 
 이를 test하기 위해 앞에서 생성한 service를 먼저 삭제함.
 
 ```
 # kubectl delete svc my-service-nameservice
 ```
 
 위에서 작성했던 my-service.yaml 파일을 다음과 같이 수정함. 
 변경된 내용은 type:LoadBalancer 임.
 
 ```
 예제 7.3 수정된 my-service.yaml
 
 # vi my-service.yaml
 ...
 
      port : 8080
    targetPort : 80
  type : LoadBalancer
  selector :
...
 ```
 
 수정된 파일로 service를 생성함.
 
 ```
 # kubectl create -f my-service.yaml
 ```
 
 서비스를 생성한 직후 kubectl로 서비스 목록을 확인했을 떄 EXTERNAL-IP가 
 <pending>이면 IP가 할당 중이라는 의미이며, 시간이 조금 지나면 외부 IP가
  할당되는 것을 확인할 수 있음.
  단, NodePort와는 다르게 무작위로 할당되는 port가 아닌 yaml 파일에서
  port항목에 정의한 port로 접근해야함. 
  
  
  이 예제에서는 EXTERNAL-IP와 8080으로 서비스에 접근함.
  
  ```
  # kubectl get svc my-service-name
  
  # kubectl get svc
  
  ```
  
  EXTERNAL-IP에 할당된 IP와 8080번 port로 접근하면 하나의 외부 IP로
  loadbalancing된 Web App을 사용할 수 있음.
  
  
  ![그림7.11](https://github.com/ALGOSOPT/DockerConcept/blob/master/picture/7.11.jpg)
  
  ---
  
  #### 7.1.3.3 Kubernetes Dashboard
  
  지금까지 kubectl 명령어로 kubernetes의 기능을 사용했음.
  command line으로 kubernetes 의 기능을 사용하는 것이 강력하고
  편리할 수 있지만 kubectl 명령어에 어느정도 익숙해지고 나면 
  
  dashboard를 사용하는 것도 나쁘지 않음. dashboard는 kubernetes의 기능을 웹 UI로 제공할 
  뿐 아니라 clustering monitoring 정보도 함께 보여주기 때문에
  
  kubernetes를 사용하려고 한다면 dashboard도 함께 설치하는 것이 좋음.
  
  
  kubernetes의 dashboard는 container로 제공되므로 쉽게 생성할 수 있음.
  
  직접 kubernetes를 설치했거나 Minikube를 사용하고 있다면 dashboard는 기본적으로
  설치되어 있지 않지만 GCE와 같은 cloud platform에서
  kubernetes cluster를 생성했다면 deployment나 replication controller
  또는 replica 셋으로 dashboard가 미리 생성되어 있을 수 있음.
  
  ```
  # kubectl get rs --namespace kube-system | grep dashboard
  ```
  
  ```
  kubectl get pods 명령어를 실행해도 dashboard pod는 출력되지 않음.
  pod와 service는 namespace를 지정해 용도별로 그룹화 할 수 있는데,
  kubernetes cluster 구동에 관련된 resource는 kube-system
  네임스페이스로 지정해 kubectl get pods 명령어에도 출력되지 않도록 설정됨.
  ```
  
  GCE에서 쿠버네티스를 사용하고 있다면 웹 콘솔 창에서 kubectl cluster-info
  명령어로  dashboard의 주소를 확인할 수 있음.
  웹 브라우저에서 이 주소로 접근해 kubernetes의 dashboard에 접속할 수 있음.
  
  
```
# kubectl cluster-info
```


로그인에 쓰이는 기본 계정은 admind이며 비밀번호는 다음 명령어로 확인할 수 있음
cluster 이름과 zone 이름은 container cluster 목록에서 확인할 수 있으며
아래 예시에서는 us-centrall-a 가 zone의 이름ㅇ.ㅁ

![그림7.12](https://github.com/ALGOSOPT/DockerConcept/blob/master/picture/7.12.jpg)

```
# gcloud container clusters decribe (클러스터 이름) --zone (zone 이름) | grep
```

클라우스 플랫폼이 아닌 on premise 환경에서 kubernetes를 설치했다면 다음 명령어를
입력해 대시보드를 설치함. yaml 파일은 local 뿐만 아니라 외부 URL에서 내려 받아 사용할 수 도
있으므로 다음 명령어로 dashboard를 생성함.

```
# kubectl create -f \
주소 블라블라  p332
```

위 명령어로 생성된 dashboard pod의 service type은 NodePort로 지정되어 있으므로 모든
node에서 접근할 수 있음. 다음 예제에서는 모든 node의 314989번 port로 접근하면 
dashboard에 접속할 수 잇음.

```
# kubectl get service --namespace kube-system
```

![그림7.13](https://github.com/ALGOSOPT/DockerConcept/blob/master/picture/7.13.jpg)
