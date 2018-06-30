## Docker image

모든 container는 image를 기반으로 생성되므로 image를 다루는 방법은 docker 관리에서 빼놓을 수 없는 부분.
image의 이름을 구성하는 저장소, image 이름, tag를 잘 관리하는 것 뿐만 아니라 image가 어떻게
생성되고 삭제되는지, image의 구조는 어떻게 돼 있는지 등을 아는 것 또한 중요함. 

그림 2.32


데비안 운영체제에서 apt-get install을 실행하면 apt repository에서 package를 내려받고 
레드햇 운영체제에서 yum install을 실행하면 yum repository에서 package를 내려받듯이
docker는 기본적으로 Docker Hub라는 중앙 image 저장소에서 image를 내려 받음.


docker hub는 docker가 공식적으로 제공하고 있는 image 저장소로서, docker 계정을
가지고 있다면 누구든디 image를 올리고 내려받을 수 있기 때문에 다른 사람들에게 
image를 쉽게 공유할 수 있음.

docker create, docker run, docker pull의 명령어로 image를 내려받을 때 
docker 는 docker hub에서 해당 image를 검색한 뒤 내려 받음.

필요한 대부분의 image는 docker hub에서 공식적으로 제공하거나 다른 사람들이
docker hub에 이미 올려놓은 경우가 대부분이라서 application image를 직접
만들지 않아도 손쉽게 사용할 수 있다는 장점이 있음.

단, docker hub는 누구나 image를 올릴 수 있기 때문에 Official 라벨이 없는 image는 사용법을
찾을 수 없거나 제대로 동작하지 않을 수 있음.

또한 image 저장소를 다른 사람들에게 공개하지 않기 위해 Private 저장소를 사용하려면
비공개 저장소의 수에 따라 요금을 지불해야 함.

이를 해결하기 위해 Docker image 저장소를 직접 구축해 비공개를 사용할 수도 있으며
이후에 자세히 설명.


Docker Hub에서 어떤 image가 있는지 확인하기 위해 Docker Hub Site를 직접 접속해 찾아볼 수도 있지만
Docker Engine에서 Docker search 명령어를 사용할 수도 있음.

예를 들어, ubuntu라는 keyword와 관련된 image는 어떤 것이 있는지 검색해보자

```
# docker search ubuntu
```

보다시피 ubuntu image도 여러 종류가 있음을 알 수 있음. STARS는 해당 image가 docker 사용자로부터 얼마나
즐겨찾기(star) 되었는지를 나타냄.


---
### 2.3.1 Docker Image 생성

앞에서 처럼 docker search를 통해 검색한 image를 pull명령어로 내려받아 사용할 수도 잇지만 docker로
개발하는 많은 경우에는 container에 application을 위한 특정 개발 환경을 직접 구축한 뒤 사용자만의
image를 직접 생성해야할 것.
이를 위해 container안에서 작업ㄹ한 내용을 image로 만드는 방법을 먼저 설명하겠음.


다음 명령어를 입력해 image로 만들 container를 생성함.
container 내부에 first라는 이름의 file을 하나 생성해 기존의 image로 부터 변경사항을 만듦.
```
# docker run -i -t --name commit_test ubuntu:14.04
root@ :/# echo test_first! >> first
```

first라는 file을 만들어 ubuntu:14.04 이미지로부터 변경 사항을 만들었다면, container에서 host로 빠져나와
docker commit명령어를 입력해 container를 image로 만듦. docker commit 명령어의 형식은 아래와 같음.

먼저 commit의 옵션을 지정하고 commit할 container의 이름을 명시한 뒤 생성될 image의 이름을 입력함.

```
# docker commit [OPTIONS] CONTAINER [REPOSITORY[:TAG]]
```

다음 명령은 commit_test라는 container를 commit_test:first라는 이름의 image로 생성

```
# docker commit \
  -a "alicek106" -m "my first commit" \
  commit_test \
  commit_test:first
```

저장소 이름은 입력하지 않아도 상관없지만 image의 tag를 입력하지 않으면 자동으로 latest로 설정됨.
위 명령어에서는 image의 이름을 commit_test로, tag를 first로 설정함.

-a 옵션은 author를 뜻하며, image의 작성자를 나타내는 metadata를 image에 포함시킴.

commit_test:first 이미지의 작성자 data는 "alicek106"으로 설정될 것. -m 옵션은 commit message를 뜻하며
image에 포함될 부가 설명을 입력함.

최상위 디렉터리에 first라는 파일이 있는 docker image가 생성됨.
docker images 명령어로 image가 생성되었는지 확인함.

```
# docker images
```

commit_test라는 이름에 first라는 tag로 image가 생성됨. 
ubuntu:14.04 이미지의 크기와 비교했을 때 차이가 없는 것은 container의 변경사항에 해당하는 파일인
first라는 file이 수 bit에 불과하기 때문.



이번에는 commit_test:first 이미지로 새로운 image를 생성해보자.
commit_test:first 이미지로 container를 생성한 뒤 second라는 파일을 추가해
commit_test:second라는 image를 새롭게 생성함.
image를 생성하는 과정은 commit_test:first 이미지를 생성할 때와 같음.

```
# docker run -i -t --name commit_test2 commit_test:first
root@ :/# echo test_second~ >> second

# docker commit \
  -a "alicek106" -m "my second commit" \
  commit_test2 \
  commit_test:second
```

docker images 명령어로 두 번 째 이미지인 commit_test:second도 생성되었는지 확인함.

```
# docker images
```

commit_test라는 이름에 first라는 tag로 image가 생성됨. ubuntu:14.04 image의 크기와 비교했을 때
차이가 없는 것은 container의 변경사항에 해당하는 file인 first라는 파일이 수 bit에 불과하기 때문.


이번에는 commit_test:first이미지로 새로운  image를 생성해보자.  commit_test:first 이미지로
container를 생성한 뒤 second라는  file을 추가해 commit_test:second 라는 image를 새롭게
생성. image를 생성하는 과정은  commit_test:first image를 생성할 때와 같음.

```
# docker run -i -t --name commit_test2 commit_test:first

root@:/# ehco test_second >> second

#docker commit \
-a "alicek106" -m "my second commit" \
commit_test2 \
commit_test:second
```

docker images 명령어로 두 번째 image인 commit_test:second도 생성되었는지 확인
```
# docker images
```

---

### 2.3.2 image 구조의 이해



위와 같이 container를 image로 만드는 작업은 commit 명령어로 쉽게 구성할 수 있음.

그러나 image를 좀 더 효율적으로 다루기 위해 container가 어떻게 image로 만들어지며,
image의 구조는 어떻게 되어 있는지 알 필요가 있음.

다음 명령어를 입력해 image의 좀 더 자세한 정보를 확인해보자

```
# docker inspect ubuntu:14.04
# docker inspect commit_test:first
# docker inspect commit_test:second
```



```
(참고) inspect 명령어는 container 뿐만 아니라 network, volume, image등 모든 
docker 단위의 정보를 얻을 때 사용할 수 있음. 단, 이름이 중복될 경우 container에 대해
먼저 수행되므로 --type을 명시하는 것이 좋음.
```


docker inspect 명령어는 많은 정보를 출력하지만 주의 깊게 살펴볼 항목은 가장 아랫부분에 있는
Layers 항목.

ubuntu:14.04, commit_test:first, commit_test:second 에 대한 각 Layers 항목은 다음과 같다.
16진수 hash 값 중 뒷부분을 생략했으며, ID값은 아래 image와 다를 수 있음.

다음 그림은 각 image에 대한 inspect명령어의 출력 결과 중 Layers 항목만 나타낸 것.

그림 2.33

이를 좀 더 보기 쉽게 나타내면,

그림 2.34


docker images에서 위 3개의 image 크기가 각각 188MB라구 출력되도 188MB 크기의 image가
3개 존재하는 것은 아님.

image를 commit할 때 container에서 변경된 사항만 새로운 layer로 저장하고,
그 layer를 포함해 새로운 image를 생성하기 때문에 전체 image의 실제 크기는
188MB + first file의 크기 + second file의 크기가 됨.

first file은  commit_test:first image를 생성할 때 사용했던 
container에서 변경된 사항이고,
second file은 commit_test:second image를 생서할 때
container에서 변경된 사항.



그림 2.35

즉, 위 그림과 같이 ubuntu:!4.04 image를 기반으로 생성했던 첫 번쨰 container commit_test에서
변경된 사항인  first file이 sha256:cc1bcbb270d layer가 됨.

이는 commit_test:second image를 생성할 때도 같음.


```
(참고) 이러한 image layer의 구조는 docker history 명령을 통해 좀 더 쉽게 확인할 수 있음.
이 명령어는image가 어떤 layer로 생성되었는지를 출력


# docker history commit_test:second

```

이번에는 생성한 image를 삭제해보자. docker rmi 명령어를 사용하면 image를 삭제할 수 있음.

```
# docker rmi commit_test:first
```

그러나 위 명령어는 다음과 같은 error를 출력함.



image를 사용 중인 container가 존재하므로 해당 image를 삭제할 수 없다는 내용. container를 삭제할 수 없다는 내용.
container를 삭제할 때 사용했던 docker rm -f  [container 이름] 처럼 -f 옵션을 추가해 image를 강제로 삭제할수도 
있지만 이는 image layer file을 실제로 삭제하지 않고 image이름만 삭제하기 때문에 의미가 없음.

따라서 다음 명령어와 같이 container를 삭제한 뒤 image를 삭제하게 함.

```
# docker stop commit_test %% docker rm commit_test2

# docker rmi commit_test:first 
```


```
(참고) Container가 사용 중인 image를 docker rmi -f 로 강제로 삭제하면 image의 이름이 
<none>으로 변경되며, 이러한  image들을  dangling image라고 함.
dagling image는 docker images -f dangling=true 명령어로 사용해 별도로 확인할 수 있음

# docker images -f dangling=true


사용 중이지 않은 dangling image는 docker image prune 명령어로 한꺼번에 삭제할 수 있음.

# docker image prune
```

commit_test:first image를 삭제했다고 해서 실제로 해당 image의 layer file이 삭제되지는 않음.
commit_test:first image를 기반으로 하는 하위 image인 commit_test:second가 존재하기 때문.

따라서 실제 image file을 삭제하지 않고 layer에 부여된 이름만 삭제함.
rmi 명령어의 출력 결과인 Untagged: ... 는 image에 부여된 이름만 삭제한다는 것을 뜻함.

그림 2.36


이번에는 commit_test:second 이미지를 삭제해보자. commit_test:second image를 사용하고 있는
container가 없으니 바로 삭제할 수 있음.

```
# docker rmi commit_test:second
```

"Deleted:"라는 출력 결과는 image layer가 실제로 삭제되었음을 뜻함. image의 이름인 
commit_test:second 를 가리키는 ID와 실제 layer를 가리키는  ID가 삭제됐는데
실제 layer file은 sha256:0b168 ... 라는 ID에 의해 참조됨.
즉, 삭제되는 image의 부모 image가 존재하지 않아야만 해당 image의 file이 실제로
삭제됨.


그렇지만  commit_test:second image를 삭제했다고 해서 ubuntu:14.04 image도 함께
삭제되는 것은 아님. 앞에서는 second, first file이 존재하는 layer만 삭제 되었는데,
commit_test:first image tag는 이미 삭제되어 존재하지 않지만 ubuntu image를
가리키는 ubuntu:14.04 image tag는 아직 존재하기 때문임.

따라서 ubuntu의 image layer는 commit_test:second image를 삭제할 때 함꼐 삭제
되는 layer의 범위에 포함되지 않음.

### 2.3.3 Image 추출

docker image를 별도로 저장하거나 옮기는 등 필요에 따라 image를 단일 binary file로 저장해야
할 때가 있음. docker save 명령어를 사용하면 container의 command, image 이름과 tag
등 image의 모든 meta data를 포함해 하나의 file로 추출할 수 있음. -o option에는 추출될
파일명을 입력함

```
# docker save -o ubuntu_14_04.tar ubuntu:14.04
```

추출된 image는 load 명령어로 docker에 다시 로드할 수 있음.
save 명령어로 추출된 image는 image의 모든 meta data를 포함하기 때문에 load명령어로 image
를 로드하면 이전의 image와 완전히 동일한 image가 docker engine에 생성됨.

```
# docker load -i ubuntu_14_04.tar
```

save, load 명령어와 유사하게 사용할 수 있는 명령어로 export, import가 있음.
docker commit 명령어로 container를 image로 만들면 container에서 변경된 사항 뿐만 아니라
container가 생성될 때 설정된 detached mode, container command의 설정 등도 image에 함께 저장됨

그러나 export 명령어는 container의 file system을 tar file로 추출하며 container 및 image에
대한 설정 정보를 저장하지 않음.

export와 import는 다음 예제처럼 사용할 수 있음. export 명령어는 mycontainer라는 container의
file system을 rootFS.tar 파일로 추출하고, 이  file을 import명령어로 myimage:0.0 이라는 image로 다시 저장함.

```
# docker export -o rootFS.tar mycontainer
# docker import rootFS.tar myimage:0.0

```

그러나 image를 단일 파일로 저장하는 것은 효율적인 방법이 아님. 추출된 image는 layer 구조의 파일이 아닌
단일 file이기 때문에 여러 버전의 image를 추출하면 image 용량을 각기 차지하게 됨.
예를 들어, ubuntu:14.04 image와 commit_test:first 라는 두 개의 image를 추출하면
각각 188MB의 파일이 생성되어 총 376MB를 차지하게 될 것임.


---

### 2.3.4 image 배포

이미지를 생성했다면 이를 다른 docker engine에 배포할 방법이 필요함. save나 export와 같은 방법으로
image를 단일 파일로 추출해서 배포할 수도 있지만 image file의 크기가 너무 크거나 docker engine의
수가 많다면 image를 파일로 배포하기 어려움. 또한 docker image 구조인 layer 형태를 이용하지 않으므로
매우 비효율적임.


이를 해결하는 첫 번째 방법은 docker 에서 공식적으로 제공하는 docker hub image 저장소를 사용하는 것.
docker hub는 docker image를 저자하기 위한 cloud service 라고 생각하면 이해하기 쉬움.

사용자는 단지 image를 올리고(docker push) 내려받기(docker pull)만 하면되므로 매우 간단하게 사용할 
수는 있음. 단, 결제를 하지 않으면 Private 저장소의 수에 제한이 있는 것이 단점.
Public 저장소는 무료로 사용할 수 있으므로 만든 image를 다른 사용자에게도 공개해도 상관없다면
docker hub를 사용하는 것도 좋은 선택.

두 번째 방법은 docker private registry를 사용하는 것으로 사용자가 직접 image 저장소를 만들 수 있음.

그러나 사용자가 직접 image 저장소 및 사용되는 server, 저장 공간 등을 관리해야 하므로 docker hub보다는
사용법이 까다로움. 그러나 회사의 내부망과 같은 곳에서 docker image를 배포해야 한다면 docker private
registry가 더 좋은 방안이 될 수있음


#### 2.3.4.1 docker hub registry

docker hub site에서 docker search 명령어를 입력했을 떄와 같이 image를 검색할 수 있으며,
image 저장소를 클릭하면 image에 대한 자세한 설명을 볼 수 있음

그림 2.37
그림 2.38

docker hub에 여러분의 저장소를 생성해보자. 저장소를 생성하려며 login이 필요하므로 
Sign up하시고요,,


**image저장소 생성 **

가입한 뒤 로그인을 하면 아래와 같은 화면을 볼 수 있음
Create Repository 항목을 클릭해 저장소를 생성

그림 2.39

저장소를 생성하기 전 저장소에 대한 정보를 입력해야 함.
저장소에 저장될 image의 이름, image의 간단한 설명과 전체 설명을 입력함.

그림 2.40

마지막으로 저장소의 Visibility를 설정해 저장소를 다른 사람에게 공개할지 여부를 결정.
Public 으로 설정하면 다른 사용자가 docker search와 pull 명령어로 image를 사용할 수 있지만
private로 설정하면 저장소의 접근 권한이 있는 계쩡으로 로그인 해야 저장소를 사용할 수 있음.

기본적으로 비공개 저장소는 1개만 무료이고 그 이상 사용하려면 매달 일정 금액을 결제해야 함.

여기서는 공재 저장소를 생성하겠다. create 버튼을 눌러 저장소를 생성

그림 2.41

이 저장소의 이름은 601keccila/myimagename 임. 회원 가입을 할 떄 입력한 이름이 저장소 이름이 되며,
위 예시에서는 계정 이름이 601kecila이기 때문에 저장소 이름 또한 601keclia로 설정됨,
myimagename은 실제로 이 저장소에 저장될 image의 이름.


**저장소 에 이미지 올리기**

방금 생성한 저장소에 image를 올려보자. docker에서 다음 명령어를 입력해 저장소에 올릴 image를 생성

```
# docker run -i -t --name commit_container1 ubuntu:14.04
root@:/# echo my first push >> test

# docker commit commit_container1 myimagename:0.0
```



ubuntu:14.04 이미지에 test라는 파일을 생성해 변경 사항을 만든 뒤 myimagename:0.0 이라는 image로
commit함. 그러나 이 이름으로는 image를 저장소에 올릴 수 없음. 2.2.1절 "도커 이미지"에서 설명했듯,
이미지의 접두어는 image가 저장되는 저장소 이름으로 설정함.

즉, 특정 이름의 저장소에 image를 올리기 위해서는 저장소 이름을 앞에 접두어로 추가해야함.

docker tag 명령어를 사용하면 image의 이름을 추가할 수있음. tag 명령어의 형식은 'docker tag [기존의 이미지 이름]
[새롭게 생성될 이미지 이름] 으로서, 다음 예시는 myimagename:0.0 이미지에 601kecila/myimagename:0.0 
이라는 이름을 추가함.

```
# docker tag myimagename:0.0 601kecila/myimagename:0.0
```

```
(참고) tag 명령어로 image의 이름을 변경했다고 해서 기존의 이름이 사라지는 것은 아니며
같은 image를 가리키는 새로운 이름을 추가할 뿐.

# docker images
```

이미지 이름을 변경하고 나면 다음 명령어를 입력해 docker hub server에 로그인함. 
로그인 하지 않으면 생성한 저장소에 image를 올릴 수 있는 권한을 가질 수 없음.

참고로 회원 가입을 할 때 사용한 email과 비밀번호를 입력함.

```
# docker login
```

```
(참고) docker engine에서 로그인한 정보는 /계정 명/.docker/config.json 파일에 저장됨.
```


login한 뒤 push명령어를 입력해 image를 저장소에 올려보자. 명령어의 출력 결과를 보면 하나의
layer에만 저장소로 전송되고 나머지 layer는 ubuntu:14.04 image에서 생성되어 docker hub의
ubuntu 이미지 저장소에 이미 존재하므로 전송되지 않음.

```
# docker push 601keclia/myimagename:0.0
```

docker hub의 저장소에 실제로 이미지가 올려졌는지 확인.
Tags 항목에서 image를 확인할 수 있음.

그림 2.42

docker에서 이 이미지를 내려받으려면 별도로 로그인하지 않고 다음 명령어를 입력하면 된다

```
# docker pull 601kecila/myimagename:0.0
```

