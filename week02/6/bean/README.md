---


## TCP: 개요(Overview)

- **점대점(point-to-point):** 한 송신자, 한 수신자
- **신뢰성 있는, 순서가 보장된 바이트 스트림(reliable, in-order byte stream)**
  - 관련 RFC: 793, 1122, 1323, 2018, 2581
- **Full duplex data:** 양방향 데이터 흐름(동일 연결 내)
- **MSS(Maximum Segment Size):** 최대 세그먼트 크기
- **메시지 경계 없음(no "message boundaries")**
- **연결 지향(connection-oriented):**
  - 파이프라이닝(pipelined): 송수신 버퍼 사용
  - 혼잡/흐름 제어: 윈도우 크기 조절
  - 소켓(socket) 인터페이스
  - 핸드셰이킹(handshaking): 데이터 교환 전 제어 메시지로 상태 초기화
  - 흐름 제어(flow controlled): 송신자가 수신자를 압도하지 않도록 조절

---

## TCP 세그먼트 구조(segment structure)

- 32비트 헤더 구조
  - **Source port #, Destination port #**: 송수신 포트 번호
  - **Sequence number**: 데이터 바이트의 첫 번째 바이트 번호
  - **Acknowledgement number**: 다음에 기대하는 바이트 번호(ACK)
  - **헤더 길이(head len)**
  - **플래그(URG, ACK, PSH, RST, SYN, FIN)**: 연결 설정/해제, 데이터 전송 등 제어
  - **Receive window**: 수신자가 수용 가능한 바이트 수
  - **Checksum**: 오류 검출(UDP와 동일)
  - **Urgent data pointer, Options**: 특수 기능(일반적으로 잘 사용하지 않음)
  - **Application data**: 실제 데이터(가변 길이)
- 데이터는 바이트 단위로 카운팅(세그먼트 단위가 아님)

---

## TCP 시퀀스 번호와 ACK(Sequence numbers and ACKs)

- **시퀀스 번호(Seq. #):** 세그먼트 데이터의 첫 번째 바이트의 바이트 스트림 번호
- **ACK 번호:** 상대방이 다음에 기대하는 바이트 번호(누적 ACK)
- **예시:**
  - Host A가 'C'를 전송(Seq=42, ACK=79, data='C')
  - Host B가 'C'를 수신하고, ACK(Seq=79, ACK=43, data='C')
  - 누적 ACK 방식 사용
- **순서가 맞지 않는 세그먼트 처리:** TCP 표준은 구현자에게 맡김(필수 동작 아님)
- **Telnet 시나리오:** 바이트 단위로 송수신, 각 바이트마다 시퀀스/ACK 번호 갱신

---

## TCP의 타임아웃과 RTT 추정

### 타임아웃(Timeout) 설정 원칙
- 타임아웃 값은 RTT(Round Trip Time)보다 길어야 함
- RTT가 변동하므로, 너무 짧으면 불필요한 재전송(조기 타임아웃), 너무 길면 손실 반응이 느림

### RTT(왕복 시간) 추정 방법
- **SampleRTT:** 세그먼트 전송~ACK 수신까지의 실제 측정 시간
- SampleRTT는 매번 다르므로, 최근 여러 측정값을 평균하여 부드럽게(평활화) 추정
- **지수 가중 이동 평균(Exponential Weighted Moving Average):**
  - EstimatedRTT = (1-α) * EstimatedRTT + α * SampleRTT
  - 과거 샘플의 영향이 지수적으로 감소
  - 일반적으로 α = 0.125 사용

### 타임아웃 간격(TimeoutInterval) 계산
- **DevRTT:** SampleRTT가 EstimatedRTT에서 얼마나 벗어나는지(편차) 측정
  - DevRTT = (1-β) * DevRTT + β * |SampleRTT - EstimatedRTT|
  - 일반적으로 β = 0.25 사용
- **TimeoutInterval = EstimatedRTT + 4 * DevRTT**
  - 변동성이 크면 더 넉넉한 마진을 둠

---

## TCP의 신뢰성 있는 데이터 전송(rdt)

- TCP는 IP의 비신뢰성 위에 신뢰성 있는 데이터 전송(rdt) 서비스를 구현
- **파이프라이닝(Pipelined segments):** 여러 세그먼트를 동시에 전송
- **누적 ACK(Cumulative ACK):** 여러 바이트를 한 번에 확인
- **단일 재전송 타이머(single retransmission timer) 사용**
- **재전송 트리거:**
  - 타임아웃 발생
  - 중복 ACK 수신
- (단순화된 TCP 송신자 기준) 중복 ACK, 흐름 제어, 혼잡 제어는 무시하고 동작 설명 가능

--- 

## TCP 송신자 이벤트와 동작

### 주요 이벤트
- **애플리케이션에서 데이터 수신(data received from app):**
  - 세그먼트 생성(시퀀스 번호: 해당 세그먼트의 첫 바이트 번호)
  - 타이머가 동작 중이 아니면 시작(가장 오래된 미확인 세그먼트 기준)
  - 세그먼트 전송(IP 계층에 전달)
  - NextSeqNum 증가(보낸 데이터 길이만큼)

- **타임아웃(timeout):**
  - 타임아웃이 발생한(가장 작은 시퀀스 번호의) 미확인 세그먼트 재전송
  - 타이머 재시작

- **ACK 수신(Ack received):**
  - 이전에 미확인된 바이트가 확인되면(ACK 번호 > SendBase)
    - SendBase 갱신(새로운 누적 ACK)
    - 아직 미확인 세그먼트가 남아 있으면 타이머 재시작

### 간단한 TCP 송신자 알고리즘
- NextSeqNum = InitialSeqNum
- SendBase = InitialSeqNum
- 무한 루프 내에서 이벤트에 따라 동작
  - 데이터 수신 시: 세그먼트 생성, 타이머 관리, 전송, NextSeqNum 증가
  - 타임아웃 시: 가장 오래된 미확인 세그먼트 재전송, 타이머 재시작
  - ACK 수신 시: SendBase 갱신, 미확인 세그먼트 있으면 타이머 재시작
- SendBase-1: 마지막으로 누적 ACK된 바이트

---

## TCP 재전송 시나리오

- **ACK 손실(lost ACK):**
  - 송신자가 세그먼트를 재전송하지만, 수신자는 이미 데이터를 받았으므로 중복 ACK만 전송
  - SendBase는 ACK가 도착할 때까지 갱신되지 않음

- **조기 타임아웃(premature timeout):**
  - 실제로 세그먼트가 손실되지 않았지만, 타임아웃이 너무 짧아 불필요한 재전송 발생

- **누적 ACK(cumulative ACK) 시나리오:**
  - 여러 세그먼트가 전송되고, 일부 ACK가 손실되어도 이후 ACK가 누적되어 모든 데이터가 확인됨

--- 
