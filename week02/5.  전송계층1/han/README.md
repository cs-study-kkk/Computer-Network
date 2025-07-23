# 5.  전송계층1

실제로 Rdt 3.0(stop-and-wait 방식) 실제로 사용할 수 있냐?
신뢰적인 패킷 송수신을 위해 하나 보내고 하나 받는 방식인데
효율이 어떠한가?

ack이 올때까지 송신자은 계속 쉬고있으니 비효율.

패킷 하나를 보내는데 걸리는 시간은
L/R(첫 패킷 비트에서 마지막 비트까지 link에 올라가는 시간) + RTT(패킷이 갔다 피드백이 오는 시간)

효울이 
U(sender) = L/R / (RTT + L/R)
신뢰성은 보장하지만 효율성이 좋지 않다.

-=> 한번에 확 보내고 피드백하는 방식이 효율성이 좋다. (pipelined protocol)

일반적인 2가지 방식의 pipelined protocols: go-back-N 과 selective repeat


## Go-Back-N (GBN)


한번에 패킷을 보내는데 이때 한번에 보낼 수 있는 양을 window라고 한다.

### 특징

> cumulative ACK: 쌓는 ack (ex ACK 11: 11번을 기다리는 중 이전까지 모두 잘 받았음을 의미한다.)
- 패킷 1,2,3,4,5를 전송했는데 receiver 측에서 1,2,4,5를 받았다면, receiver는 ACK 1,2,2,2를 보낸다. => 2까지만 받음
- sender는 ACK 1,2,3이 안오고 ACK 4 만 와도 1,2,3,4까지 잘 받았음을 알 수 있다.


> 각각의 패킷마다 timer를 갇고 있다.
- timer 만료까지 ack이 안오면 window 내 ack되지 않은 패킷을 재전송(타이머가 터지면 다시 보내달라고 보낸다)
- timer 만료 전 ack이 오면 다음 ack 받지 않은 가장 최신 패킷에 timer를 계산한다.

> 리시버는 버퍼도 없고 자신이 받아야할 패킷만 기다린다.(리시버 멍청함)
- 0,2,1,3 순으로 오고 있을 때 reciever는 ack1 보내고 다 버리고 1을 기다림 다시 받기
- 구조가 간단하고 구현이 단순, 따라서 버퍼가 따로 필요가 없다.

> sender의 window size <= seq num 이어야 한다.
- 예로 window size가 4이고 seq num이 0~3일 경우 0,1,2,3을 보냈지만 ack 0,1,2,3이 모두 유실될 경우 sender에서는 0의 타이머가 터지며 다시 0을 재전송 할텐데 이때 reciever는 0이 재전송이 아닌 3 이후 오는 0이라 생각할 수 있다.


### 단점 

- 유실이 하나 되었는데 n개를 재전송해야함


## selective repeat (SR)

마찬가지로 패킷마다 timer가 있고 타이머가 터지기 전 ack을 받으면 timer 해제
receiver 에서 받은 각 패킷에 대해 ack을 보냄
리시버가 buffer가 필요
ack 3 은 3번 받았다는 의미 

### 특징
> receiver는 ack이 순서대로 오지 않아도 그냥 받은 패킷 ack 보냄

> sender는 ack을 받지 못한 패킷에 모두 타이머를 계산한다.
- 각 타이머가 만료까지 ack이 안오면 재전송

> sender에 따로 받을 버퍼가 필요하다.
- 올바르지 않은 패킷이 와도 우선 저장하고 나중에 부족한 부분이 채워지면 삼키는 구조

> 사용되는 seq num은 sender의 window size와 receiver의 window size의 합보다 크거나 같아야한다.


---
# 정리

1. 신뢰적인 통신에서 패킷을 효율적으로 송수신을 위해 pipelining을 사용
2. 일반적인 pipelining의 방식으로는 GBN,SR이 있다.
