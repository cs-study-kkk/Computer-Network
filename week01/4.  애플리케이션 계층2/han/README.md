# 애플리케이션 계층 2

multiplexing 과 에러 check를 transport 계층해서 해줘야함
(UCP가 아무것도 안하는것 같지만 이 2가지를 해준다.)

TCP는 reliable한 통신을 보장해준다.

reliable이란 application에서 내려온 message가 유실되지 않고 다른 host로 전달되는 것

사실은 transport가 reliable한 서비스를 제공하는 것 같지만 언더 라인 환경 라우터에서는 큐가 가득차 유실되는 에러가 발생 가능 

unreliable한 상황은 Message 에러와 유실 이것만 지켜주면 reliable함

# Rdt (Reliable Data Transfer) Protocol

## Rdt 1.0

- 가정: 오류 없음, 손실 없음 (no packet errors,n o packet loss)
- 송신자는 단순히 데이터를 전송
- 수신자는 데이터를 수신 후 전달

> 가능한가? 비현실적인 상황, 실제 환경에서는 오류 및 유실 발생 가능

---

## Rdt 2.0

error가 없으려면

- error detecion 에러 있는지 확인 -> checksum bit

- feedback : reciever가 받았는데 에러가 있으면 재전송을 요청한다. (Ack, Nak)

- 재전송

### 문제점



- 수신자가 데이터를 **정상 수신**하고 `ACK`를 전송했지만,
  - `ACK`가 **유실**되면 송신자는 수신 성공 여부를 알 수 없음
- 송신자는 **재전송**을 선택하게 됨
- 수신자는 동일한 패킷을 두 번 받을 수 있음
  - 이 경우, **중복인지 새로운 패킷인지** 구분이 필요

### 해결 방법

1. **피드백에도 오류 검출 추가**
   - `ACK` 또는 `NAK`에도 `Checksum`을 포함시켜 유효성 검증

2. **패킷에 Sequence Number 추가**
   - 송신자는 각 패킷에 고유한 번호(`seq num`)를 부여
   - 수신자는 `seq num`을 기준으로 중복 여부 판단
     - 동일한 `seq num`이면 **중복된 패킷**
     - 새로운 `seq num`이면 **신규 패킷**

### 부가 고려 사항

- `seq num`의 값 범위가 커질수록 **헤더 크기(Overhead)** 증가
  - 예: 1-bit `seq num` → 중복 처리 한계 존재
  - 필요 시 순환적 번호 재사용 (modulo 사용)

---

## Rdt 2.1 / 2.2

- **Sequence Number 도입**
  - 중복 패킷 여부를 구분하기 위해 각 패킷에 번호 부여
- **피드백 개선**
  - 2.1: ACK/NAK 모두 사용
  - 2.2: NAK 제거, ACK에 마지막으로 받은 seq 번호 포함

---

## Rdt 3.0

- 패킷 유실(loss) 가능성까지 고려
- 송신자 측에서 Timeout 기능 추가
  - 일정 시간 안에 ACK 수신 못하면 재전송
- 수신자는 동일한 패킷이 들어오면 중복 처리


### 송수신 과정

1. sender: `seq num`, `checksum`, `data`를 담은 패킷 전송
2. receiver:
   - 오류 없음: ACK 전송
   - 오류 있거나 중복: 기존 ACK 재전송
3. sender:
   - ACK 수신: 다음 패킷 전송
   - ACK 미수신 (timeout): 해당 패킷 재전송
