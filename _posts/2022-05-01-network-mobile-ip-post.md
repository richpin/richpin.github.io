---
layout: post
title: "[Network] Mobile IP"
description: 단말이 움직이는 모바일에서는 어떻게 네트워킹이 이루어질까?
img: /title/mobile_ip.webp
tags: [Network]
---

우리가 자주 쓰는 스마트폰에서 데이터를 쓰는 것이 어떻게 가능할까요? 즉, 모바일 단말에서는 어떻게 네트워크가 이루어질까요? 

애초에 `IP addressing`은 호스트가 하나의 네트워크 안에 `Stationary`하게 위치해있는 것을 기준으로 만들어졌습니다. 어떠한 IP주소 또한 그 주소에 해당하는 네트워크 안에 위치할 때 `Valid`한 상태가 되는 것이 당연합니다. 그러나, 모바일 노드는 계속해서 네트워크를 옮겨 다닙니다. 그리고 네트워크가 바뀌기 때문에 원할한 네트워킹을 위해서는 **주소 또한 그에 맞게 바뀌어야 하는 것이지요.** 가장 확실한 방법은 재부팅을 하는 것입니다. 그러나 움직이면서 네트워크가 바뀔 때마다 폰을 재부팅한다는 것은 상상만 해도 끔찍한 일이 아닐 수 없습니다. 또 하나의 문제는, 전송이 일어나는 도중에 네트워크가 바뀌게 되면, 데이터 교환에 있어 차질이 생기게 되는 것이지요. `WIFI`는 지원하지 못하는 이러한 `Mobility` 문제를 해결하기 위해 `Mobile IP` 프로토콜이 생기게 되었고, 그 예로 우리가 현재 사용하고 있는 LTE 등이 있는 것입니다.

# Moblie IP

![mobile-internet](/assets/img/network/network_mobile_ip/mobile_internet.png){: width="70%" height="70%"}


Mobile IP에서 인터넷과 연결된 네트워크에는 크게 두 가지가 있습니다. `Home Network(HN)`은 폰을 처음 등록하는 변하지 않는 `Permanent`한 네트워크 입니다. 즉 우리가 가입을 하면 통신사에서 처음 딱 한 번 정해주는 말뚝과 같은 통신사의 네트워크인 것이죠. 그리고 `Foreign Network`는 모바일이 움직일 때마다 그 곳에 해당하는 네트워크를 지칭하는 것으로, 계속해서 바뀌는 `Temporary`한 네트워크입니다. Home Network에서 쓰는 주소를 `Home Address(HoA)`, Foreign Network에서 쓰는 모바일 노드의 주소를 `Care-of-address(COA)`라고 합니다. 추가적으로, Home Network와 연결된 라우터를 `Home Agent(HA)`, Foreign Network와 연결된 라우터를 `Foreign Agent(FA)`라고 하며 이 둘은 인터넷으로 연결되어 활발한 통신을 수행합니다.

이제 과정을 순서대로 알아볼까요? 추가적으로, 여기서는 우리의 스마트폰을 `Mobile Node(MN)`, 우리가 인터넷을 통해 Connection을 맺으려 하는 노드를 `Correspondent Node(CN)`이라고 합시다.

먼저 가장 편한 상황입니다. 노드가 Home Network에 있는 상황입니다.

![connection](/assets/img/network/network_mobile_ip/connection.png){: width="50%" height="50%"}

이 때는 평범하게 마치 정적인 네트워크 처럼 모바일 노드가 Home Address를 이용해서 CN과 통신합니다. 하지만 이런 경우는 거의 존재하지 않겠죠? 우리는 `Mobility`를 해야하지요!

![movement](/assets/img/network/network_mobile_ip/movement.png){: width="50%" height="50%"}

우리의 모바일 노드가 새로운 Foreign Network로 이동을 합니다. 그렇다면 Foreign Agent는 모바일 노드를 `detect`하고 가지고 있는 `CoA` 중 하나를 할당해 줍니다. 이와 동시에 Home Agent에게 우리의 모바일 노드가 이 CoA를 가진다는 `Binding Update`를 날려주게 됩니다. Home Agent는 이를 받아 `Binding Cache`의 `Mapping`을 설정해주는 것이지요. 이걸로 세팅은 끝난 겁니다! 간단하지요?:grin: 그렇다면 이제 CN과의 통신이 어떻게 이루어질지 확인해봅시다.

먼저, 모바일 노드에서 CN으로 패킷을 보낼 때를 살펴봅시다.

![mtc](/assets/img/network/network_mobile_ip/mn_to_cn.png){: width="50%" height="50%"}

모바일 노드는 Foreign Agent를 통해 직빵으로 CN에게 패킷을 전달합니다. 평범하지요? 그러나 눈여결 볼 점은 Source Address가 **HoA**라는 점입니다. 이는 처음부터 폰 안에 다 등록이 되어 있기 때문에 가능한 것인데요. 그런데 이런 식으로 자신이 속한 네트워크가 아닌 주소로 보내도 되는 것일까요? 네, 상관없습니다! 어차피 라우터는 Destination Address만 보고 패킷을 Forwarding하기 때문에 Source Address가 다른 것은 문제가 되지 않기 때문입니다. 그렇다면 왜 HoA로 보낸 것일까요? 다음 상황을 보면 이해가 되실 겁니다.

이번에는 CN에서 모바일 노드로 패킷을 보낼 때를 살펴봅시다.

![ctm](/assets/img/network/network_mobile_ip/cn_to_mn.png){: width="60%" height="60%"}

CN은 바로 Foreign Agent로 보내는 것이 아닌. Home Agent로 패킷을 전송합니다. 그래서 이전에 HoA로 패킷을 보낸 것입니다. CN에서는 응답으로 Source와 Destination을 바꿔서 패킷을 보내기 때문이죠. 그렇게 온 패킷에 Home Agent는 IP 헤더에 새로운 헤더를 하나 더 붙여 Foreign Agent로 전송합니다. `Tunneling`이 형성되는 것이지요. 패킷을 받은 Foreign Agent는 헤더를 잘라나고 패킷을 읽어 모바일 노드로 보내게 됩니다. 

그런데 이런! 당연하게도 Destination이 여전히 HoA를 가리키고 있네요. 상관없습니다! Foreign Agent에서 모바일 노드까지는 전달을 **2계층** `MAC`에서 하기 때문입니다. 아까 모바일 노드에 HoA가 이미 등록이 되었다는 점을 명시했었죠? 처음 Foreign Network로 이동해 Foreign Agent로부터 CoA를 받을 때 모바일 노드는 자신의 HoA와 그에 대응하는 자신의 MAC주소를 Foreign Agent에 등록을 해놓습니다. 그래서 Foreign Agent는 해당 HoA를 보고 그에 맞는 MAC 주소로 패킷을 흘려보내기만 하면 되는 것이죠!:smile:

이런 식으로 Mobile IP가 동작하는 것입니다. 결과적으로, CN은 Home Network에게만 패킷을 보내게 되기 때문에 모바일 노드가 어디 있는지, 네트워크 주소는 무엇인지를 알 수가 없습니다. `Transparency`를 보장하게 되는 것이지요. 이런 식으로 전국 곳곳에 Foreign Network, 그리고 그것을 관장하는 Foreign Agnet가 퍼져있어 우리는 편리하게 데이터를 쓸 수 있는 것입니다. 

## Mobile IP 방식의 Inefficiency

![traingle-routing](/assets/img/network/network_mobile_ip/triangle_routing.png){: width="60%" height="60%"}

정말 우연찮게, 통신하고 있는 CN(Remote Network)이 바로 Foreign Network와 연결되어 있을 수도 있습니다. 이렇게 바로 연결이 되어 있어도, 패킷은 굳이 인터넷으로 통해 Home Network를 거쳐 Foregin Network의 모바일 노드로 이동하게 되는데요. 이런 경우 이동 경로가 삼각형과 같은 모양을 하고 있어 `Triangle Routing`이라고 부릅니다.

![double-crossing](/assets/img/network/network_mobile_ip/double_crossing.png){: width="60%" height="60%"}

더 최악적이게도, 만약 CN이 모바일 노드와 같은 Foreign Network의 경우에 있다면요? 굳이 인터넷을 두 번 거쳐 한바퀴를 빙~둘러서야 패킷이 오게 되겠지요? 이를 전문 용어 `Double Crossing`이라 칭합니다.