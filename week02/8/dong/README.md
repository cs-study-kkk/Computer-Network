# 8. 전송계층4

## TCP congestion control (혼잡 제어)
네트워크가 과부하되지 않도록 패킷 수를 송신 측에서 전송 속도를 조절애 네트워크의 overflow를 방지하는  메커니즘

---
## TCP 혼잡 제어 동작 단계

1. Slow Start
- bottleneck bandwidth를 알지 못하니까 동일하게 처음에 패킷을 하나씩 전송
- 패킷이 문제없이 도착하면 각각의 ACK 패킷 마다 window size를 하나씩 증가 시킴, 즉 2배 씩 증가
- 윈도우 사이즈를 빠르게 두배씩 증가시키다가 혼잡 현상이 발생시 윈도우 사이즈를 1로 확 줄여버림
2. Addictive increase
- 용량이 거의 다 차게 되면 보수적으로 천천히 용량을 늘려감
3. Multiplicative decrease
- packet drop이 발생하면(혼잡이 감지되면), 급격히 송신량 감소 (혼잡 확산을 방지하려고 급격하게)

2와 3을 합쳐 AIMD 방식 => 처음에 패킷을 하나씩 보내고 문제 없이 도착하면 윈도우의 크기를 1씩 증가시켜가며 전송(Addictive increase) / 만약, 전송에 실패하면 윈도우 크기를 반으로 줄임(Multiplicative decrease)

---
## TCP Congestion Control: details
- 전송 제한 조건 : LastByteSent - LastByteAcked ≤ CongWin
=> 송신자는 아직 ACK 받지 않은 데이터 양이 Congestion Window보다 작거나 같도록 유지
=> 혼잡 윈도우 CongWin은 보낼 수 있는 최대 데이터 양을 제한하는 변수

- 전송 속도: Rate ≈ CongWin / RTT (Bytes/sec)
=> CongWin이 크면 빠르게 보내고, 작으면 느리게 보냄

- CongWin은 혼잡 상황에 따라 동적으로 변화하는데, 네트워크 혼잡을 감지하면 송신자가 CongWin을 줄여서 전송량을 감소시킴

- 혼잡 인지 방법
1. 패킷 손실 이벤트(loss event) 발생 => Timeout / 중복 ACK 3개
2. CongWin 감소 시 송신자가 혼잡으로 판단하고 전송 속도를 낮춤

- 세 가지 혼잡 제어 메커니즘
1. AIMD (Additive Increase, Multiplicative Decrease)
2. Slow Start
3. Timeout 이후 보수적 전송

---
## TCP Slow Start
- Connection 시작 시
CongWin = 1 MSS
(ex) MSS = 500 bytes & RTT = 200 msec / 초기 rate = 20 kbps
- 사용 가능한 bandwidth(대역폭) >> MSS/RTT일 수 있음 -> 빠르게 rate를 증가시키는 것이 바람직
- 연결이 시작되면 첫 번째 손실이 발생할 때까지 속도가 기하급수적으로 빠르게 증가

- 동작 원리
: 연결 시작시, CongWin은 1MSS로 시작, ACK 1개마다 CongWin += 1MSS
=> 결과적으로 RTT당 CongWin은 2배씩 증가함

- 요약: 초기 전송 속도는 느리지만 혼잡이 발생 전까지 매우 빠름

---
## Refinement (혼잡 회피로의 전환)
- 지수 증가를 멈추고 선형 증가로 바뀌는 시점?
=> CongWin이 손실 발생 직전의 절반 값에 도달했을 때 바뀜
	- 이 전환 기준값을 Threshold (ssthresh) 라고 부름
	- 혼잡 발생 시, ssthresh = CongWin / 2 로 설정

---
## TCP Tahoe vs Reno
- Tahoe: 패킷 손실 발생 시 무조건 CongWin을 1로 초기화 → 다시 Slow Start
- Reno: 중복 ACK 3개로 손실 감지 시, CongWin = ssthresh로 줄이고 선형 증가로 회복

---
## 요약
- CongWin이 임계값 이하일 때, 느린 시작 단계에 있는 발신자는 윈도우가 기하급수적으로 증가
- CongWin이 임계값을 초과하면 발신자는 혼잡 회피 단계에 있으며 윈도우는 선형적으로 증가
- 삼중 중복 ACK가 발생하면 임계값은 CongWin/2로 설정되고 Congwin은 임계값으로 설정됨
- 타임아웃이 발생하면 임계값은 CongWin/2로 설정되고 Congwin은 1 MSS로 설정됨

---
## TCP Fairness(공정성)
goal: K TCP 세션이 대역폭 R의 동일한 병목 링크를 공유하는 경우, 각 세션은 평균 R/K 비율을 가져야 함

### 왜 TCP는 공정함?
- TCP는 여러 연결이 동시에 동작할 때 네트워크 자원을 공평하게 나누는 방식으로 설계됨
- 그 핵심 원리가 바로 AIMD (Additive Increase, Multiplicative Decrease)


- Addictive Increase
: 처리량이 증가함에 따라, Additive Increase는 기울기 1을 가짐 (동일한 비율로 증가)
- Multiplicative Decrease
: 처리량을 비례적으로 감소시킴 (현재 윈도우 크기에 비례)

---
## Chapter 3 요약
- 전송 계층 서비스의 원리: multiplexing/demultiplexing, reliable data transfer, flow control, congestion control
- 인터넷에서의 인스턴스화 및 구현: UDP, TCP