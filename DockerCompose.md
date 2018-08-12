# 5. Docker Compose

---
## 5.1 Docker Compose를 사용하는 이유

여러 개의 Container가 하나의 Application으로 동작할 때 이를 test하려면
각 Container를 하나씩 생성해야함. 예를 들어 Web을 Test하려면 웹 서버 container와 
database container를 생성해야함. 다음과 같은 run 명령어를 생각해보자
```
# docker run --name mysql -d alicek106/composetest:mysql mysqld

# docker run -d -p 80:80 \
  --link mysql:db --name web \
  alicek106/composetest:web apachectl -DFOREGROUND
```

위 예제의 run명령에 사용되는 image는 test용으로서 명령어를 실행하면 아파치
웹 서버 container와 mysql container를 생성함. 이 처럼 여러 개의 container로 구성된
앱을 구축하기 위해 run 명령어를 여러 번 사용할 수 있지만 각 container가 
제대로 동작하는지 확인하는 test 단계에세는 이렇게 하기가 번거로움.

매번 run 명령어에 옵션을 설정해 CLI(Command Link Interface)로 Container를 생성하기 보다는
여러 개의 container를 하나의 service로 정의해 container 묶음으로 관리할 수 있다면 좀 더 편리
할 것임. 이를 위해 Docker Compose는 Container를 이용한 service의 개발과 CI를 위해 여러 개의
Container를 하나의 Project로서 다룰 수 있는 작업 환경을 제공함.

Docker Compose는 여러 개의 Container 옵션과 환경을 정의한 파일을 읽어 Container를 순차적으로
생성하는 방식으로 동작함. Docker Compose 의 설정 파일은 run 명령어의 option을 그대로 사용할 수 있으며
각 container의 의존성, 네트워크, 볼륨 등을 함께 정의할 수 있음. 

또한 swarm mode의 service와 유사하게 설정 파일에 정의된 service의 container 수를 유동적으로 조절할
수있으며 container의 service discovery도 자동으로 이뤄짐. 물론 이러한 기능이 필요하지 않은
소규모 container 개발 환경에서는 Docker Engine의 run 명령어로 Container를 생성하는 것이 더
편히할 수도 있음. 
그렇지만 Container의 수가 많아지고 정의해야할 옵션이 많아 진다면 Docker Compose를 사용하는 것이 좋음

---

## 5.2 Docker Compose 설치

linux에서는 다음 명령어로 Docker Compose를 설치 할 수 있으며, Docker Compose 의 GitHub 저장소에서 
직접 내려 받아 설치할 수 있음.
이번 예제는 1.1버전을 기준으로 설명함

```
# curl -L https://github.com/docker/compose/releases/download/1.11.0/\
  docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose
  
# chmod +x /usr/local/bin/docker-compose
```

윈도우와 맥 OS X에서는 docker toolbox나 Docker for MAC을 설치하면 Docker Engine과
함께 Docker Compose도 설치됨.

설치가 정상적으로 완료했는지 확인하기 위해 Docker Compose의 버전을 확인함

```
# docker-compose -v
```


---
## 5.3 Docker Compose 사용

---

### 5.3.1 Docker Compose 기본사용법

Docker Compose는 Container의 설정이 정의된 YAML 파일을 읽어 Docker Engine을 통해
Container를 생성함. 따라서 Docker Compose를 사용하려면 가장 먼저 YAML 파일을 작성해야함.



![그림5.1](https://github.com/ALGOSOPT/DockerConcept/blob/master/picture/5.1.jpg)

#### 5.3.1.1 docker-compose.yml 작성과 활용

Docker Compose의 사용법을 알아보기 위해 다음과 같은 run 명령어를 docker-compose.yml 파일로
변환해 Container를 생성하고 실행해봅시다.

```
# docker run -d --name mysql \
  alicek106/composetest:mysql \
  mysqld
  
# docker run -d -p 80:80 \
--link mysql:db --new web \
alicek106/composetest:web \
apachectl -DFOREGROUND
```

```
예제 5.1 docker-compose.yml 파일 작성

version: '3.0'
services:
  web:
    image: alicek106/composetest:web
    ports:
      - "80:80"
    links:
      -mysql:db
    command: apachectl -DFOREGROUND
  mysql:
    image: alicek106/composetest:mysql
    command: mysqld
```

```
YAML 파일에서 들여쓰기 할 때 탭은 Docker Compose가 인식하지 못하므로  2개의 space를 사용해라
```

어떠한 설정도 하지 않으면 Docker Compose는 현재 directory의 docker-compose.yml 파일을 읽어
local 의 docker engine에게 container 생성을 요청함.
예제 5.1을 docker-compose.yml 파일로 저장한 후 이 파일을 저장한 directory에서
docker-compose up -d 명령어로 container를 생성하면 다음과 같은 출력 결과를 확인할 수 있음.

이 예제에서는 /home/ubuntu 디렉터리에 docker-compose.yml 파일을 저장했음.
project의 이름을 명시하지 않으면 container들은 현재 directory의 이름으로 시작하는 이름을 갖게 됨.

```
# docker-compose up -d

```

사용된 docker-compose.yml 파일에 대한 설명은 다음과 같음.

```
version: '3.-'
sevices:
  web:
    image: alicek..

  mysql:
    image: ....
```

- version : YAML 파일 포멧의 버전을 나타냄
- services : 생성될 container들을 묶어 놓은 단위. 서비스 항목 아래에는 각 container에 적용될 
            옵션을 지정함.
- web, mysql : 생성될 서비스의 이름. 이 항목아래에는 container가 생성될 때 필요한 option을 지정할 
              수 있음. 
              
              
위 예제에서 쓰인 image는 docker compose test용 image로 서버의 /main.php 경로를 통해
간단한 로그인 page를 사용할 수 있음. 생성된 container는 docker ps 명령어뿐 아니라 docker-compose ps 명령어
로도 확인할 수 있음.
```
# docker ps

# docker-compose ps
```


#### 5.3.1.2 Docker Compose의 project, service, container

예제 5.1의 YAML 파일에서 web서비스와 mysql 서비스를 정의했고, service별로 container가 
1개씩 생성되었으며 각 container의 이름은 ubuntu_web_1, ubuntu_mysql_1 이었음.
docker compose는 container를 project 및 service 단위로 구분하므로 
container의 이름은 일반적으로 다음과 같은 형식으로 정해짐

```
[프로젝트 이름]_[서비스 이름]_[서비스 내에서 컨테이너의 번호]
```

위 예제에서 생성한 프로젝트의 이름은 ubuntu이고 각 서비스의 이름은  mysql, web임. 위에서
docker-compose up -d 를 실행했을 때 project의 이름을 별도로 입력하지 않았지만
docker compose는 기본적으로 docker-compose.yml 파일이 위치한 directory의 이름을
project 이름으로 사용함. project의 이름은 docker-compose.yml 파일이 저장된 directory의 이름에
따를 것

![그림5.2](https://github.com/ALGOSOPT/DockerConcept/blob/master/picture/5.2.jpg)

그림 5.2와 같이 하나의 project는 여러개의 service로 구성되고 각 서비스는 여러개의 컨테이너로
구성됨. swarm 모드에서의 서비스와 마찬가지로 하나의 서비스에는 여러개의 container가 존재할 수 있으므로
차례대로 증가하는 container의 번호를 붙여 서비스 내의 container를 구별함
위 예제에서는 ubuntu_mysql_1만 있지만 docker-compose scale 명령어로 ubuntu_mysql_2,
ubuntu_mysql_3을 생성할 수 있음.

```
# docker-compose scale mysql=2
```

```
(참고) docker-compose up 명령어의 끝에 service의 이름을 입력해 docker-compose.yml파일에 
명시된 특정 service만 생성할 수 있음. 다음 예제에서는 mysql 서비스의 container만 생성함.

# docker-compose up -d mysql

docker-compose run 명령어로 container를 생성할 수도 있음. 이때는 interactive shell을 
사용할 수 있음.

# docker-compose run web /bin/bash
```

docker-compose scale 명령어를 사용한 뒤 docker-compose ps 명령어로 container의 목록을
확인하면 mysql service의 container 수가 늘어난 것을 알 수 있음.
이처럼 서비스의 container 수를 늘리거나 줄여서 같은 container의 수를 일정하게 유지할 수 있음.

```
# docker-compose ps
```

생성된 project는 docker-compose down 명령어로 삭제할 수 있음. 
project를 삭제하면 sevice의 container 또한 전부 정지된 뒤 삭제됨.
```
# docker-compose down
```

docker compose는 기본적으로 현재 directory의 이름으로 된 project를 제어함.
예를 들어 /home/ubuntu directory에 docker-compose.yml 파일이 있고,
docker-compose down 명령어를 사용하면 ubuntu라는 이름을 가진 project를 삭제함.
그러나 docker-compose의 -p 옵션에 프로젝트의 일므을 사용해 제어할 프로젝트의 이름을 명시할 수 있음.
즉 -p 옵션을 사용하면 하나의 docker-compose.yml 파일로 서로 이름이 다른 여러 개의 
프로젝트를 생성하고 제어할 수 있음.

```
# docker-compose -p myproject up -d

# docker-compose -p myproject ps

# docker-compose -p myproject down
```

---
### 5.3.2 Docker Compose 활용

Docker Compose를 실제 개발 환경에서 사용하려면 이제까지보다 더 많은 옵션과 명령어가 필요함
또한 YAML 파일을 작성하는 데 능숙해져야 기존 Container의 설정을 손쉽게 Docker Compose
환경으로 옮기고 수정할 수 있음. 이번에는 Docker Compose를 다루는 자세한 방법과 YAML 파일에서
자주 쓰이는 항목들을 알아 보겠음.

#### 5.3.2.1 YAML 파일 작성
Docker Compose를 사용하려면 Container 설정을 저장해 놓은 YAML 파일이 필요함.

그러므로 기존에 사용하던 run 명령어를 YAML 파일로 변환하는 것이 Docker Compose 사용법의 대부분.

YAML 파일은 크게 버전 정의, 서비스 정의, 볼륨정의, 네트워크 정의의 4가지 항목으로 구성됨.
이 가운데 많이 사용하는 것은 sevice 정의이며, Volume 정의와 network 정의는 service로 생성된
Container에 선택적으로 사용됨. 각 항목의 하위 항목을 정의하려면 2개의 space로 들려쓰기해서
상위 항목과 구분함.

```
(참고) Docker Compose는 기본적으로 현재 Directory 또는 상위 Directory에서
docker-compose.yml이라는 이름의 YAML 파일을 찾아서 Container를 생성함.
그러나 docker-compose 명령어의 -f 옵션을 사용하면 yml 파일의 위치와 이름을 지정
할 수 있음.

# docker-compse \
-f /home/alicek106/my_compose_file.yml \
up -d

-f 옵션은 프로젝트의 이름을 지정하는 -p 옵션과 함께 사용할 수 있음. 특정 YAML 파일에서 
생성된 여러 개의 project를 제어하려면 해당 YAML 파일이 현재 directory에 존재하거나,
-f 옵션으로 경로를 지정해야함. 즉, 먼저 -f 옵션을 통해 YAML 파일을 먼저 지정해 파일을
읽은 뒤 -p 옵션으로 project의 이름을 명시하는 것.
```

  (1) 버전 정의
  YAML 파일 포멧에는 버전 1, 2, 2.1, 3이 있지만 이번 장에서는 Docker Compose 
  버전 1.10 에서 사용할 수 있는 버전 3을 기준으로 함. 버전3은 docker swarm mode와 
  호환되는 version이므로 가능하면 최신 version의 docker compose를 사용하는 것이
  좋음.
  
  version 항목은 일반적으로 YAML 파일의 맨 윗부분에 명시힘
  
  ```
  version: '3.0'
  ```
  (2) service 정의
  service는 docker compose로 생성할 container option을 정의함. 이 항목에 쓰인 각 service는 
  container로 구현되며, 하나의 project로서 docker compose에 의해 관리됨. service의 이름은
  sevices의 하위 항목으로 정의하고, container의 option은 service 이름의 하위 항목에 정의함
  
  ```
  services:
    my_container_1:
      images: ...
    my_container_2:
      images: ...
  ```
  
  이번 장에서는 service 항목이 가질 수 있는 주요 container 옵션만 설명함. 설명하지
  않은 서비스 항목에 대해서는 공식 문서를 참고하기 바람.
  
  * image : service의 container를 생성할 때 쓰일 image의 이름을 설정함. image 이름 format은 
  docker run과 같음. 만일 image가 docker에 존재하지 않으면 저장소에서 자동으로 내려 받음.
  
  ```
  services:
    my_container_1:
      image: alicek106/composetest:mysql
  ```
  
  * links : docker run 명령어의 --link와 같으며, 다른 서비스에 서비스명만으로 접근할 수 있도록 
  설정함. [SERVICE:ALIAS]의 형식을 사용하면 service에 별칭으도로 접근할 수 있음.
  ```
  services:
    web:
      links:
        -db
        -db:database
        -redis
  ```
  
  * environment : docker run 명령어의 --env, -e 옵션과 동일. service의 container 내부에서 사용할
  환경변수를 지정하며, Dictionary나 배열 형태로 사용할 수 있음.
  ```
  services:
    web:
      environment:
        - MYSQL_ROOT_PASSWORD=mypasswod
        - MYSQL_DATABASE_NAME=mydb
          또는
      environment:
        MYSQL_ROOT_PASSWORD: mypassword
        MYSQL_DATABASE_NAME: mydb
  ```
  
  * command: container가 실행될 때 수행할 명령어를 설정하며 docker run 명령어의 마지막에 붙는 cmd와 같음
  Dockerfile의 RUN과 같은 배열 형태로도 사용할 수 있음.
  ```
  services:
    web:
      image: alicek106/composetest:web
      command : apachtcl -DFOREGROUND
      또는
    web:
      image: alicek106/composetst:web
      command : [apachectl, -DFOREGROUND]
  ```
  
  * depends_on : 특정 container에 대한 의존 관계를 나타내며, 이 항목에 명시된 container가 먼저 생성되고
  실행됨. 다음 예제에서는 web container보다 mysql container가 먼저 생성됨.
  links 도 depends_on 과 같은 container의 생성 순서와 실행 순서를 정의하지만 depends_on은 service 이름으로만
  접근할 수 있다는 점이 다름.
  ```
  services:
    web:
      image: alicek106/composetest:web
      depends_on
        -mysql
      
    mysql:
      image: alicek106/composetest:mysql
  ```
  
  특정 service의 container만 생성하되 의존성이 없는 container를 생성하려면 --no-deps 옵션을 사용함.
  ```
  # docker-compose up --no-deps web
  ```
  
  * ports : docker run 명령어의 -p와 같으며 service의 container를 개방할 port를 설정함. 그러나 
  단일 host 환경에서 80:80과 같이 host의 특정 port를 serivce의 container에 연결하면
  docker-compose scale 명령어로 서비스의 container 수를 늘릴 수 없음.
  ```
  services:
    web:
      image: alicek106/composetest:web
        ports:
          - "8080"
          - "8081-8085"
          - "80:80"
          ...
  ```
  
  * build : build 항목에 정의된 dockerfile에서 image를 빌드해 service의 container를 생성하도록
  설정함. 다름 예제는 ./composetest directory에 저장된 dockerfile로 image를 빌드해 container를
  생성함. 새롭게 빌드될 image의 이름은 image 항목에 정의된 이름인 alicek106/composetest:web 이 됨.
  ```
  services:
    web:
      build: ./composetest
      image: alicek106/composetest:web
  ```
  
  또한 build 항목에서는 dockerfile에 사용될 context나 dockerfile의 이름. dockerfile에서 사용될
  인자 값을 설정할 수 있음. 다음과 같이 image항목을 설정하지 않으면 image의 이름은
  [프로젝트 이름]:[서비스 이름]이 됨.
  
  ```
  services:
    web:
      build: ./composetest
      context: ./composetest
      dockerfile: myDockerfile
      args:
        HOST_NAME: web
        HOST_CONFIG: self_config
  ```
  
  ```
  build 항목을 YAML 파일에 정의해 프로젝트를 생성하고 난 뒤 dockefile을 변경하고 다시
  프로젝트를 생성해도 image를 새로 빌드하지 않음. docker-compose up -d에 --build 옵션을
  추가하거나 docker-compose build 명령어를 사용해 dockerfile이 변경되어도 container를
  생성할 때 마다 build하도록 설정할 수 있음.
  
  # docker-compose up -d --build
  # docker-compose build [yml 파일에서 빌드할 서비스 이름]
  ```
  
  
  (3) network 정의
  * driver : docker compose는 생성된 container를 위해 기본적으로 bridge type의 network를
  생성함. 그러나 YAML 파일에서 driver 항목을 정의해 service의 container가 bridge network가 아닌
  다른 network를 사용하도록 설정할 수 있음. 특정 driver에 필요한 옵션은 하위 항목인 driver_ops로
  전달할 수 있음.
  
  다음 예제는 docker-compose up -d 명령어도 된 container를 생성할 때 mynetwrok라는 
  overlay 타입의 network도 함께 생성하고, myservice 서비스의 network가 myservice 네트워크를
  사용하도록 설정함. 단, overlay 타입의 네트워크는 swarm mode나 주키퍼를 사용하는 환경이어야만
  생성 할 수 있음.
  
  ```
  version: '3.0'
  services:
    myservice:
      image: nginx
      networks:
        - mynetwork
  networks:
    mynetwork:
      driver: overlay
      driver_opts:
        subnet: "255.255.255.0"
        IPAdress: "10.0.0.2"
  ```
  
  * ipam : IPAM(IP Address Manager)를 위해 사용할 수 있는 옵션으로서 subnet, ip 범위등을
  설정할 수 있음. driver 항목에는 IPAM을 지원하는 driver의 이름을 립혁함.
  
  ```
  services:
  ...
  
  networks:
    ipam:
      driver: mydriver
      config:
        subnet: 172.20.0.0/16
        ip_range : 172.20.5.0/24
        gateway : 172.20.5.1
  ```
  
  * external: YAML 파일을 통해 project를 생성할 때마다 network를 생성하는 것이 아닌, 
  기존의 network를 사용하도록 설정함. 이를 설정하려면 사용하려는 외부 네트워크의 이름을
  하위 항목으로 입력한 뒤 external의 값을 true로 설정함. external 옵션은 준비된 network
  를 사용하므로 driver, driver_ops, ipam 옵션과 함께 사용할 수 없음.
  
  다음 예제는 serivce의 container가 기존의 alicek106_network 라는 이름의 network를 사용
  하도록 설정함.
  
  
  ```
  services:
    web:
      image: alicek106/composetest:web
      networks:
        - alicek106_network
  networks:
    alicek106_network
      external: true
  ```
  
  (4) Volume 정의
  * driver : Volume을 생성할 때 사용될 driver를 설정함. 어떠한 설정도 하지 않으면
  local로 설정되며 사용하는 driver에 따라 변경해야함. driver를 사용하기 위한
  추가 옵션은 하위 항목인 driver_opts를 통해 인자로 설정할 수 있음.
  ```
  version: '3.0'
  services
  ...
  
  volumes:
    drivers: flocker
      driver_opts:
        opt: "1"
        opt2 : 2
  ```
  
  * external : docker compose는 YAML 파일에서 volume, volumes-from 옵션 등을 사용하면
  project 마다 volume을 생성함. 이때 external 옵션을 설정하면 volume을 프로젝트를
  생성할 때마다 매번 생성하지 않고 기존 volume을 사용하도록 설정함.
  다음 예제에서 myvolume이라는 이름의 외부 volume을 web service의 container에 mount함
  
  ```
  services:
    web:
      image: alicek106/composetest:web
      volumes:
        - myvolume:/var/www/html
  volumes:
    myvolume:
      external: true
  ```
  
  (5) YAML 파일 검증하기
  YAML 파일을 작성할 때 오타나 파일 포맷이 적절한지 등을 검사하려면 docker-compose config 명령어를
  사용함. 기본적으로 현재 directory의 docker-compose.yml 파일을 검사하지만
  docker-compose -f (yml 파일 경로) config와 같이 검사할 파일의 경로를 설정할 수 잇음.
  
  


---

#### 5.3.2.2 Docker Compose Network

YAML 파일에 network 항목을 정의하지 않으면 docker compose는 project 별로 
bridge type의 network를 생성함. 생성된 network 이름은 {프로젝트 이름}_defualt
로 설정되며, docker-compose up 명령어오 생성되고 docker-compose down 명령어로
삭제됨.

```
# docker-compose up -d

# docker-compose down
```

docker-compose up 명령어뿐 아니라 docker-compose scale 명령어로 생성되는 
container 전부가 이 bridge 타입의 network를 사용함. 서비스 내의 container는 
--net-alias가 service의 이름을 갖도록 자동으로 설정되므로 이 network가 
속한 container는 service의 이름으로 service 내의 container에 접근할 수 있음.

![그림5.3](https://github.com/ALGOSOPT/DockerConcept/blob/master/picture/5.3.jpg)

예를 들어 web 서비스와 mysql 서비스가 각기 존재할 때 web 서비스의 container가 mysql
라는 host 이름으로 접근하면 mysql service의 container 중 하나의 IP로 변환(resolve)
되며, container가 여러 개 존재할 경우 round-lobin으로 연결을 분산함.

따라서 docker-dompose scale 명령어로 container의 수를 늘려도 해당 서비스의 이름으로
서비스의 모든 container에 접근할 수 있음. service의 이름으로 container에 접근하는 방식은
swarm mode의 service와 유사하지만 작동 원리는 다름.
