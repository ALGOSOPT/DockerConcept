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





