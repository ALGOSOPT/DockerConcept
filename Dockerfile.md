# Dockerfile


### 이미지를 생성하는 방법
개발한 애플리케이션을 컨테이너화할 때 가장 먼저 생각나는 방법
1. 아무 것도 존재하지 않는 이미지(우분투, CentOS 등)로 컨테이너를 생성
2. 애플리케이션을 위한 환경을 설치하고 소스코드 등을 복사해 잘 동작하는 것을 확인
3. 컨테이너를 이미지로 commit

이 방법을 사용하면 application이 동작하는 환경을 구성하기 위해 일일이 수작업으로 package를 설치하고 소스코드를 Git에서 복제하거나 host에서 복사해야함.
물론 직접 컨테이너에서 application을 구동해보고 image로 commit하기 때문에 이미지의 동작을 보장할 수 있다는 점도 있음.


![Dockerfile로 이미지를 생성하는 방법][1]


#### 도커는
위와 같은 일련의 과정을 손쉽게 기록하고 수행할 수 있는 build 명령어를 제공합니다. 완성된 image를 생성하기 위해 컨테이너에 설치해야 하는 
package, 추가해야 하는 소스코드, 실행해야 하는 명렁어와 shell script 등을 하나의 file에 기록해두면 docker는 이 file을 읽어 컨테이너에서
작업을 수행한 뒤 이미지로 만들어 냄.

이러한 작업을 기록한 file의 이름을 Dockerfile이라 하며, 
빌드 명령어는 Dockerfile을 읽어 이미지를 생성함. 
Dockerfile을 사용하면 직접 컨테이너를 생성하고 image로 commit해야 하는 번거로움을 덜 수 있고,
Git과 같은 개발 도구를 통해 applicaion의 build 및 배포를 자동화할 수 있음.



[1]: /home/sangjo/사진/dockerfile1.jpg
