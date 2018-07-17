#5. Docker Compose

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



그림 5.1

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

그림 5.2

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
