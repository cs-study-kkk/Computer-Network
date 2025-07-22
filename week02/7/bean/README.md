# Transport Layer - Chapter 3: Flow Control & Connection Management

---

## 3장 개요(Outline)

3.1 전송 계층 서비스(Transport-layer services)
3.2 다중화와 역다중화(Multiplexing and demultiplexing)
3.3 비연결형 전송: UDP(Connectionless transport: UDP)
3.4 신뢰성 있는 데이터 전송 원칙(Principles of reliable data transfer)
3.5 연결형 전송: TCP(Connection-oriented transport: TCP)
  - 세그먼트 구조(segment structure)
  - 신뢰성 있는 데이터 전송(reliable data transfer)
  - 흐름 제어(flow control)
  - 연결 관리(connection management)
3.6 혼잡 제어 원칙(Principles of congestion control)
3.7 TCP 혼잡 제어(TCP congestion control)

---

## TCP 흐름 제어(Flow Control)

- **흐름 제어(Flow control):**
  - 송신자가 너무 빠르게 데이터를 보내서 수신자의 버퍼가 넘치지 않도록 조절하는 메커니즘
  - "송신자가 너무 많은 데이터를 너무 빨리 보내 수신자 버퍼가 오버플로우되는 것을 방지"

- **TCP 연결의 수신 측(receive side)은 수신 버퍼(receive buffer)를 가짐**
  - RevBuffer: 전체 수신 버퍼 크기
  - RevWindow: 현재 비어 있는(여유) 공간
  - 수신 버퍼에 데이터가 쌓이고, 애플리케이션이 읽어가며 비워짐
  - 애플리케이션이 버퍼에서 데이터를 읽는 속도가 느릴 수 있음

- **속도 매칭 서비스(speed-matching service):**
  - 송신 속도를 수신 애플리케이션의 데이터 처리 속도에 맞춤

---

## TCP 흐름 제어 동작 방식

- **수신자(Receiver)는 RevWindow(여유 공간) 값을 세그먼트에 포함해 송신자에게 알림**
- **송신자(Sender)는 RevWindow만큼만 미확인(ACK되지 않은) 데이터를 전송**
  - 이를 통해 수신 버퍼 오버플로우를 방지

- **수신 버퍼의 여유 공간 계산:**
  - RevWindow = RevBuffer - (LastByteRcvd - LastByteRead)
    - LastByteRcvd: 마지막으로 수신한 바이트 번호
    - LastByteRead: 애플리케이션이 마지막으로 읽은 바이트 번호

- **송신자는 RevWindow를 초과하지 않도록 데이터 전송량을 제한**
  - 보장: 수신 버퍼가 절대 넘치지 않음

---

## 용어 정리

- **RevBuffer:** 수신 버퍼 전체 크기
- **RevWindow:** 현재 수신 버퍼의 여유 공간(송신자가 전송 가능한 최대 데이터 양)
- **Flow control:** 송신자가 수신자의 처리 속도에 맞춰 전송 속도를 조절하는 기능

---

## 연결 관리(간단 개념)
- TCP는 연결 설정, 데이터 전송, 연결 해제의 과정을 관리함

---

## TCP 연결 관리(Connection Management)

### 연결 설정: 3-way handshake
- 데이터 교환 전, 송신자와 수신자가 "연결"을 먼저 설정
- TCP 변수 초기화: 시퀀스 번호, 버퍼, 흐름 제어 정보(예: RcvWindow)
- **클라이언트(client):** 연결 시작자
  - 예시: `Socket clientSocket = new Socket("hostname", "port number");`
- **서버(server):** 클라이언트의 연결 요청을 수락
  - 예시: `Socket connectionSocket = welcomeSocket.accept();`

#### 3-way handshake 과정
1. **Step 1:** 클라이언트가 서버로 SYN 세그먼트 전송(초기 시퀀스 번호 포함, 데이터 없음)
2. **Step 2:** 서버가 SYN 수신 후, SYNACK 세그먼트로 응답(버퍼 할당, 서버 초기 시퀀스 번호 포함)
3. **Step 3:** 클라이언트가 SYNACK 수신 후, ACK 세그먼트로 응답(데이터 포함 가능)

- 각 단계에서 시퀀스 번호와 ACK 번호가 교환됨
- SYN, SYNACK, ACK 플래그 사용

---

## TCP 연결 종료(Closing a TCP Connection)

### 연결 종료 과정(4-way handshake)
1. **Step 1:** 클라이언트가 FIN 세그먼트 전송(연결 종료 요청)
2. **Step 2:** 서버가 FIN 수신 후, ACK로 응답, 연결 종료 준비 후 FIN 전송
3. **Step 3:** 클라이언트가 FIN 수신 후, ACK로 응답(이후 timed wait 상태)
4. **Step 4:** 서버가 ACK 수신 후, 연결 완전히 종료

- FIN, ACK 플래그 사용
- 클라이언트는 timed wait 상태에서 일정 시간 대기(재전송 대비)

---

## TCP 상태 전이와 라이프사이클

### 클라이언트 라이프사이클
- CLOSED → SYN_SENT → ESTABLISHED → FIN_WAIT_1 → FIN_WAIT_2 → TIME_WAIT → CLOSED
- 각 상태에서 SYN, ACK, FIN 세그먼트 송수신에 따라 전이

### 서버 라이프사이클
- CLOSED → LISTEN → SYN_RCVD → ESTABLISHED → CLOSE_WAIT → LAST_ACK → CLOSED
- 서버는 LISTEN 상태에서 연결 요청 대기, 이후 클라이언트와의 세그먼트 교환에 따라 상태 전이

--- 

## 혼잡 제어의 원칙(Principles of Congestion Control)

- 네트워크 혼잡 제어는 송신자가 네트워크의 혼잡 상태에 따라 전송 속도를 조절하는 메커니즘
- TCP는 혼잡 제어를 통해 네트워크 과부하를 방지함

---

## 혼잡 제어 접근 방식(Approaches towards Congestion Control)

### 1. End-end(종단 간) 혼잡 제어
- 네트워크로부터 명시적인 피드백 없음
- 송신자가 패킷 손실, 지연 등 종단 시스템에서 관찰되는 현상으로 혼잡을 추정
- TCP가 사용하는 방식

### 2. Network-assisted(네트워크 지원) 혼잡 제어
- 라우터가 종단 시스템에 피드백 제공
  - 예: 혼잡 상태를 나타내는 비트(SNA, DECbit, TCP/IP ECN, ATM 등)
  - 송신 속도를 명시적으로 지정

--- 