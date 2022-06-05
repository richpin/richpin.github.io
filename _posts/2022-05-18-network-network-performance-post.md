---
layout: post
title: "[Network] Network Performance"
description: 네트워크의 성능은 어떻게 결정될까?
img: network_pipe.webp
tags: [Network]
---

이번 시간에는 네트워크의 성능을 어떻게 알아볼 수 있을지 알아보도록 합시다.

# Bandwidth

가장 먼저 `Bandwidth`란 초당 전송되는 비트 수, 즉 `Number of bits per second(bps)`입니다. 많은 사람들이 착각하는 것은 Bandwidth로 속도를 측정한다는 것입니다. 그게 아니라 데이터가 흐르는 파이프의 직경이라는 뜻에 더 걸맞습니다.

![bandwidth_pipe](/assets/img/network-performance/bandwidth_pipe.png){: width="50%" height="50%"}

위와 같이, 물이 파이프를 타고 배수될 수 있을 때, 파이프의 직경이 크다면 속도가 늘어나나요? 실제로 물 단위의 속도, 즉 전송되는 데이터의 단위의 속도 자체에는 영향이 없습니다. 그렇지는 않지만 빠지는 물의 양은 늘어나겠지요? 그러나, 같은 **양**의 물이 빠지는 데, 즉 같은 크기가 데이터가 이동하는 데 걸리는 시간은 더 짧아지겠죠. 같은 양이 전송되는 데 걸리는 시간이 더 짧다면 속도가 높은 게 아니냐구요? 파이프가 흐르는 속도와 어떤 양이 전송되는 속도는 서로 다른 개념입니다. 

네트워크로 다시 넘어온다면 현 시대에 데이터, 즉 비트는 빛으로 움직이죠? 따라서 파이프가 흐르는 속도는 **빛의 속도** 이고, 어떤 양이 전송되는 속도는 다운로드 되는 속도를 의미하는 것이죠. 따라서 이 Bandwidth는 속도라는 개념에 직접적으로 관여하지는 않지만, 다운로드와 같은 양적인 개념에 사용됩니다. 바로 다음 내용에서 설명하곘습니다.

# Latency

`Latency`는 데이터가 이동하는 데 걸리는 시간을 통칭적으로 뜻하는 개념입니다. 아까 데이터의 속도는 언제나 빛의 속도라고 말씀 드렸죠? 그럼 예를 들어봅시다. 4320km 만큼 떨어져 있는 곳끼리 데이터를 주고 받는 데 걸리는 시간은 빛의 속도인 200,000km/s를 나눈 21.6ms가 될 겁니다. 따라서 이상적으로 Latency가 21.6ms가 되고, 그로 인해 RTT는 43.2ms가 되곘지요. 하지만 실제 값은 이보다 크답니다. 아래의 그림을 보시죠.

![actual_latency](/assets/img/network-performance/actual_latency.png){: width="50%" height="50%"}

방금 전 설명드린 부분을 `Propagation Delay`라고 합니다. 그러나 실제로는 Router에서 작업하는 `Processing Delay`나 Router에서 나가는 패킷보다 더 많이 들어오는 패킷들을 Queue에 저장하기 위한 `Queueing Delay` 등 더 다양한 Delay 요소들이 존재하지요. 또한, 대표적으로 `Transmission Delayd`가 있습니다.

![transmission_delay](/assets/img/network-performance/transmission_delay.png){: width="50%" height="50%"}

Transmission Delay는 보내는 `Size/Bandwidth`로 구할 수 있습니다. 이는 다운로드를 하는 데 걸리는 시간에 영향을 끼칩니다. 그림의 위와 같이 작다면 똑같은 크기를 받는데 걸리는 시간이 길어지게 됩니다. 반대로 아래와 같이 Bandwidth가 크다면 한 번에 더 많은 데이터들이 병렬적으로 들어올 수 있게 때문에, 다운로드 속도가 빨라질 수 있는 것이지요. 이 뿐만인가요? 실제로 여러 종류의 패킷들이 이동하는 링크에서 Bandwidth가 크다면 여유가 더 크기 때문에 더 많은 패킷들을 효율적으로 운송시킬 수 있습니다. 그래서 Bandwidth도 실질적으로 지대한 영향을 끼치는 것이지요. 다만 헷갈리시지 말아야 할 점은 패킷이 이동하는 속도는 같다는 점입니다. 그건 무조건 빛의 속도인 것이지요. 다시 한 번, Bandwidth는 속도가 아닌 양의 개념임을 기억하셔야 합니다.

![throughput](/assets/img/network-performance/throughput.png){: width="50%" height="50%"}

전체적인 네트워크 속도에 있어서 가장 중요한 것이 바로 `Throughput`인데요. Throughput이랑 단위 시간 당 보낼 수 있는 비트 수로 여러 요소들을 고려해 최종적으로, 그리고 가장 실질적으로 네트워크 성능을 판단할 수 있는 지표입니다. 이 Thrughput이 가질 수 있는 **최대치**가 바로 **Bandwidth**인 것이지요. 패킷이 이동할 때는 여러 링크들을 통과하죠? Throughput은 그 링크들 중에 가장 낮은 Throughput보다 더 빠를 수가 없는 것이 특징입니다. 예를 들어 가장 느린 링크에서 1초에 100비트를 넘기는데 Destination에서 과연 1초에 100비트 이상을 보낼 수 있을까요?

# Bandwidth Delay Product

TCP에서 우리는 두 말단이 서로 하나의 파이프로 연결되어 있다는 개념으로 시작했습니다. 그렇다면 TCP가 가장 바라는, 소위 말해 무릉도원은 어떤 형태일까요? 최대한 남김없이 데이터들을 꽉꽉 채운 효울이 극한인 상태, 즉 파이프가 데이터들로 빈틈없이 채워진 모습입니다. 보낼 수 있는 가장 많은 데이터를 쉴 틈 없이 계속해서 보낼 수 있는 상태를 얘기하죠. 그렇다면 얼마나 많은 양을 한 번에 보내야 채울 수 있을까요? 파이프가 수용할 수 있는 양, 시간 당 보낼 수 있는 최대 비트인 **Bandwidth**랑 파이프에서 데이터가 첨부터 끝까지 이동하는 시간인 **RTT/2**를 곱하면 되겠지요?(실제로는 굳이 2를 나누지 않고 그냥 RTT를 씁니다.) 정말 쉽게 설명해 파이프의 부피를 구하기 위해 직경 X 길이를 하는 개념입니다. 이 값을 직관적으로 Bandwidth와 Delay의 곱이라고 하여 'Bandwidth Delay Product`라고 합니다.

![bdp](/assets/img/network-performance/bdp.png){: width="70%" height="70%"}

그렇다면 Sender가 이렇게 파이프를 전체적으로 채우기 위해서는 어떤 상황이여야 할까요? ACK를 받기 위해 기다리면 기다리는 동안 데이터(패킷)을 못 보낼 테니까 그러면 흐름이 끊기고 중간에 구멍이 생겨 안되겠네요? 유일한 방법은 ACK 없이 계속해서 Bandwidth Delay Product만큼의 양을 보낼 수 있어야 하는데 그러려면 Receiver에 이 만큼에 Buffer가 있고 ACK에 최소 이만큼의 Window Size를 보내야 하겠네요! 다시 말해 `BDP`는 양 End-Point 간의 최대 데이터 처리량을 의미하며, 이 값이 넘어가면 무조건적으로 Congestion이 발생하게 되는 것입니다. 

실제로는 파이프들이 여러 다른 링크들로 이루어지기 때문에 가장 작은 Bandwidth로 구해야 하는데 이 링크들도 계속 유지되지가 않으니 **가변적**입니다. 그리고 Delay또한 실제로는 굉장히 **가변적**이지요. 그렇습니다. 이 값을 정확히 측정하는 것은 불능에 가깝습니다. 최대한 노력해보자 이겁니다. 이 값을 예측하여 이를 참고하여 모델링해 TCP가 최고의 효율을 자랑하고 싶다는 것이지요. 

Bandwidth에 대해 조금 더 얘기해봅시다. Bandwidth가 작은 Thin한 파이프를 떠올려 봅시다. Bandwidth가 큰 Fat한 파이프가 한 번에 보낼 수 있는 양을 보내기 위해 Thin한 파이프는 여러 번 데이터를 전송해야 할 것입니다. 그래서 적은 Bandwidth를 가진 파이프의 경우 Throughput은 Banwidth에 큰 영향을 받게 되지요. 큰 Bandwidth는 한 번에 보낼 데이터 양은 충분하기 때문에 이동 시간 즉 RTT가 Throughput에 더 큰 영향을 미치게 되는 것이지요. 

![pipe-band](/assets/img/network-performance/pipe_band.png){: width="70%" height="70%"}

위의 극단적인 예시로서 왼쪽은 Bandwidth가 작고 거리가 가까운 즉 Thin하고 Short한 파이프를 가지는 AP입니다. Bandwidth가 작아 패킷이 처음이 도착했는데도 계속 패킷을 보내고 있습니다. 즉, 패킷 길이가 거리보다 짧은 상황이지요. 이 때는 Bandwidth와 직결된 `Transmission Delay`가 메인 Delay로 작용하게 되는 것입니다. 오른쪽은 Bandwidth는 충분하지만 거리가 굉장히 먼 Fat하고 Long한 파이프를 가지는 위성과의 통신입니다. 이 때는 오히려 전송을 끝냈더라도 아직 패킷이 도착하지 못하는 상황이지요. 이 때는 RTT와 직결된 `Propagation Delay`가 메인 Delay로 작용하게 됩니다.

:heavy_plus_sign: 기술이 증가하면서 점점 파이프가 길어지고 뚱뚱해져 가고 있습니다. 즉, Bandwidth와 Latency가 둘 다 늘어나고 있는 것이지요.

여담을 하나 해볼까요? BDP와 직접적인 관련은 없지만요. 사람들은 데이터의 양은 많고 시간은 덜 걸리기를 소망합니다. 즉 높은 Bandwidth와 낮은 Delay를 원하는 것이지요. 그런데 Bandwidth를 올리는 일은 여간 힘든 일이 아닙니다. 비싸고 많은 물리적 링크들을 다 뜯어 고쳐야 할 테니까요. 그렇다면 Delay를 줄여야 할 텐데요. 빛의 속도는 항상 일정하니 유일한 방법은 거리르 줄이는 방법입니다. 그래서 회사들은 전 세계에 하나의 서버를 두는 것이 아니라, 곳곳에 여러 대의 서버 복사본을 배치하게 됩니다. Google에 접속하기 위해 미국의 Google 서버까지 이동할 필요 없이 구글에서 복사본을 설치한 한국 서버로 DNS가 할당함으로써 훨씬 빠른 데이터 이동 속도를 체감할 수 있게 된 거지요. 이와 같은 사업을 우리는 일종의 `Cloud` 사업이라 하는 것입니다.

## TCP Prardaise

![paradaise](/assets/img/network-performance/paradaise.png){: width="70%" height="70%"}

결론적으로 TCP가 원하는 네트워크 통신은 위와 같습니다. ACK를 받고 가능한 많은 데이터를 꽉꽉 채워서 보내고, 그 많은 데이터에 대한 ACk를 받고 또 데이터를 보내고를 반복하는 것이지요. 그렇다면 이 파이프를 비우려면 어떻게 해야할까요? Sender의 Window Size가 1이 되면 되겠지요? 그렇게 된다면 바이트를 하나만 보내고, 이 Window Size는 다음 ACK를 받아야 변경이 가능합니다.(이를 `Self-Clocking`이라 하며, 이후 **Congestion Control** 부분에서 보다 자세히 다룰 예정입니다.) 단, 하나만 보내기 때문에 전체적으로 봤을 때 ACK를 받아 변경되기 전까지 파이프가 데이터로 채워지지 않게 되는 겁니다.




