---
layout: post
title: "[Network] TCP"
description: 안정성을 책임지는 TCP에 대해 알아보자
img: tcp_profile.png
tags: [Network]
---

이번 시간에는 4계층 Transport Layer에서 가장 많이 들리는 `TCP`에 대해서 알아보는 시간을 가지도록 하겠습니다.

# TCP의 특성

먼저, 당연하게도 TCP는 Transport Layer의 특성인 `Multiplexing`을 가집니다. 즉, 많은 Flow들이 서로 같은 링크(길)들을 공유하면서 지나가는 것이죠. 자동차 도로를 보더라도 많은 다른 자동차들이 간격만 유지한다면 다 같은 경로로 가는 것이 가능하죠? 네트워크 링크에서 패킷들도 마찬가지인 겁니다. TCP가 가지는 고유하는 특성은 `Byte-oriented`를 꼽을 수 있습니다. 패킷이 바이트 단위로 구성되있다는 것이죠. 그리고 그 바이트들은 서로를 구별하기 위해 `Label`을 가집니다. 그리고 이 바이트들이 모인 단위를 또 하나의 `Segment`라 칭합니다. `UDP`는 하나의 Message를 그 자체를 단위로 삼는 `Message-oriented(Chunk-oriented)`를 가지는 것과의 차이점이라 할 수 있습니다. 또한 TCP는 Connection의 `State`를 보유할 수 있지만, UDP에는 그런 것은 존재하지 않습니다.

![byte_stream](/assets/img/network_tcp/byte_stream.png){: width="50%" height="50%"}

위의 그림은 TCP를 사용하는 2개의 프로세스가 Conncection하는 장면을 나타낸 것입니다. 마치 서로의 프로세스가 하나의 `Tube`로 연결된 모습이죠? 이 튜브 내부에서는 수많은 `Byte Stream`이 흐르고 있으며, 이 바이트 스트림의 흐름은 하나의 `Chunk` 단위로 툭툭 쪼개질 수 있는데 그 쪼개진 단위를 `Segment`라 합니다.(패킷이라 생각해도 무방하겠죠?)

TCP는 `Error/Loss` 감지합니다. 먼저 TCP 헤더에는 Checksum이 있어서 Checksum이 Invalid한 Segment들은 조용히 버려지게 되죠. 또한 패킷을 받는 Receiver가 패킷에 Error가 없음을 확인하면 `Acknowledgment`를 보내게 되는데, Sender는 패킷을 보낸 뒤 이 Acknowledgment가 오는 시간을 측정하는 Timeout Mechanism이 있어 이 시간이 지나면 Error/Loss가 난 것으로 간주하게 됩니다. TCP에서는 이러한 Error/Loss가 감지되었을 때, 해당 패킷을 다시 전송하는 방식으로 해결하고 있는데요.

![sr_buffer](/assets/img/network_tcp/sr_buffer.png){: width="50%" height="50%"}

위와 같이 각각의 프로세스는 `Connection마다` 각자의 `Sending Buffer`와 `Receiving Buffer`을 가집니다. 그래서 보냈던 패킷을 이 Buffer에 저장해서 다시 재전송하는 것이 가능해지는 것이지요. 또한, 애초에 보내는 속도와 받는 속도가 같을 수가 없기 때문에 임시로 쌓아놓기 위해서도 결과적으로 Buffer가 필요할 수밖에 없습니다. IP에서는 줄줄 흐르는 Byte Stream의 형태로 데이터를 전송하는 것이 불가능하기 때문에, TCP가 이를 Segment 단위로 자릅니다. 그리고 TCP는 그 Segment앞에 자기 자신의 TCP 헤더를 붙여 IP에 전달하게 되고 그걸 IP가 또 자기 자신의 IP 헤더로 Encapsulation해서 전송을 하게 되는 것이지요.

이와 더불어, TCP는 송/수신이 각각 다른 독립적인 링크에서 일어나 양방향 통신이 동시에 일어날 수 있기에 `Full-duplex Service`라고 불립니다. 또한, 아까 두 개의 프로세스가 하나의 튜브로 연결되어 있는 그림을 보셨죠? 이와 같이 양쪽이 송수신 전에 확실하게 Connection을 맺기 때문에 `Connection-oriented Service`라고 불립니다. 그리고 간단히 언급하였듯이 `Acknowledgment`를 이용해 데이터의 도착이 안전하고 알맞게 일어났는 지를 보장할 수 있기에 `Reliable Service`이기도 합니다.

![byte_numbering](/assets/img/network_tcp/byte_numbering.png){: width="60%" height="60%"}

TCP는 Byte Stream이기 때문에 특이하게도 Segment가 아닌 모든 Bytes에 번호를 붙입니다. 그리고 이 번호는 방향마다 독립적이기 때문에 보내는 방향과 오는 방향의 번호가 같을 수는 있습니다. 그치만, 그 방향 내에서는 독립적인 번호를 가져야만 하는 것이지요. 이 번호는 0부터 시작할 필요는 없습니다. 그래서 처음 보허는 Random하게 0과 2^32-1 사이에서 정해지는데 이를 `Initial Sequence Number(ISN)`이라고 합니다. 그리고 TCP는 이 번호들을 Segment 단위로 잘라 첫 번째 번호를 Segment에게 부여하는데 이 번호를 `Sequence Number`라고 합니다. 위의 그림과 같이 첫 번째 Segment가 저렇다면 두 번째 Segemnt의 Sequence Number는 마지막 번호에 1을 더한 11010이 되겠지요?:wink: 

또한, 이와 같은 패킷을 받은 Receiver는 Error 없이 잘 받았다는 것을 Sender한테 전달해주기 위해 패킷에 Sequence Number가 아닌 `Acknowledgment Number`를 붙여 보냅니다. 방금 이 11010이 Acknowledgment Number가 될 수 있습니다. 받은 Segment의 마지막 번호에 1을 붙여 이 번호를 기대하겠다는 의사를 표하는 거지요. 이는 또한 `Cumulative`하다는 특징을 가집니다. 간단히 설명해서 여러 개의 패킷이 와도 그것들을 모두 취합에 가장 마지막 번호에 1을 더해 하나의 Acknowledgment를 보낼 수 있는 것이지요.

TCP 헤더는 아래와 같이 이루어집니다.

![tcp_header](/assets/img/network_tcp/tcp_header.png){: width="60%" height="60%"}

전에도 NAT 등을 설명할 때도 Port 번호는 4계층임을 명시한 바 있었죠? 이렇듯이 TCP에는 가장 먼저 `Source Port Address`와 `Destination Port Address`가 있습니다. 또한 방금 설명한 `Sequence Number`와 `Acknowledgment Number`가 있습니다. 보내는 패킷과 오는 패킷이 서로 다른 TCP 헤더를 쓸 일은 없겠죠? 그래서 둘 다 있습니다. 그리고 `HLEN`이 헤더 길이를 의미합니다. 맨 밑에 보시면 `Options` 영역이 있는데 TCP는 Option을 많이 쓰거든요ㅎㅎ 

Flag들을 살펴 볼까요? 
- URG(Urgent pointer is valid) : 긴급히(우선순위를 높게) 전송해야 함을 의미합니다.
- ACK(Acknowledgment is valid) : Ack임을 의미합니다.(뒤에서 설명)
- ASH(Request for push) : Buffer를 쓰지 않고 즉시 보내고 받아야 함을 의미합니다.
- RST(Reset the connection) : 비정상적인 이유로 연결을 리셋해야 함을 의미합니다.
- SYN(Synchorinize sequence numbers) : SYN임을 의미합니다.(뒤에서 설명)
- FIN(Finish the connection) : FIN임을 의미합니다.(뒤에서 설명)

'Urgent Pointer'는 urg 플래그와 연결되는 개념으로 그 급한 정도를 나타냅니다. `Checksum`은 우리가 아는 그 Checksum의 개념이 맞구요. 특이한 건 `Window Size`인데 이것은 Receiver가 Sender에게 수용 가능한 데이터 양을 알려주기 위해 존재합니다. 즉 현재 자신의 Available한 Buffer Size를 나타내는 것이지요. 다음 포스트의 TCP가 제공하는 `Flow Control`에서 보다 자세히 아실 수 있을 것입니다.

# TCP의 Connection

TCP가 처음에 데이터를 송수신 하기 위해서는 상대방과 Connection을 성립하는, 즉 `Establishment`하는 과정이 필요합니다. 3번의 송수신이 일어나기 때문에 이를 `Three-way Handshake`라 칭합니다.

![three_way_handshake](/assets/img/network_tcp/three_way_handshake.png){: width="40%" height="40%"}

클라이언트가 먼저 서버에게 동기화를 요청하는 1 바이트의 `SYN` Segment를 보냅니다. 이때 클라이언트에서 서버 방향으로의 첫 패킷이므로 Random하게 정해진 ISN이 Sequence Number가 됩니다.(위의 그림에서는 111이겠네요.) 그리고 Connection에 있어서 가장 첫 패킷이기 때문에 Acknowledgment가 있을 수 없기 때문에 Acknowledgment Number는 0이 됩니다. 그럼 이제 서버는 그 Segment를 받은 뒤 들어온 SYN에 대해 ACK를 하고 다시 클라이언트에게도 SYN을 요청하기 위해 `SYN,ACK`를 하나의 Segment로 보내게 됩니다. 이 때는 서버에서 클라이언트로의 첫 패킷이기 때문에 ISN이 마찬가지로 Sequence Number가 됩니다. 그리고 Acknowledgment는 들어온 SYN이 1바이트 였기 때문에 SYN의 Sequence Number에 1을 더한 값이 됩니다.(위의 그림에서 그래서 112가 되는 것이지요.) 그렇게 되면 최종적으로 클라이언트의 `ACK`가 남게 되는데, 이 때 Sequence Number는 그 전 SYN의 Sequence Number의 1을 더한 값이자 SYN,ACK의 Acknowledgment Number이기도 한 값이 될 겁니다. 또한 이 때의 Acknowledgment Number은 바로 전 서버에서 온 패킷의 Sequence Number에 1을 더한 값이 되는 것이지요. (SYN,ACK도 마찬가지로 다른 특별한 의미가 없는 1 바이트이기 때문입니다.)

> 이제 편의상 Sequence Number를 `Seq`으로, Acknowledgment Number를 `Ack`로 표현하도록 하겠습니다. 

이제 Connection을 끊을 때를 알아 볼까요? 이 때는 4번의 송수신이 일어나기 때문에 `Four-way Handshake`라 칭합니다.

![four_way_handshake](/assets/img/network_tcp/four_way_handshake.png){: width="40%" height="40%"}

클라언트가 먼저 너한테 보내지 않겠다는 `FIN,ACK` Segment를 보냅니다. 그럼 그걸 받은 서버도 클라이언트에게 'ACK' Segment를 보내줍니다. 그러나 서버에서 클라이언트로의 Connection은 종료되지 않았기 때문에 서버는 계속해서 클라이언트에게 데이터를 보낼 수 있는 상태입니다. 그 후에 더 이상 보낼 데이터가 없으면 서버도 클라이언트에게 `FIN,ACK` Segment를 보내고, 그걸 받은 클라이언트도 잘 받았다는 `ACK` Segmentf르 보내는 것입니다. 여기서 특이한 점은 서버가 보내는 2번 ACK와 3번 FIN,ACK Segment는 같은 방향으로 연속해서 보내는 것이기 때문에 내용을 Copy해서 Seq와 Ack가 같게 됩니다.

이렇게 되면 서버도 클라이언트에게 데이터를 보낼 수 없는 완전한 Connection Termination이 완성됩니다. 이처럼, Conncection의 Establishment는 동시에 일어나야 하지만 Termination은 꼭 그렇지 않습니다. 누구라도 먼저 끊어버릴 수 있는 것이며 이를 용어로 `Half-Close`로 합니다. 먼저 FIN을 요청하는 것을 `Active Close`라 하며, 나중에 요청하는 것은 `Passive Close`라고 합니다. 직관적이죠?

여기서 알아야 할 점은 Active Close를 한 대상은 MSL, 즉 약 4분동안 동일한 Port 번호를 사용하지 못합니다!:tired_face: 저번에 Connection을 구분하는 5 tuples에 대해 설명한 바 있지요? 만약 똑같은 Port 번호를 MSL 이내로 똑같은 상대와 Connection을 맺는다면, 모든 5 tuples가 같아 이 패킷이 저번 Connection인지 이번 Connection인지 구분이 안가는 거지요. 그래서 서버는 Active Close가 되어서는 안되는 것입니다. NAT의 Port Forwarding에서 공부했듯이 클라이언트야 Port 번호가 그때그때 Random하게 정해지지만 서버는 단 하나여야 되기 때문이죠. 서버가 기다려서는 안되겠죠? 그래서 Client가 항상 먼저 FIN을 날려야 되기 때문에 Server는 Application에서 클라이언트(브라우저)에게 Close를 해달라고 요청해야 하는 것이죠.

### Connection Resetting

추가적으로 Connection이 `Reset` 되어야 하는 경우는 아래와 같습니다.

- 한쪽 TCP에서 존재하지 않는 Port 번호로 Connection을 요청하였을 경우
- 한쪽 TCP에서 비정상적인 상황으로 인해 Connection을 죽이려 할 경우(ex. 브라우저 광클)
- 한쪽 TCP에서 상대 TCP가 오랜 기간 동안 idle 상태임을 알았을 경우

### TCP VS UDP

TCP와 UDP의 차이에 대해서는 아래와 같습니다. UDP는 나중에 포스트로 자세히 다루겠지만 먼저, TCP와 UDP에 대한 감을 잡아보도록 합시다. 다음 시간에는 TCP의 중요한 기능인 여러 `Control`에 대해 집중적으로 알아보도록 하겠습니다.

![tcp_vs_udp1](/assets/img/network_tcp/tcp_vs_udp1.png){: width="80%" height="80%"}

![tcp_vs_udp2](/assets/img/network_tcp/tcp_vs_udp2.png){: width="80%" height="80%"}


