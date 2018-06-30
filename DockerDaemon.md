# Docker daemon

지금까지 docker를 사용하는 방법을 설명. 사장 먼저 알아야할 container부터 시작해서 container의 밑바탕이 되는 image, 그리고
그 image를 생성할 수 있는 docker file을 알아봄. 그렇자면 이제는 docker 자체를 다뤄볼 차례. 
docker 자체에 사용할 수 있는 여러 옵션을 익히면 container와 image를 좀 더 쉽게 사용할 수 있을 뿐더러
docker를 이용한 개발이 더욱 수월해짐


---

### Docker의 구조

docker를 사용할 때 docker라는 명령어를 맨 앞에 붙여서 사용해왔음. 그렇다면 docker는 실제로 어디에 있는걸까?
which 명령어로 docker 명령어의 위치를 확인 할 수 있음.

```
# which docker
```
보다 시피 docker 명령어는 /usr/bin/docker에 위치한 file로 docker  명령어를 실행되고 사용하고 있음.
이번에는 실행 중인 docker process를 확인해보자

```
# ps aux | grep docker
```

```
(참고) 리눅스의 which 명령어는 명령어의 파일이 위치한 경로를 출력
ps aux 명령어는 실행 중인 프로세스의 목록을 출력
```

container나 image를 다루는 명령어는 /usr/bin/docker에서 실행되지만 docker engine의 process는
/usr/bin/dockerd 파일로 실행되고 있음. 이는 docker 명령어가 실제  docker engine이 아닌
클라이언트로서의 docker이기 때문

docker의 구조는 크게 2가지고 나뉨. 하나는 client로서의 docker이고, 다른 하나는 server로서의 docker
실제로 container를 생성하고 실행하며 image를 관리하는 주체는 docker server이고,
이는 dockerd process로서 동작함. docker engine은 외부에서 api를 입력받아 
docker engine의 기능을 수행하는데, docker 프로세스가 실행되어 server로서 입력을 받을 준비가 된 상태를
**docker daemon** 이라고 함.


