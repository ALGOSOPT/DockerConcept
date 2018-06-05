# Docker Engine


---

## 1. Docker image와 Container

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


