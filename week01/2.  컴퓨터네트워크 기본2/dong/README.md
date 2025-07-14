# 2. 컴퓨터네트워크 기본2

## 네트워크 계층

| 계층 | 이름        | 예시                 |
| ---- | ----------- | -------------------- |
| 5    | Application | HTTP                 |
| 4    | Transport   | TCP, UDP             |
| 3    | Network     | IP                   |
| 2    | Link        | Wi-Fi, LTE, Ethernet |
| 1    | Physical    | (물리 계층)          |

application에는 process가 존재, 중간의 네트워크가 어떻게 구성되는지 관심 없다

라우터에는 Physical ~ Network Layer까지 존재

## Client - Server architecture

### server

- 24시간 동작해야 함
- "고정된 IP 주소를 가져야 함" => 자기의 주소를 가져야 한다

### client

- 서버와 communicate
- 동적 IP주소를 가질 수 있음
- 서로 직접 통신 X

다른 컴퓨터상에 위치한 프로세스와 프로세스 간의 통신(inter-process communication)을 위해 OS가 만들어놓은 interface → **socket**

## Sockets

프로세스를 찾기 위해 Socket의 주소의 역할을 하는 인덱싱이 필요 → **IP address**와 **Port Number**를 사용

- **IP 주소** => 어떤 컴퓨터인지 알려줌
- **Port #** => 특정/어떤 프로세스를 인지를 지칭해줌

거의 모든 웹 서버는 Port Number는 주로 80번을 사용 (아마존이나 네이버나 구글이나..)
why? 공통된 80번을 사용하는 이유는 서버는 24시간 열려있어야 됨 + 주소가 일정해야 함 → 주소가 각각 다 틀림 + Port #까지 틀리면 복잡해짐 => 이러한 이유로 port #는 통일되게 사용

하위계층에서 상위계층에게 서비스를 제공

### Transport Layer에서 제공해줬으면 좋을 서비스?

1. 데이터 무결성 : 보내는 데이터가 유실되지 않고 목적지까지 전달
2. 타이밍: 보내는 데이터가 원하는 시간에 도착
3. throughput: 보내는 데이터의 양 (초당)
4. 보안: 보내는 데이터의 보안
   → 데이터 무결성만 제공, TCP가 (UDP는 안해요)

## HTTP

: hypertext를 전달하는 protocol (HyperText Transfer Protocol)
※ hypertext: link가 존재하는 text

request <-> response (메시지 교환)

TCP를 사용 => TCP Connection을 해줘야 함 (메시지 교환 이전에)

HTTP는 stateless, 상태가 없음 => 정말 단순해서 request가 들어오면 보내고 상태를 저장하지 않음

## HTTP connections

TCP connection 사용에 따라 2가지로 나뉨

- non-persistent HTTP (지속적X)
  : TCP connection을 끊음 -> 한 object를 보낼때마다 TCP연결이 필요
- persistent HTTP (지속적)
  : TCP connection을 끊지 않고 유지하고 재사용 -> 한번으로 object를 계속 보냄
