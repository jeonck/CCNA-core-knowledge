# OSI 7계층 모델

## 개요
OSI(Open Systems Interconnection) 7계층 모델은 네트워크 통신을 7개의 계층으로 나누어 설명하는 참조 모델입니다.

## 7계층 구조

| 계층 | 이름 | 주요 기능 | 프로토콜/장비 |
|------|------|-----------|---------------|
| 7 | [[Application Layer\|Application]] | 사용자 인터페이스 | HTTP, FTP, SMTP |
| 6 | [[Presentation Layer\|Presentation]] | 데이터 변환, 암호화 | SSL/TLS, JPEG |
| 5 | [[Session Layer\|Session]] | 세션 관리 | NetBIOS, RPC |
| 4 | [[Transport Layer\|Transport]] | 종단간 전송 | [[TCP]], [[UDP]] |
| 3 | [[Network Layer\|Network]] | 라우팅, 논리 주소 | [[IP]], ICMP, 라우터 |
| 2 | [[Data Link Layer\|Data Link]] | 프레임 전송, 물리 주소 | [[Ethernet]], PPP, 스위치 |
| 1 | [[Physical Layer\|Physical]] | 비트 전송 | 케이블, 허브 |

## 기억법
**Please Do Not Throw Sausage Pizza Away**
- **P**hysical
- **D**ata Link  
- **N**etwork
- **T**ransport
- **S**ession
- **P**resentation
- **A**pplication

## 계층별 상세 설명

### 1계층: Physical Layer
- **기능**: 물리적 신호 전송
- **데이터 단위**: 비트 (Bit)
- **장비**: 케이블, 허브, 리피터
- **특징**: 전기적, 기계적, 기능적 특성 정의

### 2계층: Data Link Layer
- **기능**: 프레임 단위 전송, 오류 검출
- **데이터 단위**: 프레임 (Frame)
- **장비**: 스위치, 브리지
- **주소**: MAC 주소
- **특징**: 흐름제어, 오류제어

### 3계층: Network Layer
- **기능**: 라우팅, 논리적 주소 지정
- **데이터 단위**: 패킷 (Packet)
- **장비**: 라우터, Layer 3 스위치
- **주소**: IP 주소
- **특징**: 경로 결정, 패킷 포워딩

### 4계층: Transport Layer
- **기능**: 종단 간 신뢰성 있는 전송
- **데이터 단위**: 세그먼트 (Segment)
- **프로토콜**: TCP (신뢰성), UDP (속도)
- **특징**: 포트 번호, 흐름제어, 오류복구

### 5계층: Session Layer
- **기능**: 세션 설정, 관리, 종료
- **특징**: 대화 관리, 동기화
- **예시**: NetBIOS, RPC

### 6계층: Presentation Layer
- **기능**: 데이터 변환, 암호화, 압축
- **특징**: 문자 인코딩, 데이터 포맷 변환
- **예시**: SSL/TLS, JPEG, MPEG

### 7계층: Application Layer
- **기능**: 사용자 인터페이스 제공
- **프로토콜**: HTTP, FTP, SMTP, DNS
- **특징**: 네트워크 서비스 접근점

## 데이터 캡슐화 과정

```
Application Data
        ↓
[Header|Application Data] - Segment/Datagram
        ↓
[Header|Segment] - Packet
        ↓  
[Header|Packet|Trailer] - Frame
        ↓
Bits - Physical transmission
```

## 실제 네트워크에서의 적용

### 웹 브라우징 예시
1. **Application**: HTTP 요청 생성
2. **Presentation**: 데이터 암호화 (HTTPS)
3. **Session**: 세션 연결 관리
4. **Transport**: TCP 포트 80/443 사용
5. **Network**: IP 주소로 라우팅
6. **Data Link**: 이더넷 프레임 생성
7. **Physical**: 전기 신호로 전송

## 문제해결에서의 활용

### Bottom-up 접근법
1. Physical: 케이블 연결 확인
2. Data Link: 스위치 포트 상태 확인
3. Network: IP 주소, 라우팅 확인
4. Transport: 포트, 방화벽 확인
5. 상위 계층: 애플리케이션 설정 확인

### Top-down 접근법
1. Application: 사용자 증상 파악
2. 하위 계층으로 순차적 확인

## 관련 주제
- [[TCP-IP 모델]] - 실제 인터넷 프로토콜 스택
- [[네트워크 토폴로지]] - 물리적/논리적 구조
- [[문제해결 방법론]] - OSI 모델 활용한 문제해결

## 태그
#ccna #osi #네트워크기초 #프로토콜 #계층모델
