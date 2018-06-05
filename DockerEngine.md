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

