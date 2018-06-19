# Docker Engine

## 목차 ##

- [Docker image와 Container](#1)
  - [Docker image](#1-1)
  - [Docker Container](#1-2)
- [Docker Container 다루기](#2)
  - [Container 생성](#2-1)
  - [Container 목록 확인](#2-2)
  - [Container 삭제](#2-3)
  - [Container를 외부에 노출](#2-4)
  - [Container Application 구축](#2-5)
  - [Docker Volume](#2-6)
      - [Host Volume 공유](#2-6-1)
      - [Volume Container](#2-6-2)
      - [Docker Volume](#2-6-3)
  - [Docker Network](#2-7)
---
---

<a name="1"></a>
## 1. Docker image와 Container
---
<a name="1-1"></a>
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
<a name="1-2"></a>
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
<a name="2"></a>
## 2 Docker Container 다루기

---
<a name="2-1"></a>
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
<a name="2-2"></a>
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
<a name="2-3"></a>
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
<a name="2-4"></a>
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
<a name="2-5"></a>
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


`host와 binding된 port만을 확인하려면 docker port 명령을 사용. `

`# docker port wordpress`

80/tcp -> 0.0.0.0:32769

-_0.0.0.0 은 host의 활용 가능한 모든 network interface에 binding 함을 뜻함__


각자에 docker port 명령에서 나온 port와 각 host의 ip로 wordpress web server에 접근이 가능함.



docker run의 옵션
* d
 ![그림2.9](https://github.com/ALGOSOPT/DockerConcept/blob/master/picture/2.9.jpg)
   ```
   -d : -i -t 가 container 내부로 진입하도록 attach 가능한 상태로 설정한다면 -d는 Detached 모드로 
        container를 실행함.
        Detached 모드는 container를 back ground에서 동작하는 application으로써 실행하도록 함.
   ```

      

       앞에서 우분투, CentOS container를 생성한 것처럼 -i, -t option으로 run을 실행하면 표준 입출력이 활성화된, 
       상호작용이 가능한 shell 환경을 사용할 수 있음.  ubuntu:14.04, centos:7과 같은 대부분의 기본 이미지들은 
       container를 시작할 때 /bin/bash를 command로 설정해 실행함으로써 bash shell을 쓸 수 있게 설정. 
       docker ps 명령어로 container 목록을 확인할 때 COMMAND에 표시되는 /bin/bash가 바로 여기에 해당함.

       그러나 -d 옵션으로 run을 실행하면 입출력이 없는 상태로 container를 실행함. container 내부에서 
       program이 terminal을 차지하는 __foreground__ 로 실행돼 사용자의 입력을 받지 않음.  
       Detached mode인 Container는 반드시 container내에서 program이 실행돼야 하며, foreground
       program이 실행되지 않으면 container는 종료함.

       mysql은 하나의 terminal을 차지하는 mysqld를, word press는 하나의 terminal을 차지하는 
       apache2-foreground를 실행하므로 -d option을 지정해 background로 설정한 것.



   ```
   # docker run -d --name detach_test ubuntu:14.04
   ```

   container가 생성됐더라도 바로 종료되므로 docker ps 명령어로 확인할 수 없음. 

   docker ps -a 명령어로 상태를 확인


   docker start 로 container를 시작해도 container 내부에  terminal을 차지하는 foreground로써 동작하는 프로그램이 없으므로
   container는 시작되지 않음. 그렇다면 반대로 mysql container를 -i -t option으로 생성하면,

   ```
   # docker run -i -t --name mysql_attach_test -e MYSQL_ROOT_PASSWORD=password \
     -e MYSQL_DATABASE=wordpress mysql:5.7
   ```

   하나의 terminal을 차지하는 mysqld 프로그램이 foreground로 실행된 log를 볼 수 있음. MYSQL image가 container에서 시작될 때
   mysqld가 동작하도록 설정돼 있기 때문입니다. 이 상태에서는 상호 입출력이 불가능하고 단순히 프로그램이 foreground mode로 동작하는 것
   만 지켜볼 수 있음. 이 같은 이유로 -d 옵션을 설정해 container가 background에서 동작하게 하는 것.




* e
   ```
   -e : -e 옵션은 container 내부의 환경변수를 설정함. container화된 application은 환경변수에서 값을 가져와
        쓰는 경우가 많으므로 자주 사용하는 옵션 중 하나. mysql container를 생성할 때 설정한 -e 옵션의 값을
        살펴보면 mysql container의 환경변수로 어떤 것이 설정되었는지 알 수 있음.
   ```

   ```
   # ... -e MYSQL_ROOT_PASSWORD=password ...
   ```
   container의 MYSQL_ROOT_PASSWORD 환경변수의 값을 password로 설정한다는 의미. 그렇다면 이 값이 실제로 container
   에 적용됐는지 확인. linux에서 환경변수를 확인하는 가장 간단한 방법은 아래와 같이 echo를 사용하는 것.
   
   ```
   # echo ${ENVIRONMENT_NAME}
   ```
   
   컨테이너 내부에서 echo로 환경변수를 출력하면 -e 옵션에 입력된 대로 값이 설정돼 있음을 확인할 수 있음.
   
   ```
   container /# echo $MYSQL_ROOT_PASSWORD
   ```
   
   그러나 echo 명령어를 사용하려면 상호 입출력이 가능한 bash shell을 사용할 수 있는 환경이 필요.
   입출력이 가능한 shell 환경을 사용하려면 docker attach 명령어로 container내부로 들어가야 하지만 
   위에서 생성한 mysql container는 -d 옵션으로 생성됐으므로 attach 명령어를 쓰는 것이 의미가 없음.
   
   attach를 쓰면 container에서 실행 중인 program의 log 출력을 보게 될 뿐.
   그러나 exec 명령어를 이용하면 container 내부의 shell을 사용할 수 있음.
   다음 며영어를 입력하면 mysql container 내부에 /bin/bash 프로세스를 실행하고 -i -t 옵션을 사용해서 
   bash shell을 쓸 수 있게 유지.
   
   ```
   # docker exec -i -t wordpressdb /bin/bash
   ```
   
   exec 명령어를 사용하면 container 내부에서 명령어를 실행한 뒤 그 결괏값을 반환받을 수 있음.
   그러나 여기서는 -i -t 옵션을 추가해 /bin/bash를 상호 입출력이 가능한 형태로 exec 명령어를 사용.
   옵션을 추가하지도 않고 단순히  exec만 쓰면 container 내부에서 실행한 명령어에 대하 결과를 반환
   
   ```
   # docker exec wordpressdb ls
   ```
   
   설정된 환경변수가 실제로 MySQL에 사용됐는지 확인하려면  container 내부에서 mysql -u root -p를 입력한 뒤
   password를 입력함.
   
* link
    ```
    --link : A container에서 B container로 접근하는 방법 중 가장 간단한 것은 NAT로 할당받은 내부 IP를 
             쓰는 것. B container의 IPrk 172.17.0.3이라면 A container는 이 IP를 써서 B container
             에 접근할 수 있음. 그러나 docker engine은 container 내부 IP를 172.17.0.2, 3, 4 ...
             와 같이 순차적으로 할당. 이는 container를 시작할 때마다 재할당하는 것이므로 매번 변경되는
             container의 IP로 접근하기가 어려움.
             
             --link option은 내부 IP를 알 필요 없이 항상 container의 별명(alias)으로 접근하도록 설정.
             위에서 생성한 wordpress web container는 --link 옵션의 값에서 wordpressdb container를
             mysql라는 이름으로 설정
             
             ... --link wordpressdb:mysql ...
             
             즉, wordpress web server container는 wordpressdb의 IP를 몰라도 mysql이라는 host 명으로
             접근할 수 있게 됨. wordpress web server container에서 mysql이라는 host 이름으로 ping을
             전송하면 wordpressdb의 내부 IP로 접그하는 것을 확인할 수 있음.
             
             # docker exec wordpress ping -c 2 mysql
    ```
    --link 옵션을 쓸 때 유의할 점은 --link에 입력된 container가 실행 중이지 않거나 존재하지 않는다면 --link를 
    적용한 container 또한 실행할 수 없다는 것. 이를 확인하는 방법은
    
    ```
    # docker stop wordpress wordpressdb
    
    (참고) start stop restart 등의 명령어를 사용할 때는 여러개의 container 이름을 순서대로 입력할 수도 있음.
    ```
   
   wordpressdb container가 정지된 상태에서 wordpress container를 실행하면 다음과 같은 오류가 출력됨
   즉, --link옵션은 container를 연결해주는 것 뿐만 아니라 container의 실행 순서의 의존성도 정의해준다는 것을
   알 수 있음.
   
   
---
<a name="2-6"></a>
### 2.6 Docker Volume

docker image로 container를 생성하면 image는 읽기 전용이 되며 container의 변경 사항만 별도로 저장해서 각 
container의 정보를 보존함. 예를 들어, 위에서 생성했던 mysql container는 mysql:5.7이라는 image로 생성됐
지만 wordpress blog를 위한 database 등의 정보는 container가 갖고 있음.

![그림2.10](https://github.com/ALGOSOPT/DockerConcept/blob/master/picture/2.10.jpg)


이미 생성된 image는 어떠한 경우로도 변경되지 않으며, container 계층에 원래 image에서 변경된 file system 등을
저장. image에 mysql을 실행하는 데 필요한 application file이 들어있다면 container 계층에서는 wordpress
에서 쓴 login 정보나 게시글 등과 같이 database를 운용하면서 쌓이는 data가 저장됨.



그러나 여기에는 치명적인 단점이 있음. mysql container를 삭제하면 container layer에 저장돼있던 database의 정보
도 삭제된다는 점. docker container는 생성과 삭제가 매우 쉬우므로 실수도 container를 삭제하면 data를 복구할 수 
없게 됨. 이를 방지하기 위해 container의 data를 영속적(Persistent) data로 활용할 수 있는 방법이 몇가지 있음.
그 중 가장 활용하기 쉬운 방법이 바로 **Volume** 을 활용하는 것.


Volume을 활용하는 방법은 여러가지가 있음. host와 volume을 공유할 수도 있고, Volume Containe를 활용할 수도 있으며,
docker가 관리하는 Volume을 생성할 수도 있음. 첫 번째로 host와 Volume을 공유함으로써 database container를 삭제해도
data는 삭제되지 않도록 설정해보자.

<a name="2-6-1"></a>
#### 2.6.1 Host Volume 공유
   
   mysql database container와 wordpress web server container를 생성.
   
   ```
   # docker run -d \
     --name wordpressdb_hostvolume \
     -e MYSQL_ROOT_PASSWORD=password \
     -e MYSQL_DATABASE=wordpress \
     -v /home/wordpress_db:/var/lib/mysql \
     mysql:5.7
   ```
   
   ```
   # docker run -d \
     -e WORDPRESS_DB_PASSWORD=password \
     --name wordpress_hostvolume \
     --link  wordpressdb_hostvolume:mysql \
     -p 80 \
     wordpress
   ```
   
   wordpress container에 -p 옵션으로 port를 외부에 노출했으므로 docker ps 명령어에서 확인한 wordpress_hostvolume container의
   host port로 wordpress container에 접속할 수 있음.
   
   이전 예제에서 database container를 생성할 때 사용한 run 명령어의 옵션과 달라진 점은 -v option을 추가했고, 그 값을
   /home/wordpress_db:/var/lib/mysql로 설정한 것. 이는 host의 /home/wordpress_db 디렉토리와 container의
   /var/lib/mysql 디렉토리를 공유한다는 뜻.
   
   __즉, [host의 공유 디렉토리]:[container의 공유 디렉토리] 형태.__
   
   
   미리 /home/wordpress_db 디렉토리를 host에 생성하지 않았어도 docker는 자동으로 이를 생성.
   실제로 해당 디렉토리에 database관련 파일이 있는지 확인하자.
   
   ```
   # ls /home/wordpress_db
   ```
   
   mysql을 구동하는데 필요한 각종 파일이 공유됨. mysql, performance_schema, sys, wordpress 디렉터리는 
   mysql에 존재하는 실제 database에 대응됨.
   
   container를 database의 data가 보존되는지 확인하자.
   다음 두 명령어를 입력해 방금 생성한 2개의 container를 삭제.
   
   ```
   # docker stop wordpress_hostvolume wordpressdb_hostvolume
   
   # docker rm wordpress_hostvolume wordpressdb_hostvolume
   ```
   
   다시 /home/wordpress_db 디렉터리를 확인하면 mysql container가 사용한 data가 그대로 남은 것을 확인할 수 있음.
   
   ```
   # ls /home/wordpress_db
   ```
   
   -v option을 써서 Container의 directory host와 공유한 것을 그림으로 나타내면 다음과 같음.
   Container의 /var/lib/mysql 디렉터리는 host의 /home/wordpress_db 디렉터로이와 동기화 되는 것이 아니라
   완전히 같은 directory.
   
   
   ![그림2.11](https://github.com/ALGOSOPT/DockerConcept/blob/master/picture/2.11.jpg)
   
  ```
  (참고)
  
  디렉터리 단위의 공유뿐 아니라 단일 파일 단위의 공유도 가능하며, 동시에 여러 개의 -v 옵션을 쓸 수도 있음.
  
   # echo hello >> /home/hello && echo hello2 >> /home/hello2
   
   # docker run -i -t --name file_volume \
     -v /home/hello:/hello -v /home/hello2:/hello2 \
     ubuntu:14.04
     
   root@ :/# cat hello && cat hello2
  ```
  
  위 예시의 경우 원래 host에는 /home/wordpress_db 디렉토리가 존재하지 않았음. -v option을 사용함으로써 host에
  /home/wordpress_db 디렉토리가 생성됐고, 이 디렉토리에 파일이 공유됨. 결과적으로 container file이 host로 파일이 
  복사 된 것. 
  
  
  ![그림2.11](https://github.com/ALGOSOPT/DockerConcept/blob/master/picture/2.11.jpg)
  
  그렇다면 host에 이미 디렉터리와 file이 존재하고 container도 존재 할 때 두 directory를 공유하면 어떻게 될까?
  
  이를 확인하기 위해 alicek106/volume_test라는 미리 준비된 image를 활용하겠음. 이 image의 /home 디렉터리에는
  testdir_2 라는 directory가 존재하고 test라는 file이 그 안에 들어있음.
  
  다음 명령어를 입력해 container를 생성한 뒤 /home/testdir_2 라는 directory를 확인하면 file이 존재하는 것을
  알 수 있음.
  
  ```
  # docker run -i -t --name volume_dummy alicek106/volume_test
  
  root@ :/# ls /home/testdir_2/
  ```
  
  container를 빠져나온 뒤 -v 옵션으로 container를 생성. -v 옵션의 값인 /home/testdir_2 directory를 확인하면
  원래 존재하던 directory에 host의 volume을 공유하면 container의 directory 자체가 덮어 씌어짐.
  
  정확히 말하면 -v 옵션을 통한 host volume host 공유는 host의 directory container의 directory에 mount함.
  
  ```
  # docker run -i -t \
    --name volume_overide \
    -v /home/wordpress_db:/home/testdir_2 \
    alicek106/volume_test
    
  
  root@ :/# ls /home/testdir_2/
  
  ```
  
  
  ```
  (참고) run 명령어를 실행하면서 alice106/volume_test 이미지가 docker에 존재하지 않으므로 image를 pull함.
  그러나 tag를 지정하지 않았는데 image를 pull된 것은 image의 tag를 지정하지 않으면 docker engine이 latest
  tag로 지정된 image를 pull하기 때문.
  ```
  


<a name="2-6-2"></a>
#### 2.6.2 Volume Container

볼륨을 사용하는 두 번째 방법은 -v option으로 볼륨을 사용하는 container를 다른 container와 공유하는 것. 
container를 생성할 때 --volumes-from 옵션을 설정하면 -v 또는 --volume 옵션을 적용한 container의 volume directory
를 공유할 수 있음.

그너라 이는 직접 volume을 공유하는 것이 아닌 -v 옵션을 적용한 container를 통해 공유하는 것.

아래의 예제는 위에서 생성한 volume_override container에서 volume을 공유받는 경우.

앞에서 생성한 volume_overide container는 /home/testdir_2 디렉토리를 host와 공유하고 있으며, 

이 container로서 volumes_from_container 컨테이너에 다시 공유하는 것.

```
# docker run -i -i \
  --name volumes_from_container \
  --volumes-from volume_overide \
  ubuntu:14.04
  
  
root@:/# ls /home/testdir_2/
```

--volumes-from 옵션을 적용한 container와 volume container 사이의 관계를 나타내면 다음 그림과 같다.

![그림2.13](https://github.com/ALGOSOPT/DockerConcept/blob/master/picture/2.13.jpg)


여러개의 container가 동일한 container에 --volumes-from 옵션을 사용함으로써 볼륨을 공유해 사용할 수도 있음.
이러한 구조를 활용하면 host에서 volume만 공유하고 별도의 역할을 담당하지 않는 일명 volume container로서
활용하는 것도 가능.

즉, 볼륨을 사용하려는 container에 -v 옵션 대신 --volumes-from 옵션을 사용함으로써 volume container에 연결해
data를 간접적으로 공유받는 방식.



<a name="2-6-3"></a>
#### 2.6.3 Docker Volume

Volume을 활용하는 세 번째 방법은 docker volume 명령어를 사용하는 것. 
지금까지 한 방법과 같이 host와 volume을 공유해 container의 data를 보존하거나
--volumes-from 옵션을 활용하는 것도 나쁘지 않지만 docker 자체에서 제공하는 volume 기능을
활용해 data를 보존할 수도 있음.



volume을 다루는 명령어는 docker volume으로 시작하며, docker volume create 명령어로 volume을 생성.

다음 명령은 myvolume이라는 volume을 생성

```
# docker volume create --name myvolume
```


생성된 volume을 확인
```
# docker volume ls
```

볼륨을 생성할 때 plug in driver를 설정해 여러 종류의 storage backend를 쓸 수 있지만 여기서는
기본적으로 제공되는 driver인 local을 사용함. 

이 volume은 local host에 저장되며 docker engine에 의해 생성되고 삭제 됨.


다음 명령어를 입력해 myvolume이라는 볼륨을 사용하는 container를 생성.
host와 volume을 공유할 때 사용한 -v 옵션의 입력과는 다르게 다음과 같은 형식으로 입력해야 함.

__[Volume의 이름]:[Container의 공유 디렉터리]__

아래의 예시에서 생성되는 container는 volume을 container의 /root/ directory에 mount하므로 /root directory에
파일을 쓰면 해당 file이 volume에 저장됨.

```
# docker run -i -t --name myvolume_1 \
  -v myvolume:/root/ \
  ubuntu:14.04
  
  
  root@ :/# echo hello, volume! >> /root/volume
```

/root directory에 volume이라는 file을 생성함. 다른 container도 myvolume volume을 쓰면
volume을 활용한 directory에 volume file이 존재할 것임. container에서 host로 빠져나온 뒤
container를 생성해 확인해보자

```
# docker run -i -t --name myvolume_2 \
  -v myvolume:/root/ \
  ubuntu:14.04
  
  
  root@ :/# cat /root/volume
```

결과를 보면 같은 file인  volume이 존재함. 
docker volume 명령어로 생성한 volume은 아래 그림과 같은 구조로 활용됨.
docker volume도 여러 개의 container에 공유되어 활용될 수 있음.

![그림2.14](https://github.com/ALGOSOPT/DockerConcept/blob/master/picture/그림2.14.jpg)


Volume은 directory 하나에 상응하는 docker engine에서 관리함. docker volume도 host volume 공유와
마찬가지로 host에 저장함으로써 data를 보존하지만 file이 실제로 어디에 저장되는지 사용자는 알 필요가 없음.

docker inspect 명령어를 사용하면 myvolume 볼륨이 실제로 어디에 저장되는지 알 수 있음.
docker inspect 명렁어는 container, image, volume 등 docker의 모든 구정 단위의 정보를 확인할 때 사용되며,
정보를 확인할 종류를 명시하기 위해 --type 옵션에 image, volume 등을 입력하는 것이 좋음.

다음 명령은 이름이 myvolume인 볼륨의 정보를 출력.

```
# docker inspect --type volume myvolume
```

```
(참고) docker의 모든 명령어는 docker 접두사 다음에 container, image, volume등을 명시함으로써 특정 구성 단위를
제어하는 명령어를 사용할 수 있음. 예를 들어 docker container inspect는 container의 정보를, 
docker volume inpect는 volume의 정보를 출력함.
```


Driver는 Volume이 쓰는 드라이버를, Label은 Volume을 구분하는 label을 나타내며, Mountpoint는 해당 volume이 실제로
host의 어디에 저장됐는지를 의미함.

그러나 volume을 쓰는 사용자 입장에서 Mountpoint를 알 필요는 없음.

해당 directory file을 살펴보면 container에서 사용했던 파일이 남아 있음을 알 수 있음.

```
# ls /var/lib/docker/volumes/myvolume/_data

# cat /var/lib/docker/volumes/myvolume/_data/volume
```


docker volume create 명령을 별도로 입력하지 않아도 -v option을 입력할 때 이를 수행하도록 설정 할 수 있음.

다음과 같이 container에서 공유할 directory의 위치를 -v option에 입력하면 해당 directory에 대한 volume을
자동으로 생성함.


```
# docker run -i -t --name volume_auto -v /root ubuntu:14.04
```


Container를 생성한 뒤 host로 빠져나와 docker volume ls 명령어로 확인하면 이름이 무작위의 16진수 형태인
Volume이 자동으로 생성된 것을 볼 수 있음.


```
# docker volume ls
```

생성된 volume_auto container가 위의 dae1b4a ... volume을 사용하는지 확인하는 다른 방법은 do
docker container inspect 명령어를 이용하는 것.
docker container inspect 명령어는 container의 상세한 정보를 출력하는데, 

그중 volume mount에 대한 정보도 포함돼 있기 때문.

```
# docker container inspect volume_auto
```


inspect 명령어는 많은 정보를 출력하믈 대부분의 출력 결과를 생략함.
여기서 주목할 정보는 "Source" 항목에 정의된 directory인 /var/lib/... 이 volume_auto
container에 mount되어 Volume으로 쓰고 있다는 것.

이렇게 Docker Volume을 생성하고 삭제하다 보면 불필요한 Volume들이 남아 있을 때가 있음.
docker volume을 사용하고 있는 container를 삭제해도 
Volume이 자동으로 삭제되지는 않기 때문.

사용되지 않는 Volume을 한꺼번에 삭제하려면 docker volume prune 명령어를 사용.

```
# docker volume prune
```

지금까지 Volume 공유를 통한 data 저장에 대해 알아보았음.

이처럼 Container가 아닌 외부에 data를 저장하고 Container는 그 data로 동작하도록 설계하는 것을
__stateless__ 하다고 말함.

Container 자체는 상태가 없고 상태를 결정하는 data는 외부에서 제공받음.
Container가 삭제 되도 data는 보존되므로 stateless한 container 설계는 docker 를 사용할 때
매우 바람직한 설계

이와 반대로 container가 data를 저장하고 있어 상태가 있는 경우 __stateful__ 하다고 말함. 
stateful 한 container는 container 자체에서 data를 보관하므로 지양하는 것이 좋음.



---
<a name="2-7"></a>
### 2.7 Docker Network

##### 2.7.1 Docker Network 구조

이 전에 Container 내부에서 ifconfig 를 입력해 Container의 network interface에 eth0과 lo 네트워크 인터페이스가 있는 것을 확인했음.

```
# root@ /# ifconfig

```

이전에 언급한 바와 같이 Docker는 Container에 내부 IP를 순차적으로 할당하며, 이 IP는 Container를 재시작할 때 마다
변경될 수 있음. 이 내부 IP는 Docker가 설치된 host, 즉 내부 망에서만 쓸 수 있는 IP이므로 외부와 연결될 필요가 있음.

이 과정은 Container를 시작할 때마다 Host에  Veth... 라는 Network Interface를 생성함으로써 이루어짐.

Docker는 각 Container에 외부와의 Network를 제공하기 위해 Container 마다 가상 network interface를 host에 생성하며

이 interface의 이름은 veth로 시작함.

veth interface는 사용자가 직접 생성할 필요는 없으며 Container가 생성될 때 Docker Engine이 자동으로 생성.

```
(참고) veth에서 v는 virtual을 뜻함. 즉 virtual eth라는 의미
```


Docker가 설치된 host에서 ifconfig나 ip addr과 같은 명령어오 network interface를 확인하면 실행 중인 Container 수 만큼
veth로 시작하는 interface가 생성된 것을 확인할 수 있음.



```
# ifconfig

veth 확인

```

출력 결과에서 eth0은 공인 IP 또는 내부 IP가 할당되어 실제로 외부와 통신할 수 있는 host의 network interface.

veth로 시작하는 interface는 container를 시작할 때 생성됐으며, 각 container의 eth0과 연결함.


veth interface뿐 아니라 docker0 이라는 bridge도 존재하는데, docker0 bridge는 각 veth interface와 binding돼
host의 eth0 interface와 이어주는 역할을 함. 즉 container와 host의 network는 아래 그림과 같은 구성

[그림 2.15]


정리하면 container의 eth0 interface는 host의 veth...라는 interface와 연결됐으며 veth interface는 docker0 bridge와
binding돼 외부와 통신할 수 있음.


```
(참고) brctl 명령어를 이용해 docker0 bridge에 veth이 실제로 binding되어있는지 
알 수 있음.

# brtcl show docker0

```

#### 2.7.2 Docker Network 기능

Container를 생성하면 기본적으로 docker0 bridge를 통해 외부와 통신할 수 있는 환경을 사용할 수 있지만, 

사용자의 선책에 따라 여러 네트워크 드라이버를 쓸 수 도 있음.

docker가 자체적으로 제공하는 대표적인 network driver는 bridge, host, non, container, overlay가 있음.

third party plugin solution으로는 weave, flannel, openswitch 등이 있으며, 더 확장된 network 구성을 위해 활용됨.

이번 장에서 bridge, non, host, container에 대한 설명을 할 예쩡



먼저 Docker에서 기본적으로 쓸 수 있는 network는 무엇이 있는지 확인해보자.
Docker network ls 명령어오 network 목록을 확인.

Docker의 network를 다루는 명령어는 docker network로 시작

```
# docker network ls
```


이미 bridge, host, none   network가 있는 것을 알 수 있다. bridge network는 container를 생성할 때 자동으로
연결되는 docker0 bridge를 활용하도록 설정돼어 있음.

이 network는 172.17.0.x IP 대역을 container에 순차적으로 할당. docker network inspect 명령어를 이용하면 
network의 자세한 정보를 살펴볼 수 있음.

docker inspect --type network를 사용해도 동일한 출력 결과를 볼 수 있음.


```
# docker network inspect bridge
```
Config 항목의 subnet과 gateway가 172.17.0.0/16과 172.17.0.1로 설정되어 있음.
또한 bridge network를 사용 중인 container의 목록을 Containers 항목에서 확인할 수 있음.
아무런 설정을 하지 않고 Container를 생성하면 Container는 자동으로 docker0 bridge를 사용.


__Bridge Network__

Bridge Network는 docker0이 아닌 사용자 정의 bridge를 생성해 각 container에 연결하는 network 구조.
Container는 연결된 Bridge를 통해 외부와 통신할 수 있음.


기본적으로 존재하는 docker0을 사용하는 bridge network가 아닌 새로운 bridge type의 network를 생성할 수 있음.

다음 명령어를 입력해 새로운 bridge network를 생성.

```
# docker network create --driver bridge mybridge
```
bridge type의 mybridge라는 network가 생성됨.

docker run 또는 create 명령어에 --net 옵션의 값을 설정하면 container가 이 network를 사용하도록 설정할 수 있음.

다음 명령어를 입력해 mybridge network를 사용하는 container를 생성

```
# docker run -i -t --name mynetwork_container \
  --net mybridge0 \
  ubuntu:14.04
```

container 내부에서 ifconfig를 입력하면 새로운 IP 대역이 할당된 것을 알 수 있음.

Bridge type의 network를 생성하면 Docker는 IP대역을 차례대로 할당함.

다음 예시에서는 172.18 대역의 내부 IP가 할당됨.

```
root@/# ifconfig

```

network의 subnet, gateway, IP 할당 범위 등을 임의로 설정하려면 network를 생성할 때 아래와 같이

--subnet, --ip-range, --gateway 옵션을 추가함.

단, --subnet과 --ip-range는 같은 대역이여야 함.

```
# docker network create --driver=bridge \
  --subnet=172.72.0.0/16 \
  --ip-range=172.82.0.0/24 \
  --gateway=172.72.0.1 \
  my_custom_network
```


__host nerwork__

network를 host로 설정하면 host의 network 환경을 그대로 쓸 수 있음.
위의 bridge driver network와는 달리 host driver의 network는 별도로 생성할 필요 없이
기존의 host라는 이름의 network를 사용함

```
root@/# docker run -i -t --name network_host \
        --net host \
        ubuntu:14.04 


     # echo "컨테이너 입니다"
```

--net 옵션을 입력해 host를 설정한 container의 내부에서 network환경을 확인하면 host와 같은 것을 알 수 있음.
host machine에서 설정한 host이름도 container가 물려받기 때문에 container의 host이름도 무작위
16진수가 아닌 docker engine이 설치된 host machine의 host이름으로 설정됨.

```
root@docker-host:/# echo "컨테이너 내부입니다."
root@docker-host:/# ifconfig
```

Container의 network host mode로 설정하면 container 내부의 application을 별도의 port forwarding 없이 
바로 서비스할 수 있음. 이는 마치 실제 host에서 application을 외부에 노출하는 것과 같음.

예를 들어, host mode를 쓰는 Container에서 아파치 웹 서버를 구동한다면 host의 IP와 Container의 아파치
웹서버 포트인 80으로 바로 접근가능


__None network__

none은 말 그대로 아무런 nerwork를 쓰지 않는 것을 뜻함.
다음과 같이 Container를 생성하면 외부와 연결이 단절됨.

```
# docker run -i -t --name network_none \
  --net none \
  ubuntu:14.04
```

--net 옵션으로 none을 설정한 container 내부에서 network interface를 확인하면 local host를 
나타내는 lo 외에는 존재하지 않는 다는 것을 알 수 있음.

```
# ifconfig
```


__Container Network__

--net option으로 container를 입력하면 다른 Container의 network환경을 공유할 수 있음.
공유되는 속성은 내부 IP, network interface의  MAC address 등. --net 옵션의 값으로
container:[다른 Container의 ID]와 같이 입력

```
# docker run -i -t --name network_container_1 ubuntu:14.04

# docker run -i -t -d --name network_container_2 \
  --net container:network_container_1 \
  ubuntu:14.04
```


```
(참고) -i, -t, -d 옵션을 함께 사용하면 Container 내부에서 Shell을 실행하지만 내부로 들어가지 않으며  
Container도 종료되지 않음. 위와 같이 test용으로 Container를 생성할 때 유용하게 쓸 수 있음.

```

위와 같이 다른 Container의 network 환경을 공유하면 내부 IP를 새로 할당받지 않으며 host에 veth로 시작하는
가상 network interface도 생성되지 않음. network_contianer_2 container의 network와 관련된 사항은 전부
network_container_1과 같게 설정됨.

다음 명령어를 입력해 이를 확인할 수 있음.

```
# docker exec network_container_1 ifconfig

# docker exec network_container_2 ifconfig

```

두 container의 eth0에 대한 정보가 완전히 같은 것을 알 수 있음.

이를 그림으로 나타내면 아래와 같음.

[그림2.16]



__Bridge network와 --net-alias

bridge type의 network와 run 명령어의 --net-alias 옵션을 함께 쓰면 특정 host 이름으로 container 여러 개에 접근할 수 있음.
위에서 생성한 mybridge network를 이용해 container를 3개 생성해보자. --net-alias 옵션의 값은 alicek106으로 설정했으며,
다른 Container에서 alicek10g6이라는 host 이름으로 아래 3개의 container에 접근할 수 있음.

```
# docker run -i -t -d --name network-alias_container1 \
  --net mybridge \
  --net-alias alicek106 ubuntu:14.04
  
  
# docker run -i -t -d --name network_alias_container2 \
  --net mybridge \
  --net-alias alicek106 \
  ubuntu:14.04
  
# docker run -i -t -d --name network_alias_container3 \
  --net mybridge \
  --net-alias alicek106 \
  ubuntu:14.04
  
```

inspect 명령어오 각 container의 IP를 확인해보자
```
# docker inspect network_alias_container1 | grep IPAddress
```

첫 번째 Container의 IP 주소가 172.18.0.3 이므로 두, 세 번째 container는 각각 172.18.0.4, 5일 것임.

세 개의 Container에 접근할 Container를 생성한 뒤 alicek106이라는 host이름으로 ping 요청을 전송해봄.

```
# docker run -i -t --name network_alias_ping \
  --net mybridge \
  ubuntu:14.04
  
root@:/# ping -c 1 alicek106

root@:/# ping -c 1 alicek106

root@:/# ping -c 1 alicek106
```

Container 3개의 IP로 각각 Ping이 전송된 것을 확인할 수 있음.

매번 달라지는 IP를 결정하는 것은 별도의 알고리즘이 아닌 round robin방식.

이 것이 가능한 이유는 Docker Engine에 내장된 DNS가 alicek106 이라는 host 이름을
--net-alias 옵션으로  alicek106을 설정한 container로 변환(resolve)하기 때문.

다음 그림은 docker network 에서 사용하는 내장 DNS와 --net-alias의 관계를 보여줌

[그림2.17]


Docker의 DNS는 host 이름으로 유동적인 Container를 찾을 때 주로 사용됨.

가장 대표적인 예가 --link 옵션인데, 이는 Container의 IP가 변경돼도 별명으로 Container를 찾을 수 있게 DNS에 
의해 자동으로 관리됨.

단, 이 경우는 Default bridge network의 container DNS라는 점이 다름.

--net-alias option 또한 --link option과 비슷한 원리도 작동함.

Docker는 기본 Bridge network 사용자가 정의한 bridge network에 사용되는 내장 DNS 서버를 가지며, DNS의 IP는

127.0.0.11 임. mybridge라는 이름의 network에 속한 3개의 container는 run으로 생성할 때 --net-alias 옵션에

alicek106이라는 hsot 이름으로 등록됨.

mybridge network에 속한 container에서 alicek106이라는 host 이름으로 접근하면 DNS 서버는 round-robin 방식을 
이용해 Container의 IP list를 반환함. 

ping 명령어는 이 IP list에서 첫 번째 IP를 사용하므로 매번 다른 IP로 ping을 전송함.

이를 확인하기 위해 dig라는 도구를 사용해보자
dig는 DNS로 domain 이름에 대응하는 IP를 조회할 때 쓰는 도구.

dig는 ubuntu:14.04 image에 설치돼 있지 않으므로 방금 생성한 network_alias_ping container 내부에서 다음
명령어를 입력해 설치람

```
root@ :/# apt-get update
root@ :/# apt-get install dnsutils
```

dig 명령어로 alicek106 host 이름이 반환되는 IP를 확인함. 

```
root@:/# dig alicek106
```
위 명령어를 반복해서 입력해보면 반환되는 IP의 list 순서가 모두 다른 것을 알 수 있음.



---
### 2.8 Container Logging

#### 2.8.1 json-file 로그 사용하기

Container 내부에서 어떤 일이 일어나는지 아는 것은 debugging 뿐만 아니라 운영측면에서도 중요함.

application level에서 log가 기록되도록 개발해 별도의 logging service를 쓸 수도 있지만 Docker는 container의 표준 출력(Stdout)
과 에러(StdErr) log를 별도의 metadata file로 저장하며 이를 확인하는 명령어를 제공함.

먼저 container를 생성해 간단한 log를 남겨보다. 

다음 명령은 mysql5.7 버전의 container를 생성함.

```
# docker run -d --name mysql \
  -e MYSQL_ROOT_PASSWORD=1234 \
  mysql:5.7
```
mysql과 같은 application을 구동하는 container는 foreground mode로 실행되므로 -d 옵션을 써서 background mode로
container를 생성하는 경우가 많음. 따라서 application이 잘 구동되는지 여부를 알 수 없지만 docker logs 명령어를 써서
Container의 상태를 알 수 있음. 

docker logs 명령어는 container 내부에서 출력을 보여주는 명령어임.

```
# docker log mysql

```

이번에는 다른 방법으로 container를 생성해보자. 동일한 mysql container를 생성하되,
-e 옵션을 제외해야함.

```
# docker run -d --name no_passwd_mysql \
  mysql:5.7
```

위 명령어를 실행한 뒤 docker ps 명령어로 container의 목록을 확인하면 mysql2 container는 
생성 되었으나 실행되지 않음.

docker start 명령어로 다시 시작해도 container는 시작되지 않음.

```
# docker ps --format "table {{.ID}}\t{{.Status}}\t{{.Ports}}\t{{.Names}}"

# docker start no_passwd_mysql

# docker ps --format "table {{.ID}}\t{{.Status}}\t{{.Ports}}\t{{.Names}}"

```

이럴 때 docker logs 명령어를 사용하면 application에 무슨 문제가 있는지 확인할 수 있음.
위의 경우에는 mysql 실행에 필요한 환경변수를 지정하지 않아 Container가 시작되지 않은 것.

이처럼 Container가 정상적으로 실행 및 동작하지 않고 docker attach 명령어도 사용하지 
못하는 개발환경에서 docker logs 명령어를 쓰면 간단하고 빠르게 에러를 확인할 수 있음.


```
# docker logs no_passwd_mysql

```

container의 log가 너무 많아 읽기 힘들다면 --tail 옵션을 써서 마지막 log 줄부터 출력할 줄의 수를
설정할 수 있음. 다음 예시에서는 container의 log 중 마지막 2줄만 출력함.

```
# docker logs --tail mysql
```

--since 옵션에서 유닉스 시간을 입력해 특정 시간 이후의 log를 확인할 수 있으며 -t 옵션으로 timestamp
를 표시할 수도 있음. Container에서 실시간으로 출력되는 내용을 확인하려면 -f 옵션을 써서 log를
stream으로 확인할 수 있음. 특히 -f 옵셔은 application을 개발할 때 유용함.

```
# docker logs --since 1474765979 mysql
```

```
# docker logs -f -t mysql
```

docker logs 명령어는 run 명령어에서 -i -t 옵션을 설정해  docker attach 명령어를 사용할 수 있는 Container에도
쓸 수 있으며, Container 내부에서 bash shell 등을 입출력한 내용을 확인할 수 있음.

```
# docker run -i -t --name logstest ubuntu:14.04

# docker logs logstest
```

기본적으로 위와 같은 container log는 JSON 형태로 docker 내부에 저장됨.
이 파일은 다음경로에 container의 ID로 시작하는 file명으로 저장됨. 

아래의 log file의 내용을 cat, vi 편집기 등으로 확인하면

logs 명령으로 정제되지 않은 JSON data를 볼 수 있으.ㅁ


```
# cat /var/lib/docker/container/${CONTAINER_ID}/${CONTAINER_ID}-json.log
```


어떠한 설정도 하지 않았따면 docker는 위와 같이 container log를 JSON file로 저장하지만

그 밖에도 각종 logging driver를 사용하게 설정해 container log를 수집할 수 있음.

사용가능한 driver의 대표적인 예로  syslog, journalId, fluentd, awslogs 등이 있으며 application의 특징에 
적합한 logging driver를 선택함

```
(참고) logging driver는 기본적으로 json-file로 설정되어 있지만 docker daemon 시작 옵셔에서 --log-driver option을 써서
기본적으로 사용할 logging driver를 변경할 수 있음. 

DOCKER_OPTS="--log-driver=syslog"
```

#### 2.8.1 syslog 로그

Container의 Log는 JSON 뿐 아니라 syslog로 보내 저장하도록 설정할 수 있음. syslog는 유닉스 계열 운영체제에서 log를 
수집하는 오래된 표준 중 하나로서, kernel, security 등 system과 관련된 log, application log 등 다양한 종류의
log를 수집해 저장함. 대부분의 unix 계열 운영체제에서는 syslog를 사용하는 interface가 동일하기 때문에 체계적으로
log를 수집하고 분석할 수 있다는 장점이 있음.

다음 명령어를 입력해 syslog에 log를 저장하는 container를 생성함. 

container의 cmd가 echo syslogtest로 설정되기 때문에 syslogtest라는 문구를 출력하고 container는 종료될 것.

```
# docker run -d --name syslog_container \
  --log-driver=syslog \
  ubuntu:14.04 \
  echo syslogtest
```

syslog 로깅 driver는 기본적으로 local host의 syslog에 저장하므로 운영체제 및 배포판에 따라 syslog file의 위치를 알아야
이를 확인할 수 있음.

ubuntu 14.04는 /var/log/syslog, CentOS와 RHEL은 /var/log/messages file에서 ubuntu 16.04와 CoreOS는 journalctl -u
docker.service 명령어로 확인할 수 있음.

다음은 ubuntu에서 syslog가 기록된 예시

```
# tail /var/log/syslog
```


syslog를 원격 서버로 설치하면 log option을 추가해 log 정보를 원격 서버로 보낼 수 있음. 이번에는 syslog를 원격에 저장하는 방법의
하나인 rsyslog를 써서 중앙 container로 log를 저장해보자. 아래의 IP는 이번 예시에서 사용된 server의 IP이며, syslog를 사용할 서버와
client 2개의 machine을 따로 설정함.


```
server host : 192.168.0.100
client host : 192.168.0.101
```

server host에 rsyslog 서비스가 시작하도록 container를 구동하고, client host에서 container 생성해 server의 rsyslog를 저장.
server host에서 다음 명령어를 입력해 rsyslog container를 생성함.


```
server@192.168.0.100# docker run -i -t \
                      -h rsyslog \
                      --name rsyslog_server \
                      -p 514:514 -p 514:514/udp \
                      ubuntu:14.04
```

container 내부의 rsyslog.conf file을 열어 syslog server를 구동시키는 항목의 주석을 해제한 후 변경사항을 저장함.

```
root@:/# vi /etc/rsyslog.conf

...

#provides UDP syslog reception
$ModLoad imudp
$UDPServerRun 514



#provides TCP syslog reception
$ModLoad imtcp
$InputTCPServer 514
....
```

다음 명령어를 입력해 rsyslog 서비스를 재시작함.
```
root@:/# service rsyslog restart
```

```
(참고) rsyslog Container가 아닌 ubuntu, CentOS 등의 host에서도 쓸 수 있으며,
위 방법은 ubuntu를 기준으로 함. 그러나 Container를 쓰면 host가 어떤 운영체제이든
상관없이 rsyslog를 사용할 수 있음.
이 처럼 application을 container로 구현하면 docker engine이 설치 될 수 있는 운영
체제라면 어떤 환경이라도 간단히 배포할 수 있음.
```


Container를 빠져나온 뒤 Client host에서 아래의 명령어를 입력해 Container를 생성함. 
Container log를 기록하기 위해 간단한 echo 명령어를 실행.



```
client@192.168.0.101# docker run -i -t \
                      --log-driver=syslog \
                      --log-opt syslog-address=tcp://192.168.0.100:514 \
                      --log-opt tag="mylog" \
                      ubuntu:14.04
            
root@:/# echo test
```


--log-opt는 logging driver에 추가할 옵션을 뜻하며 syslog-address에 rsyslog container에 접근할 수 있는
주소를 입력함.
tag는 log data가 기록될 때 함께 저장될 tag이며 log를 분류하기 위해 사용됨.

다시 server의 rsyslog container로 되돌아와 container 내부의 syslog file을 확인하면 log가 전송된 것을
알 수 있음. 또한 각기 log앞에 mytag라는 명칭이 추가됨.


```
root@rsyslog:/# tail /var/log/syslog
```


```
(참고) rsyslog의 설정 file을 변경할 때 tcp와 udp의 두 방법 
모두 활성화 했으므로 syslog흫 쓸 때 tcp 뿐만 아니라 udp로 쓸 수도 있다.

# docker run -i -t \
  --log-driver=syslog \
  --log-opt syslog-address=udp://192.168.0.100:514 \
  --log-opt tag="mylog" \
  ubuntu:14.04
```


--opt-log 옵션으로 syslog-facility를 쓰면 log가 저장될 파일을 바꿀 수 있음.  facility는 log흫 생성하는 주체에 따라 log를 다르게 저장하는 것으로, 여러 application에서 수집되는 log를 분류하는 방법. 기본적으로 daemon으로 설정돼 있지만 kern, user, mail 등 다른 facility를 사용할 수 있음.

```
# docker run -i -t \
  --log-driver syslog \
  --log-opt syslog-address=tcp://192.168.0.100:514 \
  --log-opt tag="maillog" \
  --log-opt syslog-facility="mail" \
  ubuntu:14.04
```

facility 옵션을 쓰면 rsyslog 서버 container에 해당 facility에 해당하는 새로운 log file이 생성됨.

```
root@rsyslog:/var/log# ls /var/log/
```


rsyslog는 ubuntu에서 쓸 수 있는 기본적인 log 방법이므로 별도의 UI를 제공하지 않지만 logentries, LogAnalyzer 등과 같은 log 분석기와
연동하면 web interface를 활용해 편리하게 log를 확인할 수 있음.

---
### 2.9 Container 자원 할당 제한

container를 생성하는 run, create 명렁어에서 container의 자원 할당량을 조정하도록 option을 입력할 수 있음.
아무런 option을 입력하지 않으면 container는 host의 자원을 제한 없이 쓸 수 있게 설정되므로 제품 단계의 container를 고려한다면 container의 자원할당을 제한해 host와 다른 container의 동작을 방해하지 않게 설정하는 것이 좋음.

container의 자원할당 option을 설정하지 않으면, host의 자원을 전부 점유해 다른 container들 뿐 아니라 host 자체의 동작이 멈출 수도 있음.

현재의 container에 설정된 자원 제한을 확인하는 가장 쉬운 방법은 docker inspect 명령어를 입력하는 것.


자원 할당을 제한하기 위해 container에 적용할 수 있는 option은 많지만, 여기서는 대표적인 몇가지만 설명.

#### 2.9.1 Container memory 제한
docker run 명령어에 --memory를 지정해 container의 memory를 제한 할 수 있음. 입력할 수 있는 단위는 m, g.

제한할 수 있는 최소한의 memory는 4MB. 다음 명령어는 container의 memory사용량을 1GB로 제한함.

```
# docker run -d \
  --memory="1g" \
  --name memory_1g \
  nginx
```

위 명령어로 container를 생성한 뒤 inspect 명령어로 memory의 값을 확인하면 1GB에 해당하는 바이트 값이 설정됐음을
확인할 수 있음.

```
# docker inspect memory_1g | grep \"Memory\"
```

container 내에서 동작하는 process가 container에 할당된 메모리를 초과하면 container는 자동으로 종료되므로 application에 따라
메모리를 적절하게 할당하는 것이 좋음. 다음 명령어는 memory를 매우 적게 할당하는 경우로서 4MB의 메모리로 mysql container를 실행하면
memory가 부족해 container가 실행되지 않음.

```
# docker run -d --name memory_4m \
  --memory="4m" \
  mysql:5.7
  
  
# docker ps -a --format "table {{.ID}}\t{{.Status}}\t{{.Names}}"

```

기본적으로 container의 Swap memory는 memory의 2배로 설정되지만 별도로 지정할 수 있음.
다음 명령어는 Swap memory를 500MB로, memory를 200MB로 설정해 container를 생성

```
# docker run -i -t --name swap_500m \ 
  --memory=200m \
  --memory-swap=500m \
  ubuntu:14.04
```


#### 2.9.2 container CPU 제한

**--cpu-shares**
가상 머신이 특정 개수의 CPU를 할당받던 것과는 다르게 모든 container의 작업은 CPU scheduling에서 
같은 비율로 처리됨. 따라서 contaienr 하나에 CPU를 한 개 할당받는 방식이 아닌 CPU scheduling에서
CPU를 얼마나 마니 차지할 것인가를 설정해야함. 다음 명령어에서 --cpu-shares option은 container가
CPU를 얼마나 차지할 것인지를 설정함.

```
# docker run -i -t --name cpu_share \
  --cpu-shares 2048 \
  ubuntu:14.04
```

--cpu-shares option은 상대적인 값을 가짐. 아무런 설정을 하지 않았을 때 container가 가지는 값은 1024로서
이는 CPU할당에서 1의 비율을 뜻함. 

즉, 위의 예제와 같이 container의 --cpu-shares를 2048로 설정하면 CPU 작업 시 일반 container보다 2배 많은 시간을
할당 받음.

--cpu-shares 옵션으로 간단한 test를 해봅시다. 다음 명령어를 입력해 2개의 container를 생성.
cpu_1024 container는 cpu_512 container보다 할당시간이 2배 높음. 여기서 사용한 docker host는 
1개의 CPU를 가지고 있음. 각 container의 명령어로는 1번째 CPU에 부하를 주는 명령어(stress --cpu 1)로 설정됨.

```
# docker run -d --name cpu_1024 \
  --cpu-shares 1024 \
  alicek106/stress \
  stress --cpu 1
```

```
# docker run -d --name cpu_512 \
  --cpu-shares 512 \ 
  alicek106/stress \
  stress --cpu 1
```

host에서 아래의 명령어를 입력해 process의 자원 사용량을 확인하면 cpu_512 container는 cpu_1024 container보다
CPU 할당을 약 절반만큼 받은 것을 알 수 있음.

```
# ps aux | grep stress
```


```
(참고) 위에서 사용한 stress라는 명령어는 CPU와 memory에 과부하를 줘서 성능을 test함.
ubuntu container에서는 다음 명령어로 설치할 수 있음. alicek106/stress docker image는
미리 준비한 stress가 설치된 ubuntu image임.

# apt-get install stress

위에서 생성한 container는 -d 옵션으로 생성되어 백그라운드에서 계속 cpu에 부하를 주므로 test가 끝나면
반드시 container를 삭제해야함
```

**--cpuset-cpu**

host에 CPU가 여러개 있을 때, --cpuset-cpus 옵션을 지정해 container가 특정 cpu만 사용하도록 설정할 수 있음.
CPU 집중적인 작업이 필요하다면 여러개의 CPU를 사용하도록 설정해 작업을 적절하게 분배하는 것이 좋음
다음 예시는 container가 3번째 CPU만 사용하도록 설정

```
# docker run -d --name cpuset_2 \
  --cpuset-cpus=2 \
  alicek106/stress \
  stress --cpu 1
```
cpu별 사용량을 확인할 수 있는 대표적인 도구로 htop이 있으며 ubuntu에서는 apt-get install htop, 
CentOS에서는 yum -y install epel-release && yum -y install htop으로 설치할 수 있음.
htop 명령어로 CPU 사용량을 확인하면 3번째 CPU만 사용되는 것을 알 수 있음.


```
(참고) --cpuset-cpus="0.3"은 1, 4번째 cpu를, --cpuset-cpus="0-2"는 1,2,3번째 cpu를 사용하도록 설정
```

**--cpu-period, --cpu-quota**
container의 CFS(Completely Fair Scheduler) 주기는 기본적으로 100ms로 설정되지만 run 명령어의
option 중 --cpu-period와 --cpu-quota로 이 주기를 변경할 수 있음.

다음 명령어를 입력해 container를 생성

```
# docker run -d --name quato_1_4 \
  --cpu-period=100000 \
  --cpu-quota=25000 \
  alicek106/stress \
  stress --cpu 1
```

--cpu-period의 값은 기본적으로 100000이며 이는 100ms를 뜻함.
--cpu-quota는 --cpu-period에 설정된 시간 중 CPU scheduling이 얼마나 할당할 것인지를 설정함.
위 예시에서 100000(--cpu-priod) 중 25000(--cpu-quota)만큼을 할당해 cpu주기가 1/4로 줄었으므로
일반적인 container보다 CPU 설능이 1/4 정도로 감소함. 즉, container는 [--cpu-period 값]/[--cpu-quota 값]
만큼 CPU 시간을 할당받음.

성능 비교를 위해 다음 명령어로  container를 추가로 생성

```
# docker run -d --name quota_1_1 \
  --cpu-period=100000 \
  --cpu-quota=100000 \
  alicek106/stress \
  stress --cpu 1
```

htop 명령어나 ps aux | grep stress 로 cpu를 할당량을 확인하면 첫 번째 container가 1/4만큼 
cpu를 적게 사용하고 있음을 알 수 있음.

```
# ps aux | grep stress
```


**--cpus**

--cpus 옵션은 --cpu-period, --cpu-quota와 --cpu-share와 동일한 기능을 하지만 좀 더 직관적으로 CPU의
개수를 직접 지정한다는 점에서 다름.

예를 들어 --cpus 옵션에 0.5를 설정하면 --cpushare=512 또는 --cpu-period=100000 --cpu-quota=50000과 
동일하게 container의 CPU를 제한할 수 있음.

```
# docker run -d --name cpus_container \
  --cpus=0.5 \
  alicek106/stress \
  stress --cpu 1
```

마찬자기로 container의 사용량을 확인해보면 CPU의 약 50%르 점유하고 있음을 알 수 있음.

```
# ps aux | grep stress
```


#### 2.9.3 Block I/O 제한
Container를 생성할 때 아무런 option도 설정하지 않으면 container 내부에서 file을 읽고 쓰는 대역폭에
제한이 설정되지 않음.

하나의 container가 block 입출력을 과도하게 사용하지 않게 설정하려면 run 명령어에서 --device-write-bps,
--device-read-bps, --device-write-iops, --device-read-iops 옵션을 지정해 block 입출력을 제한할 수 있음.
단, Direct I/O의 경우에만 block 입출력이 제한되며, Buffered I/O는 제한되지 않음.

--device-write-bps, --device-read-bps는 각기 쓰고 읽는 작업의 초당 제한을 설정하며, kb, mb, gb 단위로
제한할 수 있음. 예를 들어, 다음 명령어로 container를 생성하면 초당 쓰기 작업의 최대치가 1MB로 제한 됨.

```
# docker run -it \
  --device-write-bps /dev/xvda:1mb \
  ubuntu:14.04
```

```
(참고) 여기서 사용되는 Block I/O를 제한하는 옵션은 [디바이스 이름]:[값] 형태로 설정해야함.
위 예시는 AWS의 EC2 instance에서 test했기 때문에 /dev/xvda라는 device를 사용하고 있으며
이 device에서 container의 저장 공간을 할당받고 있어 /dev/xvda:1mb로 설정함.
단, devicemapper를 storage driver를 사용하는 docker engine에서 loop device를 storage로 사용하고 있다면,
/dev/loop0:1mb와 같은 형식으로 써야함
```

위의 명령어로 생성된 container에서 쓰기 작업을 test해보면 속도가 초당 1MB로 제한되는 것을 확인할 수 있음.
다음 명령어는 10MB의 file을 Direct I/O를 통해 쓰기 작업을 수행함.

```
root@ :/# dd if=/dev/zero of=testd.out bs=1M count=10 oflag=direct
```

CPU의 자원할당에서 --cpu-share에 상대적인 값을 입력했던 것 처럼 --device-write-iops, 
--device-read-iops 에도 상대적인 값을 입력함. 예를 들어 --device-write-iops의 값이
2배 차이 나는 container로 쓰기 작업을 수행하면 수행 시간 또한 2배 가량 차이가 나는 것을
알 수 있음.

```
# docker run -it \
  --device-write-iops /dev/xvda:5 \
  ubuntu:14.04


root@:/# dd if=/dev/zero of=test.out bs=1M count=10 oflag=direct
```

```
# docker run -it \
  --device-write-iops /dev/xvda:10 \
  ubuntu:14.04
  
root@:/# dd if=/dev/zero of=test.out bs=1M count=10 oflag=direct
```

