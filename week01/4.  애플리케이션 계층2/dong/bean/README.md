# 전송 계층 (Transport Layer) 핵심 정리

---

1. **Transport-layer services**: 전송 계층 서비스
2. **Multiplexing and demultiplexing**: 멀티플렉싱과 디멀티플렉싱
3. **Connectionless transport: UDP**: 비연결 전송: UDP
4. **Principles of reliable data transfer**: 신뢰성 있는 데이터 전송 원칙
5. **Connection-oriented transport: TCP**: 연결 지향 전송: TCP
   - segment structure: 세그먼트 구조
   - reliable data transfer: 신뢰성 있는 데이터 전송
   - flow control: 흐름 제어
   - connection management: 연결 관리
6. **Principles of congestion control**: 혼잡 제어 원칙
7. **TCP congestion control**: TCP 혼잡 제어

---

## 신뢰성 있는 데이터 전송 원칙

### 왜 신뢰성 있는 데이터 전송 프로토콜이 필요한가?

**신뢰할 수 없는 채널에서 발생할 수 있는 문제:**
- **메시지 오류**: 비트 오류
- **메시지 손실**: 패킷 손실

### 신뢰성 있는 데이터 전송 프로토콜 구성 요소

#### 1. **오류 처리 메커니즘**
- **오류 검출**: 체크섬 비트 추가
- **피드백**: ACK(긍정적 확인), NAK(부정적 확인)
- **재전송**: NAK 수신 시 패킷 재전송

#### 2. **시퀀스 번호**
- 중복 패킷 구분
- 순서 보장

#### 3. **타이머**
- 패킷 손실 처리
- 타임아웃 시 재전송

### RDT 프로토콜 발전 과정

#### RDT1.0: 완벽한 채널
- 오류 없음, 손실 없음
- 추가 메커니즘 불필요

#### RDT2.0: 패킷 오류 (손실 없음)
- 오류 검출, 피드백, 재전송
- ACK/NAK 오류 시 중복 패킷 문제

#### RDT2.1: 시퀀스 번호 추가
- 중복 패킷 구분
- ACK/NAK 오류 처리

#### RDT3.0: 손실과 오류 모두
- **타이머 추가**
- 타임아웃 시 재전송

### 성능 문제: Stop-and-Wait의 한계

**예시**: 1Gbps 링크, 15ms 지연, 1KB 패킷
- 전송 시간: 8 마이크로초
- RTT: 30ms
- **이용률**: 0.00027 (매우 낮음)

### 해결책: 파이프라이닝

#### 파이프라이닝의 장점
- 여러 패킷을 동시에 전송
- 시퀀스 번호 범위 확장
- 송신자/수신자 버퍼링

#### 파이프라이닝 프로토콜 종류
1. **Go-Back-N**: 손실 시 N개 패킷 재전송
2. **Selective Repeat**: 손실된 패킷만 재전송
