# Docker Engine


---
---

## 1. Docker image와 Container
---
### 1.1 Docker image

image는 Container를 생성할 때 필요한 요소이며, 가상 머신을 생성할 때 사용하는 iso 파일과 비슷한 개념.
image는 여러개의 계층으로 된 binary file로 존재하고, Container를 생성하고 실행할 때 read only로 사용됨.

image는 docker command로 내려 받을 수 있으므로 별도로 설치할 필요는 없음.


Docker에서 사용하는 image의 이름은 기본적으로 [저장소 이름]/[이미지 이름]:[태그]의 형태


예
```
alice106/ubuntu:14.04       -> 저장소 이름 = alice106, 이미지 이름  = ubuntu , 이미지 버전 = 14.04

ubuntu:latest               -> 이미지 이름 = ubuntu, 이미지 버전 = latest

```

  * Repository(저장소)는 image가 저장된 장소를 의미. 저장소 이름이 명시되지 않은 image는 docker에서
  기본적으로 제공하는 image 저장소인 Docker Hub의 공식(Official) 이미지를 뜻함. 그러나 이미지를 생성
  할 때 저장소 이름을 명시할 필요는 없어서 생략하는 경우도 있음.
  
  * Image 이름은 해당 image가 어떤 역할을 하는지 나타냄. 위 예시는 ubuntu container를 생성하기 위한 
  image라는 것을 알 수 있음. image이름은 생략할 수 없으며 반드시 설정해야함
  
  * Tag는 image의 버전관리, 혹은 Revision 관리에 사용. 일반적으로 14.04와 같이 버전을 명시하지만 Tag
  를 생략하면 Docker Engine은 image의 tag를 lastest로 인식
  
---

### 1.2 Docker Container

앞에서 설명한 Docker image는 ubuntu, CentOS 등 기본적인 리눅스 운영체제부터 아파치 웹 서버, MySQL 등의
각종 Application, Hadoop, Spark, Storm 등의 빅데이터 분석 도구까지 갖가지 종류가 있음.
이러한 image로 Container를 생성하면 해당 image의 목적에 맞는 file이 들어 있는 file system과 격리된 시스템
자원 및 네트워크를 사용할 수 있는 독립된 공간이 생성되고, 이 것이 바로 Docker Container가 됨.

대부분의 Docker Container는 생성될 때 사용된 Docker image의 종류에 따라 알맞은 설정과 파일을 가지고 있기 때문에
Docker image의 목적에 맞도록 사용되는 것이 일반적. 예를 들어 웹 서버 Docker image로부터 여러 개의 Container를 
생성하면 이 Container들은 외부에 Web 서비스를 제공하는 데 사용될 것.


Container는 Image를 Read only로 사용하되 Image에서 변경된 사항만 Container 계층에 저장하므로 Container에서 무엇
을 하든지 원래 Image는 영향을 받지 않음. 또한 생성된 각 Container는  각기 독립된 filesystem을 제공받으며 host와
분리돼 있으므로 특정 Container에서 어떤 Application을 설치하거나 삭제해도 다른 컨테이너와 host는 변화가 없음.



---
---

## 2 Docker Container 다루기

---
### 2.1 Container 생성


도커의 버전 확인 
```
# docker -v
```


도커 생성
```
# docker run -i -t ubuntu:14.04
```
docker run 명령어는 Container를 생성하고 실행하는 역할. -i -t 옵션은 Container와 상호(interactive) 입출력을 가능하게 함.


docker run명령어로 Container 생성 및 실행, 컨테이너 내부로 들어옴. Shell의 사용자와 Host 이름이 변경된 것이 Container의 내부에
들어와 있다는 것. Container에서 기본 사용자는 root이고 host의 이름은 무작위의 16진수 hash value. 

-i 옵션으로 상호 입출력
-t 옵션으로 tty를 활성화해서 bash shell을 사용하도록 Container를 설정함.
docker run 명령어세어 이 두 옵션 중 하나라도 사용하지 않으면 shell을 정상적으로 사용할 수 없음.



Container와 host의 file system은 서로 독립적이므로 ls 명령어로 file system을 확인해보면 아무 것도 설치 되지 않은 상태임을
확인할 수 있음.



Container 내부에서 Container를 정지시키면서 빠져나오는 방법은 exit, Ctrl + D.
Container 내부에서 Container를 정지하지 않고 빠져나오는 방법은 Ctrl + P, Q



pull 명령어.
Docker 공식 image 저장소에서 centos:7 이미지를 내려 받음.
```
#docker pull centos:7
```


images 명령어는 Local Docker Engine에 존재하는 image의 목록을 출력함.
```
# docker images
```



컨테이너를 생성할 때, create 명령어를 사용할 수도 있음.

```
# docker create -i -t --name mycentos centos:7
```
--name 옵션으로 컨테이너의 이름을 설정.

create 명령어는 Container를 생성할 뿐 Container로 들어가지는 않음.


start 명령어.
```
# docker start mycentos
```
Container를 시작.


attath
```
# docker attath mycentos
```
Container의 내부로 들어감.




`docker run = docker pull(이미지가 없을 때) + docker create + docker start + docker attach(-i -t 옵션을 사용 했을 때)`
`docker create = docker pull(이미지가 없을 때) + docker create`




---

### 2.2 Container 목록 확인

```
# docker ps
```
ps 명령어는 정지되지 않은 Container만 출력. 

즉, exit를 통해 빠져나온 Container는 정지 상태이기 때문에 Container 목록에 않지만, 

Ctrl + P, Q를 입력해 빠져나온 Container는 실행 중이기 때문에 Container 목록에 출력됨.


-a 옵션을 추가하면
```
# docker ps -a
``` 
정지된 Container를 포함한 모든 Container가 출력됨.



 * CONTAINER ID : 컨테이너에게 자동으로 할당되는 고유한 ID의 일부분. 모든 ID를 알고 싶으면, # docker inspect 명령어를 사용해 확인하면됨.
 
 * IMAGE : Container를 사용된 image의 이름. 
 
 * COMMAND : Container가 시작될 때 실행될 명령어. 커맨드는 대부분의 image에 미리 내장되어 있기 때문에 별도로 설정할 필요는 없음. 
 
 * CREATED : Container가 생성되고 난 뒤 흐른 시간을 나타냄.
 
 * STATUS : Container의 상태를 나타내며, Container가 실행 중임을 나타내는 'Up', 종료된 상태인 'Exited',일시 중지된 상태인 'Pause'
 등이 있음.
 
 * PORTS : Container가 개방한 port와 host에 연결한 post를 나열함. 앞에서 Container를 생성할 때는 외부에 노출하도록 설정하지 않았으므로 PORTS 항목에는 아무것도 출력되지 않았음.
 
 * NAMES : Container의 고유한 이름. Container를 생성할 때 --name 옵션으로 이름을 설정하지 않으면 Docker Engine이 임의의 형용사와 명사를 무작위로 조합해 이름을 설정. Container의 이름은 ID와 마찬가지로 중복될 수는 없음. Docker rename 명령어를 사용하면 Container의 이름을 변경하능
 
---

### 2.3 Container 삭제

더 이상 사용하지 않는 Container를 삭제할 때는 docker rm 명령어를 사용.

한 번 삭제한 Container는 복구할 수 없음.


```
# docker rm 이름
```

docker ps -a 명령어로 삭제가 되었는지 확인.



** 실행 중인 ** Container는 삭제 할 수 없음. docker stop 을 먼저 한 후 docker rm으로 삭제.


모든 Container를 삭제
```
# docker container prune
```


```
# docker stop $(docker ps -a -q)
# docker rm $(docker ps -a -q)
```
docker ps -a 는 모든, -q는 Container의 ID만 출력하게 하고 이를 변수로 활용해 사용하는 예제.





---

### 2.4 Container를 외부에 노출

Container는 가상 머신과 마찬가지로 가상 IP 주소를 할당받음. 

기본적으로 Docker는 Container에 172.17.0.X의 IP를 순차적으로 할당.

```
# docker run -i -t --name network_test ubuntu:14.04
/# ifconfig
```
docker의 NAT IP인 172.17.0.2를 할당받은 eth0 인터페이스와 local host인 lo interface가 있음.

아무런 설정을 하지 않았다면 이 Container는 외부에서 접근할 수 없으며 Docker가 설치된 host에서만 접근할 수 있음.

외부에 Container의 App을 노출하기 위해서는 eth0의 IP와 port를 host의 ip와 port에 binding 해야함.




Container에서 host로 빠져나온 뒤 다음 명령어를 입력해 Container를 생성.
```
#docker run -i -t --name mywebserver -p 80:80 ubuntu:14.04
```
-p 옵셔은 container의 port를 host의 port와 binding해서 연결할 수 있게 설정
-p 옵션의 입력 형식은 다음과 같음.

`[host port]:[container port]`

host의 특정 IP를 사용하려면 `192.168.0.100:7777:80'과 같이 binding할 IP와 port를 명시.
또한 여러개의 포트를 외부에 개방하려면 -p 옵션을 여러번 써서 설정.

```
# docker run -i -t -p 3306:3306 -p 192.168.0.100:7777:80 ubuntu:14.04
```

---

### 2.5 Container Application 구축

대부분의 서비스는 단일 프로그램으로 동작하지 않음. 여러 agent와 database 등과 연결되어 완전한 서비스로써 
동작하는 것이 일반적입니다 이런 서비스를 Containerize할 때 여러개의 Application을 한 Container 안에
설치 할 수도 있음. 그러나 Container에 Application을 하나만 동작시키면 Container간의 독립성을 보장함과
동시에 Application의 버전관리, 소스코드 모듈화 등이 더욱 쉬워짐.




database와 web server를 container하나에 설치할 수도 있음. 그러나 database container와 web server container를
구분하는 편이 docker image를 관리하고 component의 독립성을 유지하기가 쉬움.

이 같은 구조는 여러 docker 커뮤니티 뿐 아니라 docker 공식 홈페이지에서도 권장하는 구조.

한 container에서 process 하나만 실행하는 것이 docker의 철학이기 때문.




이번에는 database와 word press web server container를 연동해 word press기반 블로그 서비스를 만들어 보자.

```
# docker run -d --name wordpresdb -e MYSQL_ROOT_PASSWORD=password -e MYSQL_DATABASE=wordpress mysql:5.7
```

```
# docker run -d -e WORDPRESS_DB_PASSWORD=password --name wordpress --link wordpressdb:mysql -p 80 wordpress
```

첫 번째 명령어는 mysql 이미지를 사용해 database container를, 두 버째 명령어는 미리 준비된 wordpress 이밎지를 이용해 
wordpress web server container를 생성함. wordpress web server container의 -p 옵션에서 80을 입력했으므로
host의 port 중 하나와 container의 80번 port가 연결됨.

```
# docker ps
```
docker ps 명령어로 host의 어느 port와 연결됐는지 확인.


