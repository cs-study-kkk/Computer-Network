# 7.  전송계층3

## Flow Control 이란?

TCP 연결이 생성되면 각 커넥션마다 **2개의 버퍼(send, receive)** 가 생성된다.  
Flow control은 수신 측의 수신 버퍼(receive buffer)가 수용 가능한 만큼만 송신 측이 데이터를 보내도록 조절하는 기능이다.

- 수신 버퍼의 가용 공간(available space)은 **TCP 헤더의 window size 필드**를 통해 송신자에게 전달된다.
- 송신자는 이 값을 기준으로 전송 가능한 데이터의 양을 조절한다.


### Flow Control는 속도를 조절할까? 보내는 양을 조절할까?

- 단위 시간당 전송량이 많으면 속도도 많아진다.
- 결국 **속도와 전송량은 같은 개념**이라고 볼 수 있다.

예시:  
10M bit/sec = 10메가비트의 데이터를 1초에 보내는 것 → 단위 시간당 전송량

<br>

## connection management(TCP 3-way handshake)

TCP는 연결형 프로토콜이며, **신뢰성 있는 데이터 전송을 위해 연결을 설정하는 과정**이 필요함.
이를 3-way handshake 과정으로 충족한다.

### 연결 설정 과정

client가 server에 tcp 연결을 요청한다

1. **Client → Server (SYN)**
   - 데이터 부분에 아무것도 없이 SYN bit = 1, 임의의 초기 Sequence Number를 포함한 세그먼트 전송 (SYN msg)
2. **Server → Client (SYN + ACK)**
   - SYN bit = 1, ACK bit = 1로 설정된 세그먼트 전송
   - 클라이언트의 시퀀스 번호 + 1에 대한 ACK 포함
3. **Client → Server (ACK)**
   - ACK bit = 1, 서버의 시퀀스 번호 + 1에 대한 ACK 전송
   - 연결 성립


<br>

### 3-way handshake하는 이유

- 클라이언트와 서버 **양쪽이 서로의 연결 의사를 명확히 확인**할 수 있도록 하기 위함
- 중간에서 지연되거나 유실된 이전 연결 요청 메시지가 잘못 해석되는 것을 방지


<br>

### 연결 끊기

1. **FIN 플래그를 설정(fin bit=1)**으로 데이터 없이 연결 종료 요청 전송
2. 상대방은 아직 보낼 데이터가 있다면 계속 보내고, **모두 전송 후 FIN 전송**
3. FIN을 받은 쪽은 ACK 응답
4. 마지막 ACK를 보낸 쪽은 **TIME-WAIT 상태로 일정 시간 대기** 후 CLOSED 상태로 전이


클라이언트는 fin을 받고 ack을 보낸 후 일정 timed wait를 기다리고 closed를 한다.

왜? 
> 만약 클라이언트의 마지막 ack가 유실되면 서버는 타임아웃되어 다시 보냄 그래서 ack이 서버에 정말 전송 되때까지 기다려 주는 것.

**연결이 안전하게 종료되었음을 보장**

(timeout value는 향상 변하는 값)



<br>

## congestion control

sender의 속도는 net(네트워크 상태)와 recv(수신자 상태) 중 min(network, recv)에 맞춰서 보내야한다.

recv 상태는 수신 버퍼를 통해 파악할 수 있는데 network는 어떻게 알 수 있을까? 

congestion control로 알 수 있다. (다음 강의로)




