# 애플리케이션 계층 1 - 소켓 프로그래밍

---

## 소켓이란?

소켓은 **애플리케이션과 네트워크 사이의 인터페이스**다. 애플리케이션이 소켓을 생성하고, 소켓 타입이 통신 스타일을 결정한다.

### 소켓의 역할
- 애플리케이션이 소켓에 데이터를 전달하면 네트워크로 전송
- 네트워크를 통해 전송된 데이터를 소켓에서 받음
- 애플리케이션 개발자가 제어, 운영체제가 관리

---

## 두 가지 필수 소켓 타입

### SOCK_STREAM (TCP)
- **신뢰성 있는 전송**
- **순서 보장**
- **연결 지향적**
- **양방향 통신**

### SOCK_DGRAM (UDP)
- **신뢰성 없는 전송**
- **순서 보장 없음**
- **연결 개념 없음** - 각 패킷마다 목적지 지정
- **송신 또는 수신 가능**

---

## 소켓 API

### 소켓 생성 및 설정
```c
#include <sys/socket.h>
```

### 주요 함수들

#### socket()
```c
int socket(int domain, int type, int protocol);
```
- 소켓을 생성한다
- 성공 시 파일 디스크립터 반환, 실패 시 -1 반환

**매개변수:**
- **domain**: 프로토콜 패밀리 (PF_INET for IPv4)
- **type**: 통신 스타일 (SOCK_STREAM for TCP, SOCK_DGRAM for UDP)
- **protocol**: 패밀리 내 프로토콜 (보통 0)

#### bind()
```c
int bind(int sockfd, struct sockaddr* myaddr, int addrlen);
```
- 소켓을 로컬 IP 주소와 포트 번호에 바인딩
- 성공 시 0 반환, 실패 시 -1 반환

#### listen()
```c
int listen(int sockfd, int backlog);
```
- 소켓을 수동 상태로 설정 (연결 대기)
- 비차단 함수 (즉시 반환)

#### accept()
```c
int accept(int sockfd, struct sockaddr* cliaddr, int* addrlen);
```
- 새로운 연결을 수락
- 차단 함수 (연결이 올 때까지 대기)

---

## TCP 서버/클라이언트 동작 과정

### TCP 서버
1. **socket()**: 소켓 생성
2. **bind()**: 소켓을 포트에 바인딩
3. **listen()**: 연결 대기 상태로 설정
4. **accept()**: 클라이언트 연결 수락 (차단)
5. **read()/write()**: 데이터 송수신
6. **close()**: 연결 종료

### TCP 클라이언트
1. **socket()**: 소켓 생성
2. **connect()**: 서버에 연결
3. **write()/read()**: 데이터 송수신
4. **close()**: 연결 종료

### TCP 3-way handshaking
클라이언트의 connect()와 서버의 accept() 사이에서 TCP 3-way handshake가 발생한다.

---


- 차단 함수 (연결이 올 때까지 대기)

#### connect()
```c
int connect(int sockfd, struct sockaddr* servaddr, int addrlen);
```
- 다른 소켓에 연결
- 성공 시 0 반환, 실패 시 -1 반환
- 차단 함수

**매개변수:**
- **sockfd**: 소켓 파일 디스크립터
- **servaddr**: 서버의 IP 주소와 포트 번호
- **addrlen**: 주소 구조체의 길이

---

## TCP 소켓 연결 설정 과정

### 클라이언트
1. **socket()**: 소켓 생성
2. **connect()**: 서버에 연결 (TCP 3-way handshaking 발생)

### 서버
1. **socket()**: 소켓 생성
2. **bind()**: 소켓을 포트에 바인딩
3. **listen()**: 연결 대기 상태로 설정
4. **accept()**: 클라이언트 연결 수락

---

## 데이터 송수신 함수

### write()
```c
int write(int sockfd, char* buf, size_t nbytes);
```
- 스트림(TCP)에 데이터를 쓴다
- 쓰여진 바이트 수 반환, 실패 시 -1 반환
- 차단 함수 (데이터가 전송된 후에만 반환)

### read()
```c
int read(int sockfd, char* buf, size_t nbytes);
```
- 스트림(TCP)에서 데이터를 읽는다
- 읽은 바이트 수 반환, 실패 시 -1 반환
- 소켓이 닫히면 0 반환
- 차단 함수 (데이터를 받은 후에만 반환)

---

## UDP 소켓 함수

### UDP 클라이언트
1. **socket()**: 소켓 생성
2. **sendto()**: 데이터 전송
3. **recvfrom()**: 데이터 수신
4. **close()**: 소켓 종료

### UDP 서버
1. **socket()**: 소켓 생성
2. **bind()**: 소켓을 포트에 바인딩
3. **recvfrom()**: 데이터 수신
4. **sendto()**: 데이터 전송

---

## 연결 종료

### close()
```c
int close(int sockfd);
```
- 소켓 사용이 끝나면 소켓을 닫아야 함
- 성공 시 0 반환, 실패 시 -1 반환
- 소켓이 사용하던 포트를 해제
- SOCK_STREAM의 경우 연결을 닫음

### 포트 해제 팁
때로는 프로그램의 "거친" 종료(예: ctrl-c)가 포트를 제대로 해제하지 못할 수 있다. 몇 분 후에 포트가 해제되지만, 이 문제를 줄이기 위해 다음 코드를 포함할 수 있다:

```c
#include <signal.h>
void cleanExit(){exit(0);}

// 소켓 코드에서:
signal(SIGTERM, cleanExit);
signal(SIGINT, cleanExit);
```

---

## 샘플 코드

### 서버 코드 구조
```c
#include <sys/socket.h>
#include <netinet/in.h>

#define PORT 3490
#define BACKLOG 10

int main() {
    int sockfd, new_fd;
    struct sockaddr_in my_addr, their_addr;
    
    // socket() 생성
    // bind() 바인딩
    // listen() 대기 상태
    // accept() 연결 수락
    // 데이터 송수신
    // close() 종료
}
```

### 클라이언트 코드 구조
```c
int main() {
    int sockfd;
    struct sockaddr_in their_addr;
    
    // socket() 생성
    // connect() 연결
    // 데이터 송수신
    // close() 종료
}
```

## 소켓 프로그래밍 참고자료

**TCP/IP 소켓 프로그래밍 - C버전**
- Michael J. Donahoo (박준철 번역)
- 사이텍미디어

---

## 전송 계층 (Transport Layer)

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

## 멀티플렉싱과 디멀티플렉싱

### 멀티플렉싱/디멀티플렉싱
- **디멀티플렉싱**: 수신 호스트에서 받은 세그먼트를 올바른 소켓으로 전달
- **멀티플렉싱**: 송신 호스트에서 여러 소켓에서 데이터를 수집하고, 헤더로 감싸기

### 디멀티플렉싱 동작 방식
호스트는 IP 데이터그램을 받는다:
- 각 데이터그램은 소스 IP 주소, 목적지 IP 주소를 가짐
- 각 데이터그램은 1개의 전송 계층 세그먼트를 운반
- 각 세그먼트는 소스, 목적지 포트 번호를 가짐
- 호스트는 IP 주소와 포트 번호를 사용해 세그먼트를 적절한 소켓으로 전달

### TCP/UDP 세그먼트 형식
```
32 bits
source port # dest port #
other header fields
application
data
(message)
```

### 연결 지향 디멀티플렉싱: 스레드 웹 서버
여러 클라이언트가 같은 포트(80)로 연결하지만, 각각 다른 소켓으로 처리된다.

---

## UDP: User Datagram Protocol

### UDP 특징
- **"no frills," "bare bones"** 인터넷 전송 프로토콜
- **"best effort"** 서비스, UDP 세그먼트는 손실되거나 순서가 바뀔 수 있음
- **비연결적**: UDP 송신자와 수신자 간 핸드셰이킹 없음
- 각 UDP 세그먼트는 다른 세그먼트와 독립적으로 처리

### UDP가 존재하는 이유
- **연결 설정 없음** (지연을 줄일 수 있음)
- **단순함**: 송신자, 수신자에 연결 상태 없음
- **작은 세그먼트 헤더**
- **혼잡 제어 없음**: UDP는 원하는 만큼 빠르게 전송 가능

### UDP 사용
- **스트리밍 멀티미디어 앱**에 자주 사용
  - 손실 허용
  - 속도 민감
- **기타 UDP 사용**: DNS, SNMP
- **UDP를 통한 신뢰성 있는 전송**: 애플리케이션 계층에서 신뢰성 추가

### UDP 세그먼트 형식
```
32 bits
source port # dest port #
length checksum
Application
data
(message)
```

---