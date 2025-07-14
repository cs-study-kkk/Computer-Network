# 1. 컴퓨터네트워크 기본1

## 네트워크 구조

- **network edge**에는 랩탑, 데스크탑, 웹 서버 등이 위치
- **network core**에는 라우터들이 연결되어 있음

## Network edge

edge에 있는 컴퓨터 + core에 있는 라우터들로 이루어져 있고 이를 연결해주는 link들로 구성되어 있음

어떤 link는 유선 / 어떤 link는 무선으로 연결되어 있음

- end system (hosts)
- client/server model
  → client는 자기가 원할때 링크에 연결해서 서버로 부터 정보를 가져오는 요소
  → server는 항시(24시간) 연결되어서 client로부터 요청을 기다리는 요소
  => client와 server는 서로 데이터를 주고 받음

## 인터넷에서 제공하는 data 통신 서비스

1. **connection-oriented service (TCP가 제공)**
   TCP -> 사용자에게 (1) reliable, in-order byte-stream data transfer (2) flow control (3) congestion control을 제공
   (1)reliable, in-order byte-stream data transfer: 신뢰성 있고 순서를 지켜 데이터를 전달
   (2)flow control: receiver에 능력에 맞춰 sender의 조절 속도를 맞춰줌
   (3)congestion control: 네트워크 상황에 맞춰 받아들일 수 있는만큼 보내줌

2. **connection-oriented service(UDP가 제공)**
   connectionless, unreliable data transfer, no flow control, no congestion control
   → 아무것도 안해줌

UDP는 아무것도 안해주는데 왜 씀? => 굳이 reliable하지 않는 경우
→ (ex) 음성전화 => 실시간 오디오 패킷이 유실되도 상관 없음/모름

## protocol

: 둘 사이의 암묵적인 약속 / 이야기 하는 방식 / 대화의 약속
→ 중요한 메시지를 주고받기 위한 준비 동작
→ 서로 같은 protocol여야 소통이 가능

## Network core

중심에 라우터가 위치 -> 데이터를 목적지까지 전달

- 데이터를 라우터가 어떤 방식으로 전달?

1. **circuit switching**
   : 출발지-목적지 길을 미리 예약해두고 특정 사용자만을 위해 사용하도록 만들어 놓은 것 ex) 유선 전화망

2. **packet-switching**
   : 단순히 사용자가 보내는 packet을 받아 전송
   → 들어오는 packet 순서에 따라 보내줌

1Mb/s link가 존재, 사용자는 100 kb/s로 데이터를 보낸다고 가정
circuit switching의 경우, 최대 10명만 지원이 가능
packet-switching의 경우, 제약이 없음 / 동시에 사용자가 몰린다면 혼잡이 발생할 수 있다.
=> packet-switching이 인터넷 사용에 더 적합

## Delay / loss ( packet-switching이기 때문에 생기는 문제 )

라우터에서 packet을 받음
→ packet 검사 : **nodal processing delay** 발생
→ 들어오는 속도가 나가는 속도보다 빠름 => 임시 버퍼/Queue에 기다리는 packet들을 저장 => 대기열이 발생 : **Queuing delay** 발생
→ 임시 버퍼/Queue에서 link로 나감 : **Transmission delay** 발생 -> L(packet 길이)/R(link bandwidth)
→ Queue에 저장된 마지막 비트가 link에 올라와서 다음 라우터까지 걸리는 시간 : **propagation delay** 발생 (link의 길이 / bit의 속도 = d/s)

## delay를 줄여보자!

propagation delay: 신의 영역이라 불가
nodal processing delay: 라우터 성능을 개선
Transmission delay: 케이블 공사, bandwidth 늘려버려
Queuing delay: Queue를 조절을 못함 -> 사람들의 인터넷 사용 주기에 따라 Queue의 크기가 조정되기 때문에 조절을 할 수가 없음 => network dynamic의 근원
Queue 크기는 한정되어 있음, 크기보다 더 많이 들어오면 버림 -> packet loss가 발생

- TCP가 reliable한 data transfer를 제공해 준다 했는디 packet loss가 발생한다는데?
  → 유실된 packet을 재전송하여 해결
  → 직전 라우터가 재전송 해주거나 / 걍 다시 처음부터 재전송 해주거나 둘 중 하나를 해야 함
  → 인터넷 디자이너는 전송한 애가 처음부터 다시 재전송하게 함
  → 라우터는 단순작업만 하는 목적, 재전송까지 하면 사용 못함 너무 많은 걸 하고 있음(router == dumb core)
