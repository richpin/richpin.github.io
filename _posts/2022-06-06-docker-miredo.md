---
layout: post
title: "[Docker] docker에서 miredo 사용하기"
description: "docker에서 miredo를 통해 IPv6 터널링을 해보자"
img: /title/docker_miredo.png
tags: [Docker]
---

대학교에서 컴퓨터 네트워크개론 과제 중 IPv6를 사용해야 할 수 있는 과제를 마주하였습니다. 과제 때문에 인터넷 서비스 업체에 IPv6 주소를 신청할 수는 없으니 수업에서 마련한 방안은 `Miredo`를 이용하는 것이었습니다.

> Miredo는 IPv4 기반 인터넷에 있지만 IPv6 네트워크에 직접 연결되지 않은 컴퓨터 시스템에 대한 완전한 IPv6 연결을 허용하도록 설계된 Teredo 터널링 클라이언트입니다. (위키피디아)

설명대로 실제로는 IPv4를 사용하지만 터널링 기법을 통해 외부에서 마치 `IPv6`를 사용하는 것처럼 연기할 수 있게 해주는 기술입니다. Miredo는 `Linux`와 `BSD` 운영체제 내에서 동작하기 때문에 Macbook을 사용하는 저는 여느 때와 같이 `Docker`로 Ubuntu 컨테이너에 접속하였습니다. 

```sh
apt-get install miredo
service miredo start
```

위와 같은 명령어로 Miredo를 설치한 뒤 활성화 해줄 수 있습니다. 그런데 아래와 같은 문장과 함께 miredo를 실행할 수 없다고 뜹니다.

![service-nothing](/assets/img/etc/docker_miredo/service_nothing.png){: width="70%" height="70%"}

그리고 Miredo가 실제로 잘 수행되고 있는지 `ifconfig` 명령어로 확인해본 결과는 다음과 같습니다. 정상적으로 작동된다면 터널링 기법으로 Global한 IPv6 주소를 가지는 teredo 인터페이스가 생성되어야 할 것입니다. 

![ifconfig-nothing](/assets/img/etc/docker_miredo/ifconfig_nothing.png){: width="70%" height="70%"}

역시나 teredo라는 인터페이스조차 생성되지 않았네요...:pensive:

# 해결책

가장 먼저 해야할 점은 docker 컨테이너를 `Privileged` 모드로 생성해야 합니다. Privileged 모드는 컨테이너가 호스트의 기능들을 쓸 수 있게 해주는 모드입니다. 아쉽게도, 현재 가지고 있는 컨테이너의 모드를 바꿀 수는 없고 아래와 같이 새로 생성해야 합니다.ㅎㅎ

```sh
docker run --privileged -it --name <container 이름> <image 이름>
```

이것이 끝은 아니고요. docker 컨테이너에는 기본적으로 IPv6 기능이 막혀 있습니다. 그래서 컨테이너가 IPv6를 사용할 수 있도록 아래와 같이 커널 매개변수를 바꿔주어야 합니다.

```sh
sysctl net.ipv6.conf.all.disable_ipv6=0
sysctl net.ipv6.conf.default.disable_ipv6=0
sysctl net.ipv6.conf.lo.disable_ipv6=0
```

그런 다음에 다시 Miredo를 재시작해봅시다.

```sh
service miredo restart
```

다시 한 번 ifconfig로 확인해볼까요?

![result](/assets/img/etc/docker_miredo/result.png){: width="70%" height="70%"}

이렇게 되면 이제 teredo의 `inet6`라고 적혀져 있는 global IPv6 주소를 내 주소로 사용할 수 있게 됩니다!!! (빨간 박스로 표시해놓은 부분)`ping`을 사용해서 확인해보면 확실히 주소가 유효함을 알 수 있습니다.:stuck_out_tongue:

**+** 호스트가 실제로 사용하고 있는 인터넷 환경에 따라 teredo가 적용될 수도 있고 안되는 경우도 많으니 이 점 참고하시기 바랍니다.
**+** 그래도 작동되지 않는다면 `apt-get update`나 miredo를 재설치 해보시기 바랍니다.








