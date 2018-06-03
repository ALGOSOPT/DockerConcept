# 1. Dockerfile


### 이미지를 생성하는 방법
개발한 애플리케이션을 컨테이너화할 때 가장 먼저 생각나는 방법
1. 아무 것도 존재하지 않는 이미지(우분투, CentOS 등)로 컨테이너를 생성
2. 애플리케이션을 위한 환경을 설치하고 소스코드 등을 복사해 잘 동작하는 것을 확인
3. 컨테이너를 이미지로 commit

이 방법을 사용하면 application이 동작하는 환경을 구성하기 위해 일일이 수작업으로 package를 설치하고 소스코드를 Git에서 복제하거나 host에서 복사해야함.
물론 직접 컨테이너에서 application을 구동해보고 image로 commit하기 때문에 이미지의 동작을 보장할 수 있다는 점도 있음.


![Dockerfile로 이미지를 생성하는 방법](https://github.com/ALGOSOPT/DockerConcept/picture/dockerfile1.jpg)


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

`docker 엔진은 Dockerfile을 읽어 들일 때 기본적으로 현재 directory에 들어 있는 Dockerfile이라는 이름을 가진 file을 선택함.
따라서 아래 예제에서는 Dockerfile이라는 이름으로 file을 저장. Dockerfile은 Empty 디렉터리에 저장하는 것이 좋은데,
이는 이미지를 빌드할 때 사용되는 Context 때문.`


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
  * image를 만들기 위해 container 내부에서 명령어를 실행함. 예제에서는 apt-get update와 apt-get intall apache2 명령어를 
  실행하기 때문에 아파치 웹 서버가 설치된 이미지가 생성. 
  
 *ADD
   * 파일을 이미지에 추가 추가하는 file은 Dockerfile이 위치한 디렉터리인 Context에서 가져옴. 
  Context에 대한 설명은 나중에 하고 지금은 Dockerfile이 위치한 Directory에서 file을 가져온다고 이해하면된다. 
  예제에서는 Dockerfile이 위치한 directory에서 test.html file을 imagedml /var/www/html directory에 추가함.
  
 *WORKDIR
   * 명령어를 실행할 디렉터리를 나타냄. bash shell에서 cd 명령어를 입력하는 것과 똑같은 기능을 함.

 *EXPOSE
   * Dockerfile의 빌드로 생성된 image에서 노출할 port를 설정. 그러나 EXPOSE를 설정한 image로 container를 생성했다고 해서 반드시 
   이 port가 호스트의 port와 binding되는 것은 아니며, 단지 container의 80번 port를 사용할 것임을 나타내는 것. EXPOSE는 컨테이너를 
   생성하는 run 명령어에서 모든 노출된 컨테이너의 포트를 호스트에 publish하는 -p flag와 함께 사용됨. 
  
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

### image 생성
Dockerfile을 빌드.

```
# docker build -t mybuild:0.0 ./
```

-t 옵션은 생성될 iamge의 이름을 설정. 위 명령을 실행하면 mybuild:0.0 이라는 이름의 이미지가 생성됨. -t 옵션을 사용하지 않으면 16진수의 
image가 저장됨.


build 명령어의 끝에는 Dockerfile이 저장된 경로를 입력. 일반적으로 로컹에 저장된 Dockerfile을 사용하지만 외부 URL로부터 Dockerfile의 
내용을 가져와 빌드 할 수도 있음 


다음 명령어로 생성된 이미지로 컨테이너를 실행.
```
# docker run -d -P --name myserver mybuild:0.0
```

-P 옵션은 이미지에 설정된 EXPOSE의 모든 port를 host에 연결하도록 설정.
즉, image를 생성하기 위한 Dockerfile을 작성하는 개발자로서는 EXPOSE를 이용해 이미지가 실제로 사용될 때
어떤 포트가 사용돼야 하는지 명시할 수 있으며, 이미지를 사용하는 입장에서는 컨테이너의 application이 컨테이너 내부에서 어떤 포트를 사용하는지
알 수 있게 됨.

```
#docker ps
#docker port
```


---
# 기타 Dockerfile 명령어

전체 Dockerfile 명령어 목록을 확인하고 싶다면 도커 공식 사이트의 Dockerfile reference를 참고

 *ENV
  * Dockerfile에서 사용될 환경변수를 지정. 설정한 환경변수는 ${ENV_NAME} 또는 $ENV_NAME의 형태로 사용할 수 있음.
    이 환경 변수는 Dockerfile 뿐 아니라 이미지에도 저장되므로 빌드된 이미지로 컨테이너를 생성하면 이 환경변수를 사용할 수 있음.
    다음 Dockerfile에서는 test라는 환경변수에 /home이라는 값을 설정함.
  ```
  #vi Dockerfile
  
  FROM ubuntu:14.04
  ENV test /home
  WORKDIR $test
  RUN touch $test/mytouchfile
  ```
  
  이미지를 빌드하고 컨테이러르 생성해 환경변수를 확인하면 /home 값이 적용된 것을 확인할 수 있음. 
  run 명령어에서 -e 옵션을 사용해 같은 이름의 환경변수를 사용하면 기존의 값은 덮어 쓰여짐.
  
  ```
  #docker build -t myenv:0.0 ./
  
  #docker run -i -t --name env_test myenv:0.0 /bin/bash
  
  /home# echo $test
  
  /home
  ```
  
  ```
  #docker run -i -t --name env_test_override -e test=myvalue myenv:0.0 /bin/bash
  
  /home# echo $test
  
  myvalue
  ```
  
  
  Dockerfile에서 환경변수의 값을 사용할 때 bash shell에서 사용하는 것처럼 값이 설정되지 않은 경우와 설정된 경우를 구분해 사용할 수 있음.
  ${env_name:-value}는 env_name이라는 환경변수의 값이 설정되지 않았으면 이 환경변수의 값을 value로 사용함.
  반대로, ${env_name:+value}는 env_name의 값이 설정되어 있으면 value를 값으로 사용하고, 값이 설정되지 않았다면 빈 문자열을 사용함.
  
  ```
  #vi Dockerfile
  FROM ubuntu:14.04
  ENV my_env my_value
  RUN echo ${my_env:-value} / ${my_env:+value} / ${my_env2:-value} / ${my_env2:+value}
  ```
  
  위 내용으로 Dockerfile을 작성한 뒤 image를 빌드하면 RUN에 입력된 echo 명령어를 실행한 결과를 확인 할 수 있음.
  
  
  
  
  * VOLUME 
    * 빌드된 image로 컨테이너를 생성했을 때 host와 공유할 container 내부의 directory를 설정함. VOLUME ["/home/dir", "home/dir2"] 
    처럼 JSON 배열의 형식으로 여러 개를 사용하였거나 VOLUME /home/dir /home/dir2 로도 사용할 수 있음.
    다음 예는 컨테이너 내부의 /home/volume directory를 host와 공유하도록 설정
    
    ```
    #vi Dockerfile
    
    FROM ubuntu:14.04
    RUN mkdir /home/volume
    RUN echo test >> /home/volume/testfile
    VOLUME /home/volume
    ```
    이미지를 빌드 한 뒤 컨테이너를 생성하고 볼륨의 목록을 확인해보면 볼륨이 생성된 것을 확인할 수 있.
    
    ```
    # docker build -t myvolume:0.0. .
    
    # docker run -i -t -d --name volume_test myvolume:0.0
    
    # docker volume ls
    ```
    
  * ARG
    * build 명령어를 실행할 때 추가로 입력을 받아 Dockerfile 내에서 사용될 변수의 값을 설정.
    다음 Dockerfile은 build 명령어에서 my_arg와 my_arg_2라는 이름의 변수를 추가로 입력받을 것이라고 ARG를 통해
    명시합니다. ARG의 값은 기본적으로 build 명령어에서 입력 받아야 하지만 다음의 my_arg_2와 같이 기본값을 지정할 수도 있음.
    
    ```
    #vi Dockerfile
    FROM ubuntu:14.04
    ARG my_arg
    ARG my_arg_2=value2
    RUN touch ${my_arg}/mytouch
    ```
    
    위 내용을 Dockerfile로 저장한 뒤 이미지를 빌드함. build 명령어를 실행 할 때 --build-arg 옵션을 사용해서 Dockerfile ARG에 
    값을 지정할 주 있음 
    
    ```
    # docker build --build-arg my_arg=/home -t myarg:0.0 ./
    ```
    ARG와 ENV의 값을 사용하는 방법은 ${}로 같으므로 Dockerfile에서 ARG로 설정한 변수를 ENV에서 같은 이름으로 다시 정의하면 
    --build-arg 옵션에서 설정하는 값은 ENV에 의해 덮어쓰여집니다.
    
    위의 Dockerfile 예제에서는 $(my_arg)의 디렉터리에 mytouch 라는 파일을 생성했기 때문에 빌드된 이미지로 컨테이너를 생성해 
    확인하면 mytouch라는 이름의 파일을 확인할 수 있음.
    
    ```
    #docker run -i -t --name arg_test myarg:0.0
    
    #ls /home/mytouch
    ```
    
  *USER
    * USER로 컨테이너 내에서 사용될 사용자 계정의 이름이나 UID를 설정하면 그 아래의 명령어는 해당 사용자의 권한으로 실행됨.
    일반적으로 RUN으로 사용자의 그룹과 계정을 생성한 뒤 사용함. 루트 권한이 필요하지 않다면 USER를 사용하는 것을 궈장
    ```
    RUN groundadd -r author && useradd -r -g author alicek106
    USER alicek106
    ```
    
  *Onbuild
    * 빌드된 이미지를 기반으로 하는 다른 이미지가 Dockerfile로 생성될 때 실행할 명령어를 추가.
   ```
   #vi Dockerfile
   FROM ubuntu:14.04
   RUN echo "this is onbuild test!"
   ONBUILD RUN echo "onbuild!" >> /onbuild_file
   ```
   
   ONBUILD에 RUN echo "onbuild!" >> /onbuild_file을 입력해서 onbuild!라는 명령어가 최상위 디렉터리의 onbuild_file에 저장되도록 함.
   이 명령어는 이 Dockerfile을 빌드할 때 실행되지 않으며, 별도의 정보로 이미지에 저장될 뿐.
   
   
   ```
   #docker run -i -t --rm onbuild_test:0.0 ls /
   ```
   
   이번에는 위에서 생성한 이미지를 기반으로 하는 새로운 이미지를 Dockerfile로 생성해보자. 다음과 같이 Dockerfile을 작성
   ```
   #vi Dockerfile2
   FROM onbuild_test:0.0
   RUN echo "this is child image"
   ```
   
   그리고 빌드.
   
    ```
    #docker build -f ./Dockerfile2 ./ -t onbuild_test:0.1
    ```
    
    
    ```
    #docker run -i -t --rm onbuild_test:0.1 ls /
    ```
    
    이처럼 ONBUILD는 ONBUILD, FROM, MAINTAINER를 제외한 RUN, ADD 등, 이미지가 빌드될 때 수행되야 하는 각종 Dockerfile의 
    명령어를 나중에 빌드될 이미지를 위해 미리 저장 해놓을 수 있다. 단, 이미지의 속성을 설정하는 다른 Dockerfile 명령어와는 달리 
    ONBUILD는 부모 이미지의 자식 이미지에만 적용되며, 자식 이미지는 ONBUILD 속성을 상속 받지 않음.
    
    
  *STOPSIGNAL
    *컨테이너가 정지 될  시스템 콜의 종류를 지정. 아무것도 설정하지 않으면 SIGTERM로 설정되지만 Dockerfile에 STOPSIGNAL을 정의해 
    컨테이너가 종료되는 데 사용될 신호를 선택할 수 있음.
    
    ```
    FROM ubuntu:14.04
    STOPSIGNAL SIGKILL
    ```
    
  *HEALTHCHECK : HEALTHCHECK는 이미지로부터 생성된 컨테이너에서 동작하는 application의 상태를 체크하도록 설정. 
  컨테이너 내부에서 동작 중인 application의 process가 종료되지는 않았으나 application이 동작하고 있지 않은 상태를 방지하기 
  위해 사용될 수 있음.
  
  다음 예제는 1분 마다 curl -f ...를 실행해 nginx 앱의 상태를 체크하여, 3초 이상이 소요되면 이를 한 번의 실패로 간주.
  3번 이상의 타임 아웃이 발생하면 해당 container는 unhealthy 상태가 됨. 단, HEALTHCHECK에서 사용되는 명령어가 curl이므로 
  컨테이너에 curl을 먼저 설치
  ```
  #vi Dockerfile
  FROM nginx
  RUN apt-get update -y && apt-get install curl -y
  HEALTHCHECK --interval=1m --timeout=3s --retries=3 CMD curl -f http://localhost || exit 1
  ```
  
  HEALTHCHECK에서 --interval은 컨테이너의 상태를 체크하는 주기. 마지막에 지정한 CMD curl .. 부분이 상태를 체크하는 명령어가 되고, 
  --interval에 지정된 주기마다 이를 실행함. 상태를 체크하는 명령어가 --timeout에 설정한 시간을 초과하면 상테 체크에 실패한 것으로 
  간주하고 --retries의 횟수만큼 명령어를 반복함. --retries에 설정된 횟수만큼 상태 체크에 실패하면 해당 컨테이는 unhealthy 상태로 설정됨.
  
  이미지를 빌드한 후 해당 이미지로 컨테이너를 생성하면 docker ps 의 출력 중 해당 컨테이너의 STATUS에 정보가 추가 된 것을 확인 할 수 있음.
  
  
  ```
  #docker build ./ -t nginx:healthcheck
  #docker run -d -P nginx:healthcheck
  #docker ps
  ```
  
  
  상태 체크에 대한 로그는 컨테이너의 정보에 저장되므로 애플리케이션에 장애가 있을 때 해당 로그를 확인 할 수 있음.
  
  docker inspect의 출력 중 State - Health -Log 항목에서 확인 할 수 있음.
  
  
  *SHELL
    * Dockerfile에서 기본적으로 사용하는 shell은 리눅스에서 "/bin/sh -c", 윈도우에서 "cmd /S /C"
    하지만 사용하려는 shell을 따로 지정하고 싶을 수도 있음. 
    다음은 node를 기본 shell로 사용하도록 설정한 예
    ```
    FROM node
    RUN echo hello, node!
    SHELL ["/usr/local/bin/node"]
    RUN -v
    ```
    
    ```
    #docker build ./ -t nodetest
    ```
    
    
    
  *COPY
    *COPY는 로컬 디렉터리에서 읽어 들인 컨텍스트로부터 이미지에 파일을 복사하는 역할을 함.
    COPY를 사용하는 형식은 ADDdhk rkxek.
    
    
    ```
    COPY test.html /home/
    COPY ["test.html", "/home/"]
    ```
    그렇다면 ADD와 COPY의 차이점은 없는 것처럼 보임. 사실 ADD와 COPY의 기능은 그 자체로만 봤을 때는 같음.
    그러나 COPY는 로컬의 파일만 이미지에 추가할 수 있지만 ADD는 외부 URL 및 tar 파일에서도 파일을 추가할 수 있다는 점에서 다름.
    
    즉, COPY의 기능이 ADD에 포함되는 셈.
    
    하지만 ADD를 사용하는 것은 그다지 권장하지 않음 그 이유는  ADD로 URL이나 tar파일을 추가할 경우 이미지에 정확히 어떤 파일이 
    추가 될지 알 수 없기 때문. 그에 비해  COPY는 로컬 context로부터 file을 직접 추가하기 때문에 빌드 시점에서도 어떤 파일이 
    추가 될지 명확.
    
    


  * ENTRYPOINT
    * CMD는 컨테이너가 시작될 때 실행할 명령어를 설정. 이는 docker run 명령어에서 맨 뒤에 입력했던 cmd와 같은 역할을 함. 
    CMD와 비슷한 ENTRYPOINT라는 것이 있음.
    
    * ENTRYPOIN와 CMD의 차이점
    entrypoint는 커맨드와 동일하게 컨테이너가 시작될 때 수행할 명령을 지정한다는 점에서 같음 
    그러나 entrypoint는 커멘드를 인자로 받아 사용할 수 있는 스크립트의 역할을 할 수 있다는 점에서 다름.
    
    
    다음예는 ubuntu:14.04 이미지로 containaer를 생성하고 container가 시작될 때 실행될 명령어를 /bin/bash로 설정.
   ```
   #docker run -i -t --name no_entrypoint ubuntu:14.04 /bin/bash
   ```
   
   지금까지 컨테이너를 생성했던 것처럼 컨테이너 내부로 들어왔으며,  cmd로 bash shell을 실행하도록 설정했으므로 shell을 통해 표준입출력을
   할 수 있음. 그렇다면 entrypoint로 특정 명령ㅇ어를 넣으면
   
   ```
   #docker run -i -t --entrypoint="echo" --name yes_entrypoint ubuntu:14.04 /bin/bash
   ```
   
   /bin/bash 라는 내용이 터미널에 출력됨. 즉, container에 entrypoint가 설정되면 run 명령어의 매 마지막에 
   입력된 cmd를 인자로 삼아 명령어를 출력.
   그러므로 위 예에서 echo /bin/bash 가 컨테이너에서 실행되어 /bin/bash가 출력된 것.
   
   entrypoint가 설정되지 않았다면 cmd에 설정되 명령어를 그대로 실행하지만 entrypoint가 설정됐다면 cmd는 단지 
   entrypoint에 대한 인자의 기능을 함.
   
   * entrypoint를 이용한 script 실행
   앞에서 살펴본 것 처럼 entrypoint에 하나의 명령어만 입력할 수도 있지만 일반적으로는 script file을 entrypoint의 인자로 사용해 
   컨테이가 시작될 때 마다 해당 script file을 실행하도록 설정. script file을 entrypoint에 설정하려면 script file의 이름을 
   entrypoint의 인자로 입력
   
   ```
   #docker run -i -t --name entrypoint_sh --entrypoint="/test.sh" ubuntu:14.04 /bin/bash
   ```
   
   단 실행할 script 파일은 컨테이너 내부에 존재해야함. 
   
   ```
   #vi Dockerfile
   FROM ubuntu:14.04
   RUN apt-get update
   RUN apt-get install apache2 y
   ADD entrypoin.sh /entrypoint.sh
   RUN chmod +x /entrypoint.sh
   ENTRYPOINT ["/bin/bash", "/entrypoint.sh"]
   ```
   
   entrypoint.sh파일의 내용
   ```
   #vi entrypoint.sh
   echo $1 $2
   apachectl -DFOREGROUND
   ```
   
   ```
   #docker build -t entrypoint_image:0.0 ./
   
   #docker run -d --name entrypoint_apache_server entrypoint_image:0.0 first second
   
   #docker logs entrypoint_apache_server
   ```
   
   
   
---
# 5. Dockerfile로 빌드 할 때 주의할 점

Dockerfile을 사용하는 데도 좋은 습관(practice)이라는 것이 있음. 예를 들어, 하나의 명령어를 \로 나눠서 가독성을 높일 수 있거나
.dockerignore 파이릉ㄹ 작성해 불필요한 file을 빌드 컨텍스트에 포함하지 않는 것이 있음.
또는 build cache를 이용해 기존에 사용했던 이미지 layer 를 재사용하느 방법도 Dockerfile을 활용하는 방법 중 하나.

```
#vi Dockerfile
...
RUN apt-get install package-1 \
package-2 \
package-3
```
 
 
다음은 Dockerfile을 사용할 때 이미지를 비효율적으로 빌드하는 예.
fallocate라는 명령어는 100MB 크기의 파일을 가상으로 만들어 container에 할당하고, 
이를 image layer로 build함. 그리고 이 file을 rm 명령어로 삭제.
즉, 빌드가 완료되어 최종 생성된 이미지에는 100MB 크기의 파일인 /dummy가 존재하지 않음

잘못된 Dockerfile의 활용
```
#vi Dockerfile
FROM ubuntu:14.04
RUN mkdir /test
RUN fallocate -l 100m /test/dummy
RUN rm /test/dummy
```

위의 Dockerfile로 빌드한 이미지의 크기는 어떻게 될까?
ubuntu:14.04 이미지에 100MB를 더한 약 290MB 정도를 차지하게 됨

```
#docker build -t falloc_100mb:0.0 .

#docker images
```


그러나 위에서 빌드한 image로 컨테이너를 생성하면 100MB크기의 /test/dummy라는 file은 존재하지 않음
Dockerfile에서 RUN rm /test/dummy 명령어로 이 파일을 삭제한 상태로 이미지를 빌드 했기 때문
이는 컨테이너를 이미지로 생성할 때 컨테이너에서 변경된 사항만 새로운 이미지 레이어로 생성하는
방식의 단점 중 하나. RUN rm /test/dummy 명령어를 수행해 100MB 크기의 파일을 삭제하더라도 이는 파일을
삭제했다 라는 변경사항으로서의 레이어로 새롭게 저장될 뿐, 실제 100MB 크기의 파일은 이전 레이어에 남아있기 때문
즉, 실제로 컨테이너에 사용하지 못하는 파일이 이미지 레이어로 존재하기 때문에 저장 공간은 차지하지만 실제로는
의미가 없는 저장 공간일 수도 있음.

이를 방지하는 방법은 매우 간단. Dockerfile을 작성할 때 &&로 각 RUN명령을 하나로 묶는 것.

즉, 하나의 RUN으로 여러 개의 명령어를 실행하도록 작성하면됨.

```
# vi Dockerfile
FROM ubuntu:14.04
RUN mkdir /test && \
fallocate -l 100m /test/dummy && \
rm /test/dummy
```


RUN이 하나의 이미지 레이어가 된다는 것을 생각해보면 매우 간단한 해결책.
이 방법은 이미지 레이어 수를 줄이는 데도 활용할 수 있음. 여러 개의 RUN명령어가 하나로 묶일 수 있다면
이미지의 layer의 개수 또한 하나로 줄어들기 때문.



다른 사람이 빌드한 image에 불필요한 image layer가 들어있다면 해당 image로 컨테이너를 생성하고 docker export, import 명령어를
사용해 컨테이너를 이미지로 만듦으로써 이미지의 크기를 줄일 수 있음.export file을 import 해서 다시 docker에
저장하면 layer가 한개로 줄어 듦.

그러나 이전 이미지에 저장돼 있던 각종 이미지 설정은 잃어버리게 되므로 주의해야함. 

```
#docker run -i -t -d --name temp falloc_100mb:0.0

#docker export temp | docker import - falloc_100mb:0.1

#docker run -i -t -d --name temp2 falloc_100mb:0.1
```


---

## docker layer
도커의 image layer 구조는 위의 경우에 단점으로 작용할 수도 있지만 잘만 활용하면 장점으로 작용할 수 있음.
예를 들어 A, B 애플리케이션이 500MB 크기의 같은 라이브러리를 사용해야 한다면 각 application의 image에 library를
각기 설치하기 보다는 500MB 크기의 라이브러리 이미지를 미리 만들어 놓은 다음 이 이미지를 이용해 A,B 애플리케이션의 이미지는
이 library오 인해 각각 500MB를 차지해 1GB의 저장공간을 사용하는 것이 아닌, 500MB의 이미지 레이어를 공유하게 됨.

또한 이미지를 배포할 때에도 애플리케이션에 해당하는 변경된 부분만 내려받으면 되므로 이미지를 빠르게 전송할 수 있음.

