# 8.  전송계층4

## TCP Congestion Control

- 네트워크 내부는 직접적인 피드백을 안해준다. 
- 송신자와 수신자의 피드백을 기반으로 네트워크 상황을 간접적으로 유추해야한다.

처음부터 1개씩 전송하면 신중하지만 어느 세월에 하는가

## 3 main phases
1. slow start (지수적 증가로 올린다.)
- 작은 값부터 시작
- 다만 증가하는 것은 이름과 달리 지수 배로 증가한다.
2. additive increase (임계값 근처로 가면 조심해야한다. -> 선형으로 증가)
- 천천히 속도를 높임
3. multiplicative decrease (어느 순간이 되었을때 패킷 유실이 일어나면 감소시킴)
- 네트워크 상황이 안 좋으면 패킷 유실이 발생
- 패킷 유실이 발생하면 데이터 전송량을 1/2배로 줄인다.

왜 늘릴때는 선형으로 줄일때는 1/2..이렇게 줄이냐.

막혔을때는 1,2개 줄인다고 풀리지 않음 확 줄여야 풀림



## MSS (Maximum Segment Size)

혼잡 제어(Congestion control)에서 전송하는 데이터량은 MSS를 단위로 한다

> MSS은 TCP 세그먼트 하나가 가질 수 있는 최대 데이터 크기(Byte)


## 혼잡 윈도우(Congestion Window) 변화 패턴

- Slow start에서는 무조건 윈도우 사이즈가 1 MSS로 시작하며 1, 2, 4, 8 MSS 순으로 증가시킨다.

- TCP에서는 데이터 전송량이 증가하다가 패킷 유실이 발생하면 Congestion window size가 절반으로 감소시킨다.

- 줄어들고 나면 다시 전송량이 증가하고, 다시 절반으로 줄어들기를 반복해서 톱날 모양으로 속도가 변화한다.

네트워크의 임계점에서 전송하는 속도만큼 계속 전송하는 것이 가장 좋지만,
 임계 지점은 매번 바뀌므로 불가능하다.

 따라서 파일 전송 시에 전송 속도가 계속해서 변화하게 된다.




## 전송속도

- 패킷이 왕복하는 시간은 RTT(Round trip time)
- 전송 속도
    ``` 
    rate = CongWin / RTT  (bytes/sec)

    (CongWin = Congestion Window Size)
    ```

- RTT는 이용하는 네트워크 회선에 따라 달라지므로 변동이 크지 않다.

- 반면 전송하게 될 윈도우 사이즈는 계속해서 변화하므로 CongWin의 변동폭은 RTT보다 더 크게 나타난다

- 네트워크 상황이 좋으면 CongWin이 증가하고 네트워크 상황이 나쁘면 CongWin은 감소한다.

따라서 네트워크 전송 속도는 CongWin에 의해 좌우된다. 

## TCP Tahoe

- TCP Tahoe는 Slow start로 시작해서 임계점(threshold)을 넘으면 선형적으로 증가한다.
- 패킷 유실이 시 Congestion Window size가 무조건 1 MSS로 감소한다.
- 이후 초기 상태로 되돌아와 다시 Slow start부터 시작한다.
- Threshold는 유실이 발생했을 때의 절반이 된다.


> Tahoe는 무조건 1 MSS로 다시 되돌아오므로 다시 원상태의 속도로 돌아올 때까지 시간이 많이 걸린다.

이를 개선한 것이 다음 버전인 Reno이다.

## TCP Reno

패킷 유실은 두 상황으로 나눌 수 있다.

1. 타임아웃 발생
- 피드백(ACK)이 돌아오지 않아 타임아웃이 발생하는 케이스
- 타임아웃이 발생할 정도라면 데이터 전송이 원활하게 이루어지지 않는 환경이므로, 네트워크 상태가 상당히 안 좋은 셈이다.

2.  3 중복 ACK
- 복제된 ACK가 세 번 도달하는 것을 3 duplicate ACK라 한다.
- 빠른 재전송(Fast retransmit)에 의해 패킷 유실을 감지하는 케이스이다.
- 해당 패킷만 문제가 있고 네트워크 상태는 정상이다.

네트워크 상황이 나쁘지 않다면 데이터 전송량을 1 MSS까지 감소시킬 이유가 있겠는가?
 Reno는 이 점을 고려한다.


따라서 Reno는 
1. 패킷 유실 판별 조건에 따라 다르게 행동
2. 3 dup ACK → 네트워크 상황이 좋으므로 Congestion Window size를 현재의 절반으로 내린다.
3. Timeout → 기존과 동일하게 Congestion Window size를 1 MSS로 내린다.

Threshold는 Tahoe와 동일하게 (Congestion Window size / 2)로 정한다.


<br>

## 초기 Threshold 값 문제

사실 여기서 하나 문제가 있는데, 바로 Threshold 값이다.

네트워크 상황은 라우터를 하나하나 확인하지 않는 한 정확하게 알 수 없다. 특히 패킷을 보내보기 전까지는 임계점(Threshold)은 예측하기 어렵다.

> Threshold 설정은 경험적이며 동적으로 조정된다