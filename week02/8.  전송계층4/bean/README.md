# TCP 혼잡 제어(TCP Congestion Control)

---

## 1. 개요

TCP 혼잡 제어는 네트워크 내 혼잡 상황에서 송신자가 전송 속도를 조절하여 네트워크 과부하와 패킷 손실을 방지하는 메커니즘입니다. 혼잡 제어는 주로 송신 측에서 수행되며, 네트워크의 상태(지연, 패킷 손실 등)를 관찰하여 동적으로 동작합니다.

---

## 2. TCP 혼잡 제어의 주요 단계(3단계)

1. **Slow Start(느린 시작)**
   - 병목 구간의 대역폭(bottleneck bandwidth)을 알 수 없으므로, 전송 윈도우 크기를 1부터 시작해 빠르게(지수적으로) 증가시킴
   - 네트워크가 허용하는 범위까지 빠르게 ramp up

2. **Additive Increase(가산적 증가)**
   - 네트워크 용량에 근접했다고 판단되면, 윈도우 크기를 천천히(선형적으로) 증가시킴
   - 보수적으로 동작하여 혼잡을 예방
   - 판단 근거는 ssthresh(slow start threshold)를 미리 사전에 정해놓고 가까워지면 선형으로 변경하는 것,
     TCP는 초기에 ssthresh를 크게 잡고 시작한다. 도중에 패킷이 유실되면 절반으로 줄이고 다시 윈도으 크기를 1로, 이와 같이 작동함

3. **Multiplicative Decrease(감산적 감소)**
   - 패킷 손실(=혼잡 신호) 발생 시, 윈도우 크기를 크게(일반적으로 절반으로) 감소시킴
   - 이후 다시 Slow Start 또는 Additive Increase 단계로 진입

---

## 3. 각 단계의 동작 요약

- **Slow Start:**
  - 병목 대역폭을 모르므로 0에서 시작, 빠르게 증가
- **Additive Increase:**
  - 네트워크 용량에 가까워지면 천천히 증가(보수적)
- **Multiplicative Decrease:**
  - 패킷 손실 시 윈도우 크기 급감, 필요시 처음부터 다시 시작

---

## 4. 혼잡 제어의 원칙

- TCP는 네트워크로부터 명시적 피드백 없이, 패킷 손실/지연 등 종단에서 관찰되는 현상으로 혼잡을 추정함(End-to-End 방식)
- 혼잡 제어를 통해 네트워크의 효율성과 공정성을 높임

---

## 5. 참고 이미지 요약

- **파이프라인 비유:**
  - 데이터를 물에 비유, 네트워크를 파이프에 비유
  - 너무 많은 데이터를 한 번에 보내면(=파이프에 물을 너무 많이 붓는다면) 병목이 발생하여 손실이 생김
  - 적절한 속도로 데이터를 보내는 것이 중요

---

## 참고: TCP 혼잡 제어 3단계 요약표

| 단계                  | 동작 방식                | 목적/상황 설명                       |
|---------------------|----------------------|-----------------------------------|
| Slow Start          | 윈도우 지수적 증가         | 병목 대역폭 모름, 빠르게 ramp up      |
| Additive Increase   | 윈도우 선형 증가           | 용량 근접, 보수적으로 증가           |
| Multiplicative Decrease | 윈도우 급감(보통 1/2)   | 패킷 손실 발생, 혼잡 신호            |

---

## 6. AIMD(Additive Increase, Multiplicative Decrease)와 혼잡 윈도우(CongWin)

- **AIMD 방식:**
  - 송신자는 전송 속도(혼잡 윈도우, CongWin)를 점진적으로 증가(additive increase)시키며, 패킷 손실이 감지되면 급격히 감소(multiplicative decrease)시킴
  - 네트워크의 사용 가능한 대역폭을 탐색하며, 손실이 발생할 때까지 윈도우 크기를 1 MSS씩 증가
  - 손실 발생 시 CongWin을 절반으로 감소
  - 이로 인해 혼잡 윈도우 크기가 톱니 모양(saw tooth behavior)으로 변화

- **CongWin(혼잡 윈도우) 동작:**
  - 송신자는 `LastByteSent - LastByteAcked ≤ CongWin` 조건을 만족할 때만 데이터를 전송
  - 전송률(rate)은 대략적으로 `CongWin / RTT (Bytes/sec)`로 계산
  - CongWin은 네트워크 혼잡 상황에 따라 동적으로 변화

---

## 7. 혼잡 인지 및 반응 메커니즘

- **송신자가 혼잡을 인지하는 방법:**
  - 패킷 손실 이벤트: 타임아웃 발생 또는 중복 ACK 3회 수신
  - 손실 이벤트 발생 시 CongWin(혼잡 윈도우) 크기를 감소시켜 전송 속도를 줄임

- **혼잡 제어의 세 가지 주요 메커니즘:**
  1. AIMD (Additive Increase, Multiplicative Decrease) : 조심스럽게 올리고 낮출때는 확 낮추자,
  2. Slow Start (느린 시작) : 처음엔 조금씩 나중엔 크게 증가 (2^n)
  3. Timeout 발생 시 보수적 동작 (리셋, 임계를 절반으로 초기값도 1로 해서 다시 시작)

---

## 8. TCP Slow Start(느린 시작) 상세

- **초기 연결 시:**
  - CongWin = 1 MSS (Maximum Segment Size)
  - 예시: MSS = 500 bytes, RTT = 200ms → 초기 전송률 약 20kbps
- **가용 대역폭이 MSS/RTT보다 훨씬 클 수 있으므로, 빠르게 전송률을 높이는 것이 바람직**
- **동작 방식:**
  - 연결 시작 시, 손실 이벤트가 발생할 때까지 CongWin을 지수적으로(매 RTT마다 2배) 증가
  - 각 ACK를 받을 때마다 CongWin을 1 MSS씩 증가시킴
  - 첫 손실 이벤트 발생 시, AIMD로 전환
- **요약:**
  - 초기 전송률은 느리지만, 매우 빠르게(지수적으로) 증가하여 네트워크 용량에 근접

---

## 9. 시각적 특징 및 그래프

- **톱니 모양(saw tooth behavior):**
  - CongWin이 선형적으로 증가하다가 손실 발생 시 절반으로 감소하는 패턴 반복
  - 이는 네트워크 대역폭을 탐색(probing)하는 과정

---

## 10. 혼잡 회피(Congestion Avoidance)와 임계값(Threshold)

- **Slow Start(지수적 증가)에서 Congestion Avoidance(선형 증가)로 전환 시점:**
  - CongWin이 임계값(Threshold)의 절반에 도달하면 전환
- **임계값(Threshold) 동작:**
  - 손실 이벤트 발생 시, Threshold를 손실 직전 CongWin의 1/2로 설정
  - CongWin < Threshold: Slow Start(지수적 증가)
  - CongWin ≥ Threshold: Congestion Avoidance(선형 증가)
- **TCP Tahoe와 Reno:**
  - Reno는 손실 후 CongWin을 Threshold로 설정, 이후 선형 증가
  - Tahoe는 손실 후 CongWin을 1 MSS로 설정, Slow Start부터 재시작

---

## 11. TCP 혼잡 제어 요약

- **CongWin < Threshold:**
  - Slow Start 단계, 윈도우 크기 지수적으로 증가
- **CongWin ≥ Threshold:**
  - Congestion Avoidance 단계, 윈도우 크기 선형적으로 증가
- **Triple Duplicate ACK 발생 시:**
  - Threshold = CongWin/2, CongWin = Threshold로 설정
- **Timeout 발생 시:**
  - Threshold = CongWin/2, CongWin = 1 MSS로 설정

---

## 12. TCP의 공정성(Fairness)

- **공정성 목표:**
  - 여러 TCP 세션이 동일한 병목 링크(대역폭 R)를 공유할 때, 각 세션은 평균적으로 R/K의 전송률을 가져야 함(K: 세션 수)
- **AIMD의 공정성:**
  - Additive Increase는 각 연결의 증가 속도를 동일하게 만듦(기울기 1)
  - Multiplicative Decrease는 손실 시 각 연결의 윈도우를 비례적으로 감소시켜 공정성 유지

---

## 13. Chapter 3 요약

- 전송 계층 서비스의 원리
  - 다중화/역다중화, 신뢰성 있는 데이터 전송, 흐름 제어, 혼잡 제어
- 인터넷에서의 구현: UDP, TCP
- 다음: 네트워크의 "엣지"(응용/전송 계층)에서 "코어"(네트워크 계층)로 이동

---
