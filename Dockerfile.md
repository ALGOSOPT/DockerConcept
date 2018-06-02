# 1. Dockerfile


### 이미지를 생성하는 방법
개발한 애플리케이션을 컨테이너화할 때 가장 먼저 생각나는 방법
1. 아무 것도 존재하지 않는 이미지(우분투, CentOS 등)로 컨테이너를 생성
2. 애플리케이션을 위한 환경을 설치하고 소스코드 등을 복사해 잘 동작하는 것을 확인
3. 컨테이너를 이미지로 commit

이 방법을 사용하면 application이 동작하는 환경을 구성하기 위해 일일이 수작업으로 package를 설치하고 소스코드를 Git에서 복제하거나 host에서 복사해야함.
물론 직접 컨테이너에서 application을 구동해보고 image로 commit하기 때문에 이미지의 동작을 보장할 수 있다는 점도 있음.


![Dockerfile로 이미지를 생성하는 방법](./home/sangjo/사진/dockerfile1.jpg)


#### 도커는
위와 같은 일련의 과정을 손쉽게 기록하고 수행할 수 있는 build 명령어를 제공합니다. 완성된 image를 생성하기 위해 컨테이너에 설치해야 하는 
package, 추가해야 하는 소스코드, 실행해야 하는 명렁어와 shell script 등을 하나의 file에 기록해두면 docker는 이 file을 읽어 컨테이너에서
작업을 수행한 뒤 이미지로 만들어 냄.

이러한 작업을 기록한 file의 이름을 Dockerfile이라 하며, 
빌드 명령어는 Dockerfile을 읽어 이미지를 생성함. 
Dockerfile을 사용하면 직접 컨테이너를 생성하고 image로 commit해야 하는 번거로움을 덜 수 있고,
Git과 같은 개발 도구를 통해 applicaion의 build 및 배포를 자동화할 수 있음.




Dockerfile은 app을 개발하는 용도 외에도 여러 목적으로 사용될 수 있다.
생성한 image를 Docker Hub 등을 통해 배포할 때 이미지 자체를 배포하는 대신 image를 생성하는 방법을 기록해 놓는 Dockerfile을
배포할 수 있음. 
배포 되는 image를 신뢰할 수 없거나 직접 이미지를 생성해서 사용하고 싶다면 Docker Hub에 올려져 있는 Dockerfile로 빌드하는 것도
하나의 방법. 실제로 Docker Hub에 올려져 있는 대부분의 Image는 __Dockerfile__을 함께 제공하고 있음.


### Application을 컨테이너화하기 위한 
장기적인 시점에서 본다면 Dockerfile을 작성하는 것이 매우 유리. Application에 필요한 package 설치 등을 명확히 할 수 있고
이미지 생성을 자동화 할 수 있으며, 쉽게 배포할 수 있기 때문.


---
# 2. Dockerfile 작성

앞에서 설명한 것처럼, Dockerfile에는 컨테이너에서 수행해야 할 작업을 명시함. 이 작업을 Dockerfile에 정의하기 위해서는
Dockerfile에서 쓰이는 명령어들을 알아둘 필요가 있음.

Dockerfile은 `도커를 위한 특수한 파일` 인 만큼 기존의 script 언어와 비겨했을 때 완전히 새로운 방식으로 쓰이지만,
컨테이너에서 사용되는 기초적인 명령어를 알기 쉽게 변환한 것이므로 어렵지 않게 익힐 수 있음.


Dockerfile을 사용하기 위한 간단한 시나리오로 웹 서버 이미지를 생성하는 예.

`#echo test >> test.html`
