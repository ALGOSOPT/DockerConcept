# Docker Machine
---
---

## 4.1 Docker machine을 사용하는 이유

Docker Container 환경에서 application을 개발해야 하는 개발자가 가장 먼저 마추지는
난관은 Docker 환경을 어디서 마련하지? 임.

조직에서 On-premise 환경의 server를 제공하거나 개인 서버를 소유하고 있거나
AWS와 같은 cloud service를 이용할 수 있다면 운이 좋은 편.

그러나 앞서 언급한 것조차 사용할 수 없다면 일반적으로 Virtual Box나 VMware 같은
가상 머신을 이용해 개인 PC에 linux를 설치 한 후 Docker로 개발함.


그러나 PaaS처럼 Multi host 환경의 Application을 개발한다면 virtual box같은 가상
환경을 사용하는 데도 한계가 있음. 일반적으로 적어도 3개 이상의 docker engine을 
사용해야 multi host환경에서 제대로 개발할 수 있는데, 가상 머신을
여러개 생성하는 것은 번거로울 뿐만 아니라 PC에도 많은 부담을 줌.


```
(참고) virtual box 같은 가상화 도구에서 가상 machine을 복제해서 사용하는
것은 바람직하지 않은. 게다가 일부 PaaS 솔우션은 MAC주소가 겹치면
오류가 발생하기도 함.
```

그림 4.1

