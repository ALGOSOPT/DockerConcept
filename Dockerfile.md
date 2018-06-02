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


아래의 Dockerfile은 아파치 웹 서버를 설치하고 local에 있는 test.html file을 web server로 접근 할 수 있는
컨테이너의 directory인 /var/www/html 에 복사함.

```
#mkdir dockerfile && cd dockerfile
#vi Dockerfile


FROM ubuntu:14.04
MAINTAINER alicek106
LABEL "purpose"="practice"
RUN apt-get update
RUN apt-get install apache2 -y
ADD test.html /var/www/html
WORKDIR /var/www/html
RUN ["/bin/bash", "-c", "echo hello >> test2.html"]
EXPOSE 80
CMD apachectl -DFOREGROUND
```

`docker 엔진은 Dockerfile을 읽어 들일 때 기본적으로 현재 directory에 들어 있는 Dockerfile이라는 이름을 가진 file을 선택함. 따라서 아래 예제에서는 Dockerfile이라는 이름으로 file을 저장. Dockerfile은 Empty 디렉터리에 저장하는 것이 좋은데, 이는 이미지를 빌드할 때 사용되는 Context 때문.`


#### Dockerfile은 한 줄이 하나의 명렁어가 되고
명령어(Instruction)를 명시한 뒤 옵션을 추가하는 방식. 
명령어를 소문자로 표기해도 상관없지만 일반적으로 대문자로 표기.

Dockerfile의 명령어는 위에서 아래로 한줄 씩 차례대로 실행됨. 

*FROM
  *생성할 image의 base가 될 image를 뜻함. FROM명령어는 Dockerfile을 작성할 때 반드시 한 번 이상 입력해야하며,
   이미지 이름의 format은 docker run 명령어에서 image 이름을 사용했을 때와 같음. 사용하려는 image가 docker에 없다면
   자동으로 pull함.

*MAINTAINER
  *image를 생성한 개발자의 정보를 나타냄. 일반적으로 Dockerfile을 작성한 사람과 연락할 수 있는 email등을 입력
   단 Docker 1.13.0 버전 이후로 사용하지 않음.

*Label
  * image에 meta data를 추가. meta data는 "key:value"의 형태로 저장되며, 여러 개의 meta data가 저장될 수 있음.
  추가된 meta data는 docker inspect 명렁어로 image의 정보를 구해서 확인 할 수 있음.
  
*RUN
  * image를 만들기 위해 container 내부에서 명령어를 실행함. 예제에서는 apt-get update와 apt-get intall apache2 명령어를 실행하기 때문에
  아파치 웹 서버가 설치된 이미지가 생성. 
  
*ADD
  * 파일을 이미지에 추가 추가하는 file은 Dockerfile이 위치한 디렉터리인 Context에서 가져옴. 
  Context에 대한 설명은 나중에 하고 지금은 Dockerfile이 위치한 Directory에서 file을 가져온다고 이해하면된다. 
  예제에서는 Dockerfile이 위치한 directory에서 test.html file을 imagedml /var/www/html directory에 추가함.
  
*WORKDIR
  * 명령어를 실행할 디렉터리를 나타냄. bash shell에서 cd 명령어를 입력하는 것과 똑같은 기능을 함.

*EXPOSE
  * Dockerfile의 빌드로 생성된 image에서 노출할 port를 설정. 그러나 EXPOSE를 설정한 image로 container를 생성했다고 해서 반드시 이 port가 호스트의 port와 binding되는 것은 아니며, 단지 container의 80번 port를 사용할 것임을 나타내는 것. EXPOSE는 컨테니러를 생성하는 run 명령어에서 모든 노출된 컨테이너의 포트를 호스트에 publish하는 -p flag와 함께 사용됨. 
  
*CMD
  * CMD는 컨테이너가 시작될 때마다 실행할 CMD를 설정하며, Dockerfile에서 한 번만 사용할 수 있음.
 
 
 
다시 정리 Dockerfile을 정리
```
FROM ubuntu:14.04
MAINTAINER alice106
LABEL "purpose"="practice"
```
FROM에서 Docker에서 사용할 base image를 ubuntu:14.04로 설정
MAINTAINER의 이름을 alice106으로 설정
Dockerfile에서 생성될 이미지의 label을 purpose=practice로 설정.


```
RUN apt-get update
RUN apt-get install apache2 -y
ADD test.html /var/www/html
```
RUN으로 apt-get update, apt-get install apache2 -y 명렁어를 실행.
ADD로 dockerfile이 위치한 directory에서 test.html 파일을 image의 /var/www/html 디렉토리에 추가


```
WORKDIR /var/www/html
RUN ["/bin/bash", "-c", "echo hello >> test2.html"]
```
WORKDIR으로 작업 Directory를 /var/www/html로 바꾼 뒤
RUN ["/bin/bash ...로 test2.html 파일이 생성됨
작업 디렉터리가 /var/www/html 로 변경되었으므로 해당 디렉토리에 test2.html 파일이 생성됨.

```
EXPOSE 80
CMD apachectl -DFOREGROUND
```
EXPOSE를 토해 컨테이너가 사용해야할 port를 80번으로 설정하고 CMD로 컨테이너의 명령어를 apachectl -DFOREGROUND로 설정해
이미지 빌드를 마침. 





---
# 3 Dockerfile 빌드

