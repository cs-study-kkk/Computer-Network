# 6. 전송계층2

## TCP
- point-to-point: 한쌍의 프로세스(더 정확하게는 한쌍의 소켓)의 통신을 책임진다.
- reliable, in-order byte stream
- pipelined
- send & receive buffers: 실제로는 sender와 receiver의 각각 관계가 아니라 sender이자 receiver이다. 
- full duplex data
- connection-oriented
- flow controlled: receiver의 소화 능력에 맞게 알맞은 양을 조절해서 보내줌


---
## TCP segment 구조

source port #: 보내는 사람의 socket


checksum: error detection


---
## TCP의 동작

TCP에서 사용하는 seq num: 제일 앞에 있는 byte의 순서 번호
ex) 덩어리가 10 ~ 17까지 있다? -> seq # =  10
TCP에서 사용하는 ACK : cumulative ACK
go-back-N에서는  ACK10은 10번까지 잘 받았다지만, TCP에서는 9번까지 잘 받았다는 뜻


---
## Timeout

timeout value를 얼마로 설정할 것인가?
작게하면, recovery 빠르지만 네트워크에 쓸데없는 overhead를 줌
크게하면, 반대


RTT가 주어질 때, timeout value를 RTT로 생각해보자
RTT값이 고정되어 있으면 좋지만, 실제로는 그렇지 않음
-> 각 segment들이 지나가는 경로가 다르기 때문 (한국 -> 일본, 한국 -> 미국)
-> 경로가 같아도 RTT값이 Queueing delay 때문에 다 다름 ( 같은 길이지만 목적지까지 가는 시간이 상황에 따라 다 다름)

=> 대표할 수 있을만한 대표 RTT 값 : EstimatedRTT (보정 값)


EstimatedRTT = (1- a) *EstimatedRTT + a*SampleRTT
(typical value: a = 0.125, SampleRTT: 실제 측정 값)


실제로 사용하는 timeout 값
TimeoutInterval = EstimatedRTT + 4*DevRTT


DevRTT = (1-ẞ) *DevRTT +B * | SampleRTT-EstimatedRTT|
(typically, ẞ = 0.25)


---
## TCP reliable data transfer
- 파이프라인 방식
- Cumulative acks
- TCP는 timer 1개 사용(go-back-N과 비슷하지만, TCP에서는 해당하는 segment만 재전송)
