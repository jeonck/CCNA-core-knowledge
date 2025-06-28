# EIGRP (Enhanced Interior Gateway Routing Protocol)

## 개요
EIGRP는 Cisco가 개발한 하이브리드 라우팅 프로토콜입니다. Distance Vector와 Link State 프로토콜의 장점을 결합하여 빠른 컨버전스와 효율적인 라우팅을 제공합니다. [[라우팅 기초]]를 기반으로 하며 [[OSPF]]와 함께 기업 네트워크에서 널리 사용됩니다.

## EIGRP 특징

### 하이브리드 프로토콜
```
Distance Vector 특징:
✅ 이웃과만 정보 교환
✅ 홉 수 대신 메트릭 사용
✅ 간단한 설정

Link State 특징:
✅ 토폴로지 정보 유지
✅ 빠른 컨버전스
✅ 루프 방지 메커니즘
```

### 주요 장점
```
✅ 빠른 컨버전스 (DUAL 알고리즘)
✅ 효율적인 대역폭 사용
✅ VLSM 및 CIDR 지원
✅ 로드 밸런싱 (Equal/Unequal Cost)
✅ 자동 요약 (설정 가능)
✅ 확장성 (대규모 네트워크)
✅ 보안 (인증 지원)
```

### Cisco 독점에서 오픈 표준으로
```
2013년: Cisco가 EIGRP를 오픈 표준으로 공개
RFC 7868: EIGRP 표준 문서 발표
현재: 다른 벤더도 구현 가능
```

## EIGRP 동작 원리

### DUAL (Diffusing Update Algorithm)
```
목적: 루프 없는 최적 경로 선택
특징:
- 유한 상태 머신 (Finite State Machine)
- 즉시 백업 경로 활성화
- 루프 방지 보장

상태:
1. Passive: 안정 상태
2. Active: 경로 재계산 중
```

### 3개의 테이블
```
1. Neighbor Table (이웃 테이블)
   - 직접 연결된 EIGRP 라우터 정보
   - Hello 패킷으로 관계 유지

2. Topology Table (토폴로지 테이블)
   - 모든 학습된 경로 정보 저장
   - Successor와 Feasible Successor 관리

3. Routing Table (라우팅 테이블)
   - 최적 경로만 저장
   - Successor 경로만 포함
```

## EIGRP 용어

### 핵심 용어
```
Successor:
- 목적지까지의 최적 경로
- 가장 낮은 FD(Feasible Distance)를 가진 경로
- 라우팅 테이블에 설치됨

Feasible Successor (FS):
- 백업 경로
- 루프 방지 조건을 만족하는 경로
- 즉시 사용 가능한 대안 경로

Feasible Distance (FD):
- 로컬 라우터에서 목적지까지의 최적 메트릭
- Successor의 메트릭

Advertised Distance (AD):
- Next-hop 라우터에서 목적지까지의 메트릭
- Reported Distance (RD)라고도 함
```

### Feasible Successor 조건
```
조건: AD < FD

예시:
목적지 네트워크 X까지의 경로:
- 경로 A: FD = 100 (Successor)
- 경로 B: AD = 80 → FS 가능 (80 < 100)  
- 경로 C: AD = 120 → FS 불가 (120 > 100)

이유: 루프 방지를 위한 안전 장치
```

## EIGRP 메트릭

### 복합 메트릭
```
EIGRP 메트릭 = 256 × (K1×BW + K2×BW/(256-Load) + K3×Delay) × (K5/(Reliability+K4))

기본 K값:
K1 = 1 (대역폭 가중치)
K2 = 0 (로드 가중치)  
K3 = 1 (지연 가중치)
K4 = 0 (신뢰성 가중치)
K5 = 0 (MTU 가중치)

실제 사용 공식 (기본값):
메트릭 = 256 × (10^7/최소대역폭 + 지연합계)
```

### 메트릭 구성 요소
```
대역폭 (Bandwidth):
- 경로상 최소 대역폭
- 단위: Kbps
- 가장 큰 영향을 미침

지연 (Delay):
- 경로상 모든 지연의 합
- 단위: 마이크로초 (μs)
- 두 번째로 큰 영향

로드 (Load):
- 인터페이스 사용률
- 0-255 범위 (255 = 100%)
- 기본적으로 사용하지 않음

신뢰성 (Reliability):
- 링크 안정성
- 0-255 범위 (255 = 100%)
- 기본적으로 사용하지 않음

MTU (Maximum Transmission Unit):
- 경로상 최소 MTU
- 메트릭 계산에 직접 사용하지 않음
```

### 메트릭 계산 예시
```
예시 토폴로지:
R1 ──(T1)── R2 ──(Ethernet)── R3

T1: 
- 대역폭: 1544 Kbps
- 지연: 20000 μs

Ethernet:
- 대역폭: 10000 Kbps  
- 지연: 1000 μs

R1에서 R3까지 메트릭:
최소 대역폭: 1544 Kbps
지연 합계: 20000 + 1000 = 21000 μs

메트릭 = 256 × (10^7/1544 + 21000)
       = 256 × (6476 + 21000)
       = 256 × 27476
       = 7,033,856
```

## EIGRP 패킷 유형

### 5가지 패킷 유형
```
1. Hello Packet:
   - 이웃 발견 및 유지
   - 주기: 5초 (고속 링크), 60초 (저속 링크)
   - Hold Time: 15초 (고속), 180초 (저속)

2. Update Packet:
   - 라우팅 정보 전송
   - 변화 발생 시에만 전송
   - 신뢰성 있는 전송 (ACK 필요)

3. Query Packet:
   - 경로 정보 요청
   - Successor 손실 시 전송
   - 신뢰성 있는 전송

4. Reply Packet:
   - Query에 대한 응답
   - 신뢰성 있는 전송

5. ACK Packet:
   - Update, Query, Reply 확인
   - Hello 패킷과 동일하지만 데이터 없음
```

### 전송 방식
```
Reliable Transport Protocol (RTP):
- TCP와 유사한 신뢰성 보장
- 시퀀스 번호 사용
- 재전송 메커니즘

멀티캐스트 주소: 224.0.0.10
유니캐스트: 특정 이웃에게만 전송
```

## EIGRP 기본 설정

### 기본 설정
```cisco
# EIGRP 프로세스 시작 (AS 번호)
Router(config)# router eigrp 100

# 네트워크 선언 (클래스풀)
Router(config-router)# network 192.168.1.0
Router(config-router)# network 10.0.0.0

# 자동 요약 비활성화 (권장)
Router(config-router)# no auto-summary

# Router ID 설정 (선택사항)
Router(config-router)# eigrp router-id 1.1.1.1
```

### 클래스리스 설정
```cisco
# 정확한 서브넷 선언
Router(config)# router eigrp 100
Router(config-router)# network 192.168.1.0 0.0.0.255
Router(config-router)# network 10.1.1.0 0.0.0.3
Router(config-router)# no auto-summary

# 인터페이스별 설정 (대안)
Router(config)# interface fastethernet 0/0
Router(config-if)# ip eigrp 100
```

### 고급 설정
```cisco
# 수동 이웃 설정 (NBMA)
Router(config-router)# neighbor 10.1.1.2 fastethernet 0/0

# 패시브 인터페이스 (라우팅 정보만 광고, Hello 안 보냄)
Router(config-router)# passive-interface default
Router(config-router)# no passive-interface fastethernet 0/1

# 대역폭 및 지연 수동 설정
Router(config)# interface serial 0/0/0
Router(config-if)# bandwidth 256
Router(config-if)# delay 100
```

## EIGRP 확인 명령어

### 기본 확인
```cisco
# 이웃 테이블
Router# show ip eigrp neighbors

# 토폴로지 테이블
Router# show ip eigrp topology

# 특정 네트워크의 토폴로지
Router# show ip eigrp topology 192.168.2.0/24

# 모든 경로 (FS 포함)
Router# show ip eigrp topology all-links

# EIGRP 라우팅 테이블
Router# show ip route eigrp
```

### 상세 정보
```cisco
# EIGRP 프로토콜 정보
Router# show ip protocols

# 인터페이스별 EIGRP 정보
Router# show ip eigrp interfaces

# EIGRP 통계
Router# show ip eigrp traffic

# 특정 AS 정보
Router# show ip eigrp 100 neighbors
```

### 이웃 테이블 해석
```
Router# show ip eigrp neighbors
EIGRP-IPv4 Neighbors for AS(100)
H   Address                 Interface              Hold Uptime   SRTT   RTO  Q  Seq
                                                   (sec)         (ms)       Cnt Num
0   10.1.1.2                Se0/0/0                  13 00:10:55   40   240  0  15

H: Handle (이웃 순서)
Address: 이웃 IP 주소
Interface: 연결 인터페이스
Hold: Hold Time (죽었다고 판단하는 시간)
Uptime: 이웃 관계 지속 시간
SRTT: Smooth Round Trip Time
RTO: Retransmission Timeout
Q Cnt: 전송 대기 중인 패킷 수
Seq Num: 마지막 시퀀스 번호
```

### 토폴로지 테이블 해석
```
Router# show ip eigrp topology
EIGRP-IPv4 Topology Table for AS(100)/ID(1.1.1.1)

P 192.168.1.0/24, 1 successors, FD is 2169856
        via Connected, FastEthernet0/0
P 192.168.2.0/24, 1 successors, FD is 2681856
        via 10.1.1.2 (2681856/156160), Serial0/0/0
        via 10.1.1.6 (3193856/2169856), Serial0/0/1

P: Passive 상태 (A: Active 상태)
192.168.2.0/24: 목적지 네트워크
1 successors: Successor 개수
FD is 2681856: Feasible Distance
via 10.1.1.2: Next-hop 주소
(2681856/156160): (FD/AD)
```

## EIGRP 고급 기능

### 언이퀄 코스트 로드 밸런싱
```cisco
# Variance 설정 (EIGRP 고유 기능)
Router(config-router)# variance 2

동작 원리:
- Successor 메트릭 × Variance 값 이하인 FS 사용
- 최대 6개 경로까지 로드 밸런싱
- 메트릭에 비례하여 트래픽 분배

예시:
Successor: 메트릭 100
FS1: 메트릭 150
FS2: 메트릭 250

Variance 2 설정 시:
100 × 2 = 200
→ FS1 사용 (150 < 200)
→ FS2 사용 안함 (250 > 200)
```

### 요약 (Summarization)
```cisco
# 인터페이스에서 수동 요약
Router(config)# interface serial 0/0/0
Router(config-if)# ip summary-address eigrp 100 192.168.0.0 255.255.252.0

# 자동 요약 (기본적으로 비활성화)
Router(config-router)# auto-summary

효과:
✅ 라우팅 테이블 크기 감소
✅ 토폴로지 변화 영향 제한
✅ 대역폭 절약
✅ 메모리 사용량 감소
```

### Stub 라우터
```cisco
# Stub 설정 (지사 라우터에 유용)
Router(config-router)# eigrp stub connected summary

Stub 옵션:
- connected: 직접 연결된 경로만 광고
- summary: 요약 경로만 광고
- static: 정적 경로 광고
- redistributed: 재분배 경로 광고
- receive-only: 아무것도 광고하지 않음

장점:
✅ Query 범위 제한
✅ 컨버전스 시간 단축
✅ 네트워크 안정성 향상
```

## EIGRP 인증

### MD5 인증 설정
```cisco
# 키 체인 생성
Router(config)# key chain EIGRP_KEYS
Router(config-keychain)# key 1
Router(config-keychain-key)# key-string cisco123

# 인터페이스에서 인증 활성화
Router(config)# interface fastethernet 0/0
Router(config-if)# ip authentication mode eigrp 100 md5
Router(config-if)# ip authentication key-chain eigrp 100 EIGRP_KEYS
```

### 인증 확인
```cisco
Router# show ip eigrp neighbors detail
EIGRP-IPv4 Neighbors for AS(100)
H   Address      Interface   Hold Uptime   SRTT   RTO  Q  Seq
                             (sec)         (ms)       Cnt Num
0   10.1.1.2     Fa0/0        14 00:05:12   12   200  0  45
   Version 23.0/2.0, Retrans: 0, Retries: 0, Prefixes: 3
   Authentication mode is md5, key-chain is "EIGRP_KEYS"
```

## EIGRP 문제해결

### 일반적인 문제
```
1. 이웃 관계 형성 실패
   원인:
   - 다른 AS 번호
   - 네트워크 선언 누락
   - 인증 불일치
   - K값 불일치

   확인: show ip eigrp neighbors
   해결: 설정 통일

2. 경로가 라우팅 테이블에 없음
   원인:
   - Successor 없음
   - 네트워크 분할
   - 필터링 적용

   확인: show ip eigrp topology
   해결: 연결성 확인, 메트릭 조정

3. Active 상태에서 멈춤 (SIA)
   원인:
   - Query 응답 지연
   - 네트워크 루프
   - 링크 불안정

   증상: show ip eigrp topology active
   해결: Stub 설정, 요약 적용
```

### 디버깅
```cisco
# EIGRP 이웃 관계 디버그
Router# debug eigrp neighbor

# EIGRP 패킷 디버그
Router# debug eigrp packets

# EIGRP FSM 디버그
Router# debug eigrp fsm

# 특정 네트워크만 디버그
Router# debug eigrp 192.168.1.0 255.255.255.0
```

### SIA (Stuck In Active) 문제
```
발생 원인:
- Query에 대한 Reply 지연
- 네트워크 크기가 너무 큼
- 링크 불안정

예방 방법:
✅ Stub 라우터 사용
✅ 적절한 요약 적용
✅ 네트워크 계층화
✅ 안정적인 링크 사용

해결:
Router(config-router)# timers active-time 90
```

## EIGRP vs 다른 프로토콜

### EIGRP vs RIP
| 특징 | EIGRP | RIP |
|------|-------|-----|
| 알고리즘 | DUAL | Bellman-Ford |
| 메트릭 | 복합 메트릭 | 홉 수 |
| 컨버전스 | 빠름 | 느림 |
| 확장성 | 높음 | 낮음 |
| 표준 | Cisco → RFC | 표준 |

### EIGRP vs OSPF
| 특징 | EIGRP | OSPF |
|------|-------|-----|
| 복잡성 | 중간 | 높음 |
| 메모리 사용 | 적음 | 많음 |
| CPU 사용 | 적음 | 많음 |
| 설정 | 간단 | 복잡 |
| 로드 밸런싱 | Unequal 지원 | Equal만 |

## EIGRP 최적화

### 성능 튜닝
```cisco
# Hello/Hold 타이머 조정
Router(config-if)# ip hello-interval eigrp 100 2
Router(config-if)# ip hold-time eigrp 100 6

# 대역폭 사용률 제한
Router(config-if)# ip bandwidth-percent eigrp 100 25

# 최대 경로 수 증가
Router(config-router)# maximum-paths 6
```

### 메트릭 가중치 조정
```cisco
# K값 변경 (모든 라우터에서 동일하게)
Router(config-router)# metric weights 0 1 0 1 0 0

# 인터페이스별 메트릭 조정
Router(config-if)# bandwidth 1000
Router(config-if)# delay 100
```

## 관련 주제
- [[라우팅 기초]] - 라우팅 프로토콜 기초
- [[OSPF]] - 다른 고급 라우팅 프로토콜
- [[정적 라우팅]] - 정적 경로와의 비교
- [[문제해결 방법론]] - EIGRP 문제해결

## 태그
#ccna #eigrp #라우팅프로토콜 #dual #동적라우팅 #cisco
