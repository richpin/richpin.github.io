---
layout: post
title: "[Network] Network Address Translation(NAT)"
description: IP 주소를 더 효율적으로 사용하기 위한 NAT를 알아보자.
img: /title/nat_img.jpeg
tags: [Network]
---

이번에는 IP 주소를 더 효율적으로 사용하기 위한 **NAT**에 대해 알아봅시다.

# NAT란 무엇인가?

IP가 부족해지면서 여러 해결책들이 도입된 것을 이전에 언급하였는데, NAT도 그 중 하나입니다. 하나의 IP를 이용해서 여러 호스트들의 네트워킹을 지원하기 위한 방법이지요. 일반 가정이나 회사(Organization)은 `Private Address`(사설 주소)를 많이 사용하게 되는데요. 사설 네트워크 IP주소는 다음과 같이 정해져 있습니다.
- 10.0.0.0/8
- 172.16.0.0/12
- 192.168.0.0/16

설명하자면, 그 사설 주소를 Routing이 가능한 외부 공인 IP 주소로 변환해주는 것이 NAT라고 할 수 있습니다. 다음 그림과 같이 Routing이 가능한 하나의 `Stub Network(Private Address Space)`의 edge에 NAT 기능이 들어간 `Border Router`를 다는 것으로 주로 사용됩니다. Space 내부에서는 `private(local) address`, 외부에서는 `public(official) address`를 사용하는 것입니다.

![nat-background](/assets/img/network/network_nat/nat_background.png){: width="50%" height="50%"}

회사와 같은 곳에서 내부에서는 사설 IP 주소를 통하여 그들끼리 통신하고 회사 밖 인터넷은 회사의 IP로써 통신하는 구조에서 사용된다 볼 수 있겠습니다.

# NAT는 어떻게 동작할까?

![nat-how](/assets/img/network/network_nat/nat_how.png){: width="50%" height="50%"}

위와 같은 그림으로 설명을 해보겠습니다. PC1과 Web Server와의 통신을 예로 들어보죠.
1. PC1은 SRC IP에 자신의 사설 주소, DST IP에 Web Server의 IP주소를 적어 packet을 전송합니다. 
2. 패킷은 모뎀과 라우터 R1(Private Address를 가진 Router)를 타고(forwarding) Border Router인 R2에 도달하게 됩니다. 
3. NAT 기능을 가진 R2는 SRC IP의 192.168.10.10을 Public IP인 209.165.200.226으로 `Mapping` 합니다(바꿔줍니다).
4. 그 패킷은 여러 Router를 타고 Web Server에 도달합니다. 그 후, 서버는 SRC와 DST를 바꿔 클라이언트(PC1)으로 packet을 전송합니다.(DST는 Public인 209.165.200.226가 되겠죠?)
5. R2는 도달한 패킷을 보고 `NAT table`을 보고 아까 Mapping 하였던 entry를 확인해 다시 사설 주소로 바꿔줍니다.
6. packet은 다시 타고 타서 PC1에 도착하게 됩니다.

간단하죠? 다만 이러한 과정은 'packet 송신 과정 중 **SRC IP와 DST IP는 절대 바뀌어서는 안된다'**는 규약을 어기게 되는 문제가 있습니다. 그렇지만 IP가 부족한 것이 먼저라 감수해야 하는 문제인 것 같습니다.:cry: 조금 더 구체적으로 NAT의 원리를 알아보도록 합시다.

## STATIC NAT

정적 방식은 단순히 사설 주소와 공인 주소를 `Mapping Table`에서 `one-to-one mapping`하는 것입니다. 그렇기 때문에 어떤 사설 IP가 어떤 공인 IP와 매칭하는지가 모두 미리 세팅되어 있어야 합니다. 해야 할 일이 확실히 많은 대역 서버 같은 경우 이와 같은 방식을 사용하는 것이 유리할 수 있는 것이지요.

## DYNAMIC NAT

![nat-pool](/assets/img/network/network_nat/nat_pool.png){: width="50%" height="50%"}

그러나, 우리는 대부분 정적인 방식보다 동적인 방식에 더 효율성을 느낍니다. 동적 방식은 보유하고 있는 공인 IP 주소의 `Pool`에서 호스트의 요청이 올 경우 동적으로 IP를 할당해 줍니다. 아무래도 정적인 방식은 하나의 공인 IP와 mapping된 호스트들의 요청이 동시에 쏟아지게 되면 하나를 제외한 나머지 호스트들이 놀고 있어야 할텐데, 동적인 방식은 정해진 게 없어 남은 공인 IP를 할당해주면 되기 때문에 조금 더 대처에 유연합니다.

# PORT ADDRESS TRANSLATION(PAT)

![pat-example](/assets/img/network/network_nat/pat_example.png){: width="70%" height="70%"}

그렇다 하더라도 STATIC이든 DYNAMIC이든 결국 공인 IP 주소가 결국 호스트의 개수보다 적을 수 밖에 없고, 이 때문에 동시에 들어오는 요청들을 모두 처리할 수가 없게 됩니다. 그렇다면 부족한 IP 주소를 해결하고자 하는 NAT의 취지마저 한 끝도 달라진 게 없는데요? 그래서 우리는 바로 `Port` 번호를 사용하게 됩니다. 동일한 IP를 여러 호스트들에게 할당 해주더라도 여기에 각자의 Port 번호를 사용하여 각각을 구분하는 것이지요! 호스트의 Port 번호는 `Random`하게 정해지기 때문에 왠만하면 서로 중복되지 않다는 것도 한 몫 합니다. 같은 IP여도 어차피 Router에서 나눠지니 성능에는 문제가 없겠지요?**(Port 번호는 구분 용도일 뿐이지, Translation 이후에도 값이 완전히 동일한 것은 보장하지 않습니다!)**그것이 바로 NAT의 확장자인 `PAT`인 것입니다. 네트워크 통신에 있어 각각의 `Connection`을 구분하는 요소에는 최소한 아래와 같은 5가지가 있습니다.

1. Source IP
2. Destination IP
3. Source Port Number
4. Destination Port Number
5. Protocol

이들을 전문 용어로 `5 Tuples`라고 칭합니다. 이와 같은 특성을 이용하여 Port 번호로 서로 독립적인 통신 관계를 유지할 수 있는 것입니다. 물론 기본적으로 각각의 계층은 독립적이야 하는 규약에서 벗어나 3계층(IP)에서 4계층(TCP Port)를 사용하는 것인데요. 이 또한, NAT와 마찬가지로 PAT 또한 IP 부족의 문제로 인해 어쩔 수 없는 것이라 합니다.:sweat_smile: 만약 서로 다른 장비가 같은 Port 번호를 사용하는 경우가 있더라도 NAT 장비가 Port 번호를 임의적으로 변경하여 처리합니다. 그렇기 때문에 항상 다른 Port 번호를 보장하게 되는 것입니다. PAT는 오늘날 대부분의 Router들이 제공하며 `NAT overload` 또는 `IP masquerading`이라 불리기도 합니다. 또한, 실제로는 위와 같은 `/` 형태가 아닌 `:` 형태를 사용합니다. (ex. 128.143.71.21:2100)

번외
- 예를 들어 ICMP처럼 4 계층의 Port Number를 사용하지 않는 패킷들은 PAT에서 조금은 다르게 동작합니다.

# PORT FORWARDING

PAT는 기본적으로 packet이 나갈 때 자신의 사설 주소를 공인 주소로 매칭한 뒤에 `Table`에 저장하는 과정입니다. 그렇기 때문에 이후에 packet이 들어올 때도 **만든 Table을 참고**하여 올바른 호스트에게 전달할 수 있는 것입니다. 

**그렇다면 서버가 사설 주소를 이용할 때는 어떨까요?**

서버는 packet이 들어오는 것으로 시작하기 때문에 **PAT 과정을 거치지 않습니다.** 그래서 `Static`하게 mapping해놓은 것을 참고하여 `Private Space`로 `Forwarding`해야 하는데 이를 `Port Forwarding`이라고 합니다. 조금 더 깊게 생각해보면 Static PAT에서 들어오는 packet을 Forwarding하는 과정과 Port Forwarding이 비슷하다는 것을 느낄 수 있을 겁니다. 그러나 PAT는 처음 나가는 과정에서 사설 주소를 공인 주소로 `Translation`한다는 점에서 명확한 차이를 가집니다. Port Forwaring은 단순히 `Forwarding`하는 과정만을 의미하는 것이지요.

![port-forwarding](/assets/img/network/network_nat/port_forwarding.png){: width="70%" height="70%"}

보다 넓은 의미로는 말 그대로 **Port를 활용하여 Forwarding하는 과정** 자체를 Port Forwarding이라 칭합니다. 이와 같이 Port 번호로 구별되는 것을 활용해 인터넷은 아래와 같이 같은 IP여도 `Protocol`에 따라 Port 번호로 구별하여 사설 서버로 Forwarding을 하고 있습니다. (ex. HTTP = 80)

![pf-egoing](/assets/img/network/network_nat/pf_egoing.png){: width="60%" height="60%"}

다른 예로 `AP`에서 들어오는 packet이 연결되어 있는 디바이스 중 어떤 곳으로 Forwarding 되어야 하는지를 결정하기 위해 Port Forwarding을 사용합니다. 이런 Port Forwarding을 다른 말로는 `Port Mapping`이라 부릅니다.

# 장단점

마지막으로 NAT, 즉 PAT의 장점과 단점을 설명하자면 아래와 같습니다. 이미 언급한 부분들도 있겠습니다.:wink:

### 장점

+ 많은 사설 주소들을 적은 공인 주소로 변환하는 과정이기 때문에 IP 주소를 `Conserve` 할 수 있습니다.
+ ISP를 바꾸더라도 공인 주소만 바뀔 뿐, 사설 주소를 갖는 호스트들과는 직접적인 연관이 없어 `Reassign`이 필요 없습니다.
+ 밖에서는 Space안의 주소를 알 수 없기 때문에 `Security`에 좋습니다.

### 단점

+ Translation 과정을 거쳐야 하기 때문에 `Latency`가 늘어나는 것을 피할 수 없습니다.
+ 패킷에 변화가 생기는 만큼 헤더의 `Checksum` 계산을 위해 `Latency`가 늘어나는 것 또한 피할 수 없습니다.
+ IP의 `End-to-End` 관계를 숨기기 때문에 몇 가지 어플리케이션에서 사용이 불가합니다.
+ IP 주소를 숨기기 때문에 `Tracking`을 불가능하게 됩니다.(보안에 좋은 장점과는 상반)


