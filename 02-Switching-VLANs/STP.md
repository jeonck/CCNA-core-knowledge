# STP (Spanning Tree Protocol)

## 개요
STP(Spanning Tree Protocol)는 레이어 2 네트워크에서 루프를 방지하고 중복 경로를 제공하는 프로토콜입니다. [[VLAN 구성]]과 함께 사용되어 안정적이고 신뢰성 있는 [[이더넷 기초]] 네트워크를 구축합니다.

## STP의 필요성

### 루프의 문제점
```
1. 브로드캐스트 스톰 (Broadcast Storm)
   - 브로드캐스트 프레임이 네트워크에서 무한 순환
   - 네트워크 대역폭 고갈
   - 네트워크 마비

2. 중복 프레임 (Duplicate Frames)
   - 같은 프레임이 여러 경로로 전달
   - 수신자가 동일한 데이터를 중복 수신

3. MAC 주소 테이블 불안정
   - 스위치가 같은 MAC 주소를 여러 포트에서 학습
   - 프레임 포워딩 결정 불가
```

### STP의 해결책
```
✅ 논리적으로 루프가 없는 토폴로지 생성
✅ 중복 경로 제공으로 가용성 향상
✅ 링크 장애 시 자동 복구
✅ 브로드캐스트 스톰 방지
```

## STP 동작 원리

### 3단계 프로세스
```
1. Root Bridge 선출
   ↓
2. Root Port 선택 (각 스위치)
   ↓  
3. Designated Port 선택 (각 세그먼트)
   ↓
4. Blocking Port 결정 (나머지)
```

### Bridge ID 구성
```
Bridge ID = Bridge Priority + MAC Address
           (2 bytes)        (6 bytes)

예시:
Priority: 32768 (기본값)
MAC:     0012.3456.789a
Bridge ID: 32768.0012.3456.789a

우선순위: 낮은 Bridge ID가 우선
```

### Root Bridge 선출
```
과정:
1. 모든 스위치가 BPDU(Bridge Protocol Data Unit) 교환
2. 가장 낮은 Bridge ID를 가진 스위치가 Root Bridge 선출
3. Root Bridge의 모든 포트는 Designated Port

Root Bridge 수동 설정:
SW1(config)# spanning-tree vlan 1 priority 4096
```

### Root Port 선출
```
기준 (우선순위 순):
1. Root Bridge까지의 Path Cost가 가장 낮은 포트
2. 직접 연결된 이웃의 Bridge ID가 가장 낮음
3. 이웃의 Port ID가 가장 낮음
4. 자신의 Port ID가 가장 낮음

특징:
- 각 스위치당 하나의 Root Port만 존재
- Root Bridge는 Root Port 없음
```

### Designated Port 선출
```
각 네트워크 세그먼트마다 하나씩 선출

기준:
1. Root Bridge까지의 Path Cost가 가장 낮은 스위치의 포트
2. Bridge ID가 가장 낮은 스위치의 포트
3. Port ID가 가장 낮은 포트

역할: 해당 세그먼트에서 Root Bridge로의 최적 경로 제공
```

## Path Cost

### IEEE 802.1D-1998 표준
| 링크 속도 | STP Cost |
|-----------|----------|
| 10 Mbps | 100 |
| 100 Mbps | 19 |
| 1 Gbps | 4 |
| 10 Gbps | 2 |

### IEEE 802.1D-2004 표준 (장거리)
| 링크 속도 | STP Cost |
|-----------|----------|
| 10 Mbps | 2,000,000 |
| 100 Mbps | 200,000 |
| 1 Gbps | 20,000 |
| 10 Gbps | 2,000 |
| 100 Gbps | 200 |

### Cost 수동 설정
```cisco
# 인터페이스별 Cost 설정
SW1(config)# interface fastethernet 0/1
SW1(config-if)# spanning-tree cost 50

# VLAN별 Cost 설정
SW1(config-if)# spanning-tree vlan 10 cost 25
```

## 포트 상태 (Port States)

### 5가지 포트 상태
```
1. Disabled (Down)
   - 포트가 비활성화됨
   - STP에 참여하지 않음

2. Blocking
   - BPDU만 수신
   - 데이터 프레임 전송/수신하지 않음
   - MAC 주소 학습하지 않음
   - 기간: 즉시 또는 Max Age (20초)

3. Listening  
   - BPDU 송수신
   - 데이터 프레임 전송/수신하지 않음
   - MAC 주소 학습하지 않음
   - 기간: Forward Delay (15초)

4. Learning
   - BPDU 송수신
   - MAC 주소 학습 시작
   - 데이터 프레임 전송하지 않음
   - 기간: Forward Delay (15초)

5. Forwarding
   - 정상적인 데이터 전송
   - 모든 프레임 처리
   - MAC 주소 학습
```

### 상태 전환 시간
```
Blocking → Listening → Learning → Forwarding
 즉시      15초       15초      정상동작

총 컨버전스 시간: 최대 50초 (20+15+15)
```

## BPDU (Bridge Protocol Data Unit)

### BPDU 유형
```
Configuration BPDU:
- Root Bridge 정보 전달
- 토폴로지 정보 교환
- 2초마다 전송

TCN BPDU (Topology Change Notification):
- 토폴로지 변화 알림
- Root Bridge로 전송
```

### BPDU 주요 필드
```
Root Bridge ID: Root Bridge 정보
Root Path Cost: Root Bridge까지의 비용
Sender Bridge ID: 송신자 정보
Port ID: 송신 포트 정보
Message Age: BPDU 나이
Max Age: BPDU 최대 유효시간 (20초)
Hello Time: BPDU 전송 간격 (2초)
Forward Delay: 상태 전환 시간 (15초)
```

## STP 타이머

### 기본 타이머
```
Hello Timer: 2초
- Configuration BPDU 전송 간격
- Root Bridge에서만 생성

Max Age: 20초  
- BPDU 최대 유효시간
- 이 시간 후 BPDU 폐기

Forward Delay: 15초
- Listening과 Learning 상태 유지 시간
- 안전한 상태 전환 보장

Message Age: 현재값
- BPDU가 Root Bridge에서 생성된 후 경과 시간
```

### 타이머 수동 설정
```cisco
# Root Bridge에서만 설정 (전체 네트워크에 영향)
SW1(config)# spanning-tree vlan 1 hello-time 1
SW1(config)# spanning-tree vlan 1 max-age 10  
SW1(config)# spanning-tree vlan 1 forward-time 7

# 권장 비율
Max Age ≥ 2 × (Forward Delay - 1)
Max Age ≤ 2 × Hello Time
```

## STP 종류

### STP (802.1D)
```
특징:
- 원조 Spanning Tree Protocol
- 모든 VLAN이 하나의 STP 인스턴스 공유
- 컨버전스 시간: 30-50초
- 로드 밸런싱 불가

단점:
- 느린 컨버전스
- 모든 VLAN 동일한 토폴로지
```

### PVST+ (Per-VLAN STP)
```
특징:
- Cisco 독점 프로토콜
- VLAN별로 별도의 STP 인스턴스
- VLAN별 다른 Root Bridge 설정 가능
- 로드 밸런싱 가능

장점:
- VLAN별 최적화된 경로
- 링크 활용도 향상
- 세밀한 제어 가능
```

### RSTP (Rapid STP, 802.1w)
```
특징:
- 빠른 컨버전스 (수초 내)
- 백워드 호환성 (802.1D와 호환)
- 포트 역할 세분화
- P2P 링크에서 즉시 전환

개선사항:
- 새로운 BPDU 포맷
- 포트 상태 단순화 (3개 상태)
- Proposal/Agreement 메커니즘
```

### RPVST+ (Rapid PVST+)
```
특징:
- RSTP + PVST+ 결합
- VLAN별 빠른 컨버전스
- Cisco 독점

장점:
- RSTP의 빠른 컨버전스
- PVST+의 VLAN별 제어
```

## RSTP 개선사항

### 포트 역할
```
Root Port: Root Bridge로의 최적 경로
Designated Port: 세그먼트의 지정 포트
Alternate Port: Root Port의 백업
Backup Port: Designated Port의 백업 (같은 스위치)
```

### 포트 상태 (RSTP)
```
Discarding: 데이터 전송하지 않음 (Blocking + Listening)
Learning: MAC 주소 학습
Forwarding: 정상 동작
```

### 빠른 전환 조건
```
Edge Port: 
- 엔드 디바이스 연결 포트
- 즉시 Forwarding 상태
- PortFast와 유사

Point-to-Point Link:
- 두 스위치 간 직접 연결
- Full-duplex 링크
- Proposal/Agreement 사용
```

## STP 기본 설정

### 기본 STP 확인
```cisco
# STP 상태 확인
SW1# show spanning-tree

# VLAN별 STP 정보
SW1# show spanning-tree vlan 1

# 인터페이스별 STP 정보  
SW1# show spanning-tree interface fastethernet 0/1

# STP 요약
SW1# show spanning-tree summary
```

### Root Bridge 설정
```cisco
# Primary Root Bridge 설정 (우선순위 24576)
SW1(config)# spanning-tree vlan 1 root primary

# Secondary Root Bridge 설정 (우선순위 28672)
SW2(config)# spanning-tree vlan 1 root secondary

# 수동 우선순위 설정 (4096 단위)
SW1(config)# spanning-tree vlan 1 priority 4096
```

### 로드 밸런싱 설정
```cisco
# VLAN별 다른 Root Bridge 설정
SW1(config)# spanning-tree vlan 1,3,5,7 root primary
SW1(config)# spanning-tree vlan 2,4,6,8 root secondary

SW2(config)# spanning-tree vlan 1,3,5,7 root secondary  
SW2(config)# spanning-tree vlan 2,4,6,8 root primary
```

## PortFast와 BPDU Guard

### PortFast
```
목적: 엔드 디바이스 연결 포트의 빠른 활성화
동작: STP 상태 전환 과정 생략 (즉시 Forwarding)
주의: 스위치나 허브 연결 시 루프 위험

설정:
SW1(config)# interface fastethernet 0/1
SW1(config-if)# spanning-tree portfast

# 모든 액세스 포트에 적용
SW1(config)# spanning-tree portfast default
```

### BPDU Guard
```
목적: PortFast 포트에서 BPDU 수신 시 포트 차단
보안: 스위치 연결로 인한 루프 방지
동작: BPDU 수신 시 포트를 err-disabled 상태로 변경

설정:
SW1(config)# interface fastethernet 0/1
SW1(config-if)# spanning-tree bpduguard enable

# 글로벌 설정
SW1(config)# spanning-tree portfast bpduguard default
```

### BPDU Filter
```
목적: 특정 포트에서 BPDU 송수신 차단
주의: 잘못 사용 시 루프 위험

설정:
SW1(config)# interface fastethernet 0/1
SW1(config-if)# spanning-tree bpdufilter enable
```

### Root Guard
```
목적: 특정 포트가 Root Port가 되는 것을 방지
용도: 네트워크 경계에서 불법 Root Bridge 방지

설정:
SW1(config)# interface fastethernet 0/1
SW1(config-if)# spanning-tree guard root
```

## STP 문제해결

### 일반적인 문제
```
1. 컨버전스 지연
   원인: 기본 타이머 사용, RSTP 미사용
   해결: RSTP 적용, 타이머 조정

2. 서브옵티말 경로
   원인: 잘못된 Root Bridge 위치
   해결: 적절한 위치에 Root Bridge 설정

3. 루프 발생  
   원인: STP 비활성화, PortFast 남용
   해결: STP 활성화, 적절한 보안 기능 사용

4. 포트 Blocking
   원인: STP 계산 결과
   해결: Cost 조정, 우선순위 변경
```

### 디버깅 명령어
```cisco
# STP 이벤트 디버그
SW1# debug spanning-tree events

# BPDU 디버그
SW1# debug spanning-tree bpdu

# Root Port 선출 과정
SW1# debug spanning-tree root

# 포트 상태 변화
SW1# debug spanning-tree states
```

### 토폴로지 변화 추적
```cisco
# 토폴로지 변화 횟수 확인
SW1# show spanning-tree detail | include changes

# 마지막 토폴로지 변화 정보
SW1# show spanning-tree interface fastethernet 0/1 detail
```

## STP 최적화

### 디자인 권장사항
```
1. Root Bridge 위치
   - 네트워크 중앙에 배치
   - 고성능 스위치 사용
   - 이중화 구성 (Primary/Secondary)

2. 링크 Cost 최적화
   - 고속 링크에 낮은 Cost
   - 백업 링크에 높은 Cost
   - 명시적 Cost 설정

3. 포트별 기능 활용
   - PortFast: 엔드 디바이스
   - BPDU Guard: 보안 강화
   - Root Guard: 경계 보호

4. RSTP 사용
   - 빠른 컨버전스
   - 개선된 메커니즘
   - 백워드 호환성
```

### 성능 모니터링
```cisco
# STP 통계
SW1# show spanning-tree summary totals

# 인터페이스별 통계
SW1# show spanning-tree interface fastethernet 0/1 detail

# BPDU 통계
SW1# show spanning-tree detail | include BPDU
```

## STP 보안

### 보안 위협
```
1. Root Bridge 공격
   - 공격자가 낮은 우선순위로 Root Bridge 탈취
   - 트래픽 가로채기 가능

2. TCN 공격
   - 지속적인 토폴로지 변화 알림
   - MAC 주소 테이블 플러시
   - 네트워크 성능 저하

3. BPDU 공격
   - 가짜 BPDU 전송
   - 네트워크 토폴로지 조작
```

### 보안 대책
```cisco
# Root Guard 설정
SW1(config)# interface range fastethernet 0/1-20
SW1(config-if-range)# spanning-tree guard root

# BPDU Guard 설정
SW1(config)# spanning-tree portfast bpduguard default

# Loop Guard 설정 (단방향 링크 방지)
SW1(config)# spanning-tree loopguard default
```

## 관련 주제
- [[VLAN 구성]] - VLAN과 STP 연동
- [[이더넷 기초]] - 레이어 2 기반 기술
- [[스위치 동작원리]] - 스위칭과 STP
- [[문제해결 방법론]] - STP 문제해결

## 태그
#ccna #stp #스위칭 #루프방지 #네트워크안정성
