---
layout: post
title: "[Network-01] Network Models"
description: Network 공부를 시작하기 전 기초를 다져보자.
img: /title/network_models.webp
tags: [Network]
---

네트워크의 기초를 다지기 위해 가장 기본적인 `Network Model`을 알아봅시다.

# Simplified Network Model

![simplified](/assets/img/network/network_models/simplified.png){: width="90%" height="90%"}

사람과 사람이 소통하는 데 필요한 정보는 `Analog` 정보입니다. 글, 비디오, 음성, 사진 등이 있겠지요. 하지만, 이를 네트워크를 통해 주고받기 위해 사용하는 컴퓨터는 `Digital`만을 다룰 수 있습니다. 그래서 컴퓨터는 Analog 정보를 Digital 데이터로 바꾸는 `Encoding` 과정을 수반하게 됩니다. 다시 말해 `Information`을 `Data`로 바꾸는 과정이라고도 할 수 있습니다. 하지만 실제로 전송을 하기 위해서는, 쉽게 말하자면 유선을 타거나 무선으로 쏘는 과정을 위해서는 다시 이 `Digital` 데이터를 `Analog` 신호로 바꾸는 `Modulation` 과정을 거쳐야 합니다. 다시 말해 `Data`를 `Signal`로 바꾸는 과정이라고도 할 수 있습니다. 그렇게 네트워크를 거쳐 이번에는 반대로 `Demodulation`과 `Decoding`을 거쳐서 수신 단말이 전송된 정보를 받게 되는 것입니다. 이 과정에서 가장 중요한 핵심은 **Source와 Destination의 정보는 항상 같아야 한다는 것입니다.** 그래야 의미가 있을 테니까요.:sweat_smile:

이제 아까 실질적인 전송이라고 뭉뚱그려 이야기 하였던 `Transmission System`에 대해 알아보도록 합시다. 이것이 정말 네트워크의 핵심이라고 할 수 있겠지요? 네트워크는 범위에 따라 크게는 `Local Area Network(LAN)`와 `Wide Area Network(WAN)`으로 나눌 수 있습니다.

먼저 **LAN**은 직역하자면 근거리 통신망으로 집, 사무실, 학교 등의 건물과 같은 가까운 지역을 한데 묶는 컴퓨터 네트워크를 의미합니다. 이들은 서로가 `WiFi`나 `Ethernet` 등을 통해서 물리적으로 연결이 되어있는데 이 물리적 연결을 관장하는 프로토콜이 바로 `Medium Access Control(MAC)`입니다. 위의 예시를 보면 3개의 `Host`가 하나의 `Medium(위와 같은 경우는 ethernet)`에 연결되어 있죠? 이와 같은 형태를 `Bus-Topology`라고 하며, 이 때 각 host들을 구분하기 위해 각각에 부여되는 주소를 `MAC Address`라고 합니다. 그런데, 이렇게 되면 문제가 3개가 하나의 길을 이용하기 때문에 성능이 약 1/3로 줄어들게 되는 것이며, 이 과정이 원할히 일어날 수 있도록 교통 정리를 해주는 것이 바로 MAC의 역할인 것입니다. 

추가적으로 이와 같은 비효율성을 위해 개발된 것이 바로 `ethernet HUB`입니다. HUB는 ethernet을 `Emulation`해서 하나의 컴퓨터(host)가 마치 자신만의 medium(ethernet)을 가지는 것과 같이 통신할 수 있게 해줍니다. 

![ethernet_hub](/assets/img/network/network_models/ethernet_hub.png){: width="80%" height="80%"}

HUB을 통해 이제 여러 host들이 위와 같이 가장 널리 사용되고 있는 `Star Topology` 형태를 갖추게 됩니다. 정확히는 HUB가 `Buffering`을 해주어 각 호스트들이 더 이상 다른 호스트들을 기다릴 필요 없이, 마치 자신이 하나의 ethernet에 연결된 것처럼 행동할 수 있는 것입니다. MAC의 교통 정리를 훨씬 효율적이도록 도와주는 셈이지요. 그런데 HUB는 한 host에서 주고받는 데이터가 같은 허브에 연결된 다른 모든 host에 전달됩니다. 따라서 연결된 컴퓨터의 개수가 많아질 수록 네트워크에서 충돌(collision)이 많아지고 속도가 느려지는 치명적 문제가 존재하였지요. 이 문제를 해결하기 위해 최근에는 데이터를 필요로 하는 host에만 필터링하여 전송하는 `ethernet Switch`를 많이 사용합니다.

**WAN**은 이러한 LAN들이 `Router`라는 Node로 연결된 가장 넓은 범위의 네트워크로, 우리가 생각하는 전세계의 인터넷이라고 이해하셔도 좋습니다. 실제로 물리적인 전송은 위의 선으로 표시된 LAN의 MAC으로서 수행되고 노드인 Router는 오직 나아가야 할 경로만을 선정해주는데, 이를 **Switch**한다고 하여 인터넷을 `Switched Network`라고 부르기도 합니다. 

> Switch는 하나의 네트워크를 연장해주는 개념이며, Router는 서로 다른 네트워크를 연결해주는 개념입니다.

다시 한 번, 아래의 그림을 통해 확인해봅시다.

![packet_model](/assets/img/network/network_models/packet_model.png){: width="70%" height="70%"}

가장 첫 번째 그림을 이해했다면, 위 그림에서 빨간색으로 표시된 부분이 LAN을 나타냄을 아실 수 있을 겁니다. 이렇게 LAN안에서 물리적으로 연결된 디바이스들끼리의 통신을 `Direct delivery`라고 합니다. 이 LAN들이 Router들로 연결되어 검정색으로 표시된 부분과 같이 하나의 `Local Network`를 형성합니다. Local Network는 LAN, WAN과는 다른 개념으로 LAN들이 모여 이룬 하나의 작은 네트워크라고 보시면 됩니다. 예를 들어, 아파트 하나에 하나의 Router가 배치되어 각 호수들이 HUB를 통해 연결되어 있다고 해봅시다. 그렇다면 아파트 한 동이 하나의 LAN이며, 그것들이 모인 아파트 단지가 하나의 Local Network라고 부를 수 있는 것입니다. 이 Local Network안에서 일어나는 통신은 Local한 개념이기는 하지만 LAN으로 물리적으로 연결된 것은 아니기 때문에 `Local indirect delivery`라고 합니다. 그것들이 멀리 떨어진 `Remote Network`까지 전달되기 위해서 인터넷이라는 거대한 WAN을 거쳐야 할 것입니다. 이 통신은 `Internet indirect delivery`라고 합니다.

![lan_wan](/assets/img/network/network_models/lan_wan.png){: width="80%" height="80%"}

LAN과 WAN의 차이점에 대해 간단히 정리한 표는 위와 같습니다. 우리는 오른쪽을 다시 볼까요? Campus라는 하나의 Local Network에서 인터넷을 통한 통신을 할 때에는 결국 모든 통신이 하나의 Router를 통해 인터넷의 WAN으로 뻗쳐나가게 되는데 이 라우터를 입구 Router 또는 `Border Router`라고 합니다. 계약하고 있는 인터넷 서비스 업체, 즉 `Internet Service Provider(ISP)`가 KT라면 위와 같이 Border Router가 KT의 기지국과 연결되어 전세계 인터넷으로 연결될 수 있는 것입니다. 조금 더 작은 단위로 들어가 볼까요?

![campus_network](/assets/img/network/network_models/campus_network.png){: width="60%" height="60%"}

하나의 Local Network를 좀 더 깊게 들어가 볼까요? 위와 같이 하나의 사옥에서도 부서 단위로 여러 LAN으로 나누어져 있을 수 있습니다. 이와 같이 하나의 Local Network는 여러 개의 `Subnetwork`로 나누어져 있습니다. 그렇다고 Subnetwork가 하나의 LAN을 의미하는 것은 아닙니다! Local Network의 모든 하위 네트워크를 Subnetwork라 부르며, 기준이 없는 개념적인 단어입니다. 앞서 설명한대로 각 Subnetwork와 연결된 Router가 또 사옥 전체의 Border router로 연결이 되며, 대부분의 Border router는 `Firewall`이라는 시스템을 수반하고 있습니다. 우리에게 방화벽이라는 한국어로 더 친숙한 Firewall은 미리 정의된 보안 규칙에 기반하여, 들어오고 나가는 네트워크 트래픽을 모니터링하고 제어하는 네트워크 보안 시스템입니다. 간단하게 말하면, Local Network의 보안을 지키기 위해 모든 트래픽이 통하는 Border router가 벽이 되어주는 것이지요. 물론, 이런 방화벽의 기능으로 인해 원하는 통신을 못하는 경우가 생길 수 있는데, 이때 Firewall이 파악하지 못하도록 암호를 걸어 제재를 피할 수 있습니다. 이를 `Virtual Private Network(VPN)`라고 하는데, 4장의 Network Layer protocol에서 다루도록 하겠습니다.:yum:

![small_network](/assets/img/network/network_models/small_network.png){: width="60%" height="60%"}

조금 더 작은 단위로 들어가봅시다. 그렇게 계층구조의 아래로 내려오다 보면 하나의 작은 공간(방) 단위까지 내려오게 되는데요. 다시 HUB나 Switch를 통해 여러 사람들이 연결될 수도 있지만, 유선이 아니라 편리하게 무선을 사용하고 싶으면 MAC으로 WiFi를 사용하는 `Access Point(AP)`나 `Wifi 공유기`를 사용할 수 있습니다. 이중 우리가 일상에서 주로 사용하는 공유기에 대해서 조그마한 설명을 덧붙이자면, 공유기는 여러 명이 무선을 이용해서 하나의 IP주소를 사용할 수 있게 합니다. 이처럼 하나의 IP를 공유하다 보니, Wifi를 통한 미러링 같은 것들도 가능할 수 있는 것입니다. 이러한 기술을 `Network Address Translation(NAT)`이라고 하며, 7장에서 다루도록 하겠습니다.:yum:

# Switched Networks

네트워크 모델을 설명할 때 노드, 즉 라우터는 들어오는 데이터의 경로를 설정하여 그대로 내보내는데(구체적으로 들어가면 정말 '그대로'는 아니지만 지금은 그렇게 이해하셔도 좋습니다. 이를 `Switch`한다라고 했습니다. 다른 말로는 `Forward`한다고도 표현할 수 있습니다. 그래서 이러한 네트워크를 `Switched Network`라고 표현하며, Router에서 트래픽을 내보내는 과정을 `Forwarding`이라고 합니다. Forwarding에 대해서는 또 3강에서 자세히 다룰 예정입니다.

그런데, 이러한 Switched Network에도 두 가지 종류가 있습니다.

![switching](/assets/img/network/network_models/switching.png){: width="70%" height="70%"}

먼저, `Circuit Switching`은 경로가 사전에 고정되어 있을 뿐더러 공유되지도 않습니다. 무조건 독점적인 물리적, 논리적으로 연결된 경로가 있어야 하기 떄문에 데이터 전송 전에 경로가 설정되어야 합니다. 초창기 전화 모델에서는 이러한 이유로 교환원이 직접 전화를 하는 양 단말의 경로를 연결해 주었습니다. 

물리적으로 연결이 되어있기 때문에, 한 번 연결이 되면 전송이 명백하다는 장점이 있지만 그보다 더 많은 단점들이 존재했죠. 하나의 경로에 하나의 연결만이 존재하기 때문에 연결이 없을 때 idle한 시간이 길어져 `Capacity`의 낭비가 심해 비효율적인 문제가 너무 큽니다. 뿐만 아니라 한 번 연결을 하기 위해 시간이 걸리는 `Delay` 문제도 간과할 수 없었지요.

이와 달리 `Packet Switching`은 경로가 고정되어 있지 않으며 많은 Connection들끼리 공유하는 특징을 가지고 있습니다. 하나의 node-to-node `link`에 같은 시간에 많고 다양한 데이터가 지나갈 수 있기 때문에 엄청난 효율성을 자랑합니다. 이와 같은 특징으로 Packet Switching은 1970년 대 인터넷의 초시가 드리우는데 기반이 되었던 것이지요. 또한 노드가 buffering을 통해 데이터의 속도를 조절할 수 있으며, 데이터의 우선순위를 반영할 수 있다는 점 또한 강력한 장점이 될 수 있습니다. 물론 특정하게 어떤 링크가 유독 바쁘게 되어 Delivery가 느려질 수 있는 단점이 있을 수 있기에 이를 대비하고 보안하기 위한 여러 방법들이 존재하며 차후의 내용에서도 그것들 중 몇 가지를 알 수 있을 겁니다.

Packet Switching에 대해 조금 더 자세히 알아볼까요? Packet Switching에서 데이터는 작은 `Packet`이라는 단위로 전송이 됩니다. 전형적으로 packet의 크기는 약 1000바이트 정도이며, 엄청나게 큰 데이터도 네트워크 내에서는 이 packet이라는 단위로 쪼개져 전송이 일어나는 것이지요. 각각의 packet들은 일부 `user data`와 몇몇 `control information`으로 구성되어 있습니다. 이 control information은 주로 `Routing`과 `Addressing`으로 활용이 되는데 각각 5, 6장과 2장에서 다룰 예정입니다.

또한 언급하였던 `Switching`에도 두 가지의 모드가 있습니다. 먼저, `Store-and-forward` 모드는 모든 패킷이 전체가 들어와 에러 처리를 포함한 모든 과정이 끝난 후 내보내는 것, 즉 forwarding을 진행하는 모드입니다. 이에 반해 `Cut-through` 모드는 조금 더 빨리 전송을 하기 위해 목적지만 확인하면 바로 forwarding을 진행하는 모드입니다.

# 5 Layers in Packet Switched Networks

![5_layer](/assets/img/network/network_models/5_layer.png){: width="90%" height="90%"}

Packet Switched Networks에는 크게는 위와 같이 총 **독립적인** 5개의 **계층**이 존재합니다. (이후에는 더 세부적으로 7계층으로 나누어집니다.)먼저 1계층은 `Physical Layer`로 말 그대로 물리적인 하드웨어로서의 전송을 의미합니다. 정말 간단하게 빛으로 슝슝 이동하는 0과 1의 데이터들을 생각해볼 수 있겠지요? 2계층은 `Data Link Layer`로 초반에 설명하였던 MAC을 의미합니다. 물리적으로 연결된 같은 LAN안에서 MAC 주소를 통해 전송이 이루어지는 계층으로 네트워를 공부할 때만큼은 (1계층이 있지만) 실적인 데이터 이동이 이루어지는 계층으로 이해하셔도 좋습니다. 3계층은 `Network Layer`로 node, 즉 router에서 IP주소를 통해 전송이 이루어지는 계층입니다. 

그렇게 router에서나 router로 부터 패킷이 이동할 때에는 3계층의 Network Layer에서 IP를 통해 경로를 찾고 2계층의 Data Link Layer의 MAC에서(실제로는 1계층의 Physical Layer도) 실질적인 데이터 전송이 이루어지는 것입니다. 이 이동의 단위를 hop이라 표현하기 때문에 `hop-to-hop delivery`라고 표현합니다. 

4, 5계층은 양쪽 단말(디바이스)끼리만 사용되기 때문에 `peer-to-peer protocol`이라 부르는 것이며, 이 이동의 단위를 `end-to-end delivery`라고 표현합니다. 4계층은 `Transport Layer`로 주로 패킷의 에러를 잡거나 놓친 패킷을 구별하고 패킷 전송의 속도를 조절하고 패킷을 구분합니다. 5계층은 `Application Layer`로 우리가 네트워킹을 하기 위해 실제로 사용하는 프로그램, 즉 어플리케이션을 의미합니다.

`Network Layer`에 대 조금 더  알아볼까요? 3계층인 Network Layer의 목표는 서로 다른 네트워크와 라우터를 통해 source에서 destination까지 forward되도록 하는 데 있습니다. 각각의 호스트와 라우터들이 **IP**라는 network layer address로 식별 됩니다. 이는 LAN에서 사용하는 MAC이라는 이름의 data link layer address와는 완전히 **독립적**임을 알아야 합니다. Network Layer의 서비스는 또한 `Unreliable connectionless service`와 `Reliable connection-oriented service`로 나눠질 수 있습니다.

![datagram](/assets/img/network/network_models/datagram.png){: width="80%" height="80%"}

Network Layer의 Unreliable connectionless service를 다른 말로 `Datagram`이라고도 표현하며, 우리가 쓰는 IP가 이에 해당하기 떄문에 이것으로 3계층을 이해하셔도 무방합니다. Datagram은 **3계층 관점에서는** connectionless 즉 두 호스트 간에 확실한 connection이 성사되지 않습니다. 각각의 패킷들은 모두 독립적으로 처리되기에 서로가 실용적이라 판단한 다른 루트를 통할 수 있는 것이지요. 그렇기 때문에 순서가 서로 뒤죽박죽이 될 수 있으며, 몇몇 패킷들은 경로를 잘못 타서 사라질 수도 있는 것이지요. 이 또한, 그에 대한 대비책을 뒤의 글을 통해 차차 알아갈 수 있을 겁니다. 그래도 Datagram이 이러한 특성으로 갖는 유연함 덕분에 현재 우리가 쓰는 IP가 이를 채택하고 있는 것입니다.

`Transport Layer`는 Network Layer와 Application Layer 사이에 있죠? 4계층이 Transport Layer는 어플리케이션에서 데이터의 사용이 가능하도록 Network Layer가 제공하는 서비스를 향상시키는 역할을 수행합니다. `Reliablity`와 `Multiplexing` 등이 그 예시가 될 수 있겠지요. 4계층도 마찬가지로 `Unreliable connectionless service`와 `Reliable connection-oriented service`를 가지는 데 4계층에서는 각각의 목적과 용도에 따라 두 가지 서비스를 모두 사용합니다. 전자를 `UDP`, 후자를 `TCP`라 칭하며 이에 대한 내용들을 추후에 자세하게 다루니 확인해 보시기 바랍니다.







